---
title: Mac_Antigravity_GUI_配置_Clash_Verge_代理完整笔记
date: 2026-10-15
tags: [Toolchain, Mac, Antigravity, Proxy]
aliases: []
---

Antigravity（Google 的 AI IDE，含 `agy` CLI）在 Mac 上已配置终端代理但仍报 "Authentication required"、模型闪一下就断开的完整排障与修复笔记。

## 一、核心结论

**终端代理配好了 ≠ GUI 应用能走代理。**

三层配置相互独立：

1. **macOS 系统代理**：只对浏览器等遵循系统设置的 GUI 应用生效
2. **终端环境变量**（`HTTP_PROXY` 等）：只对该终端及其子进程（`agy` CLI、curl、git）生效
3. **GUI 应用会话环境**：Antigravity 这类 Electron + Go 语言服务的应用，从 Dock 启动时**既不继承终端环境变量，也不读取 macOS 系统代理**，必须通过 `launchctl setenv` 注入 GUI 会话

Antigravity 报 "Authentication required" 是**误导**——真实原因往往是网络连不上 Google，认证流程走不完。

---

## 二、故障现象与诊断

### 现象

- monocode / Antigravity 里无法使用模型，提示 "Authentication required, Run `agy` once in Terminal to sign in"
- "Gemini 3.8 Flash (High) worked for 2s" 后断开
- 但 `which agy` 有输出（`/Users/comet/.local/bin/agy`）
- 终端里 `agy -p "..."` 也长时间无响应、挂起

### 诊断步骤（按顺序排查）

**1. 确认 CLI 是否安装**

```bash
which agy
```
有输出只代表已安装，不代表已登录、不代表模型可用。

**2. 确认是否已登录（钥匙串凭据）**

```bash
security dump-keychain 2>/dev/null | grep -iE "antigravity|gemini"
```
看到 `svce = gemini` / `acct = antigravity` 条目即代表**登录过**，凭据在钥匙串里。

**3. 确认网络能否直连 Google（关键一步）**

```bash
curl -sS -m 10 -o /dev/null -w "http_code=%{http_code}\n" https://www.googleapis.com/
curl -sS -m 10 -o /dev/null -w "http_code=%{http_code}\n" https://daily-cloudcode-pa.googleapis.com/
```
`http_code=000`（超时）= 网络不通，这就是"需认证"的真相。

**4. 验证代理端口是否可用**

```bash
scutil --proxy            # 查看系统代理端口
nc -z -w2 127.0.0.1 7897  # 探测端口
curl -sS -m 15 -x http://127.0.0.1:7897 -o /dev/null -w "http_code=%{http_code}\n" https://www.googleapis.com/
```
走代理返回 `http_code=404`（根路径无内容但可达）即代理正常。

**5. 验证 `agy` 带代理后可用**

```bash
HTTP_PROXY=http://127.0.0.1:7897 HTTPS_PROXY=http://127.0.0.1:7897 \
ALL_PROXY=http://127.0.0.1:7897 agy -p "reply with exactly: OK"
```
返回 `OK` 即确认：**凭据有效 + 网络通了就能用**。

### 相关日志位置

| 日志 | 路径 |
| --- | --- |
| IDE 主进程（Electron/自动更新） | `~/Library/Logs/Antigravity/main.log` |
| 语言服务（模型请求、认证） | `~/Library/Logs/Antigravity/language_server.log` |
| Antigravity 数据目录 | `~/.gemini/antigravity/` |

语言服务日志里反复出现 `Get "https://www.googleapis.com/oauth2/v2/userinfo": EOF`、`net::ERR_CONNECTION_CLOSED` 即网络不通的铁证。

---

## 三、修复方案：一键脚本（已部署）

脚本路径：`~/.local/bin/antigravity-proxy-setup`

**作用**：检测代理端口 → `launchctl setenv` 注入 GUI 会话 → 写 LaunchAgent 保证重启后自动生效 → 若 Antigravity 在运行则自动重启。

```bash
#!/bin/zsh
# antigravity-proxy-setup
# 为 GUI 会话注入 Clash Verge 代理环境变量，使 Antigravity 能连接 Google 服务
# 用法:
#   antigravity-proxy-setup            配置 + 若 Antigravity 在运行则自动重启
#   antigravity-proxy-setup --no-restart  仅配置，不重启应用
set -euo pipefail

PORT="${CLASH_MIXED_PORT:-7897}"
HTTP_URL="http://127.0.0.1:${PORT}"
SOCKS_URL="socks5://127.0.0.1:${PORT}"
PLIST="$HOME/Library/LaunchAgents/com.user.antigravity-proxy.plist"

if ! nc -z -w2 127.0.0.1 "$PORT"; then
  echo "❌ 端口 ${PORT} 未监听，请先启动 Clash Verge"
  exit 1
fi
echo "✅ 代理端口 ${PORT} 可用"

# 1) 注入当前 GUI 会话（Dock 启动的应用会继承）
launchctl setenv HTTP_PROXY "$HTTP_URL"
launchctl setenv HTTPS_PROXY "$HTTP_URL"
launchctl setenv ALL_PROXY "$SOCKS_URL"
echo "✅ 当前会话已设置代理"

# 2) 登录时自动设置（重启电脑后仍然生效）
mkdir -p "$HOME/Library/LaunchAgents"
cat > "$PLIST" <<EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.user.antigravity-proxy</string>
    <key>RunAtLoad</key>
    <true/>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/sh</string>
        <string>-c</string>
        <string>for i in \$(seq 1 30); do nc -z -w1 127.0.0.1 ${PORT} &amp;&amp; break; sleep 1; done; if nc -z -w1 127.0.0.1 ${PORT}; then launchctl setenv HTTP_PROXY ${HTTP_URL}; launchctl setenv HTTPS_PROXY ${HTTP_URL}; launchctl setenv ALL_PROXY ${SOCKS_URL}; fi</string>
    </array>
</dict>
</plist>
EOF
launchctl unload "$PLIST" 2>/dev/null || true
launchctl load "$PLIST"
echo "✅ 已安装登录自启动配置: $PLIST"

# 3) 重启 Antigravity 使语言服务继承新环境变量
if [ "${1:-}" != "--no-restart" ] && pgrep -qf "Antigravity.app/Contents/MacOS/Antigravity"; then
  echo "↻ 正在重启 Antigravity…"
  osascript -e 'quit app "Antigravity"' 2>/dev/null || true
  sleep 3
  if pgrep -qf "Antigravity.app/Contents/MacOS/Antigravity"; then
    echo "⚠️ Antigravity 尚未退出（可能有未保存提示），请手动退出后重新打开"
  else
    open -a Antigravity
    echo "✅ Antigravity 已重新启动"
  fi
else
  echo "ℹ️ Antigravity 未在运行，下次打开时自动生效"
fi
```

### 设计要点

- **LaunchAgent 内置 30 秒重试**：开机后 Clash Verge 可能比登录晚启动，脚本每秒探测端口、最多等 30 秒再注入，避免开机时代理还没起来导致配置落空
- **端口可通过环境变量覆盖**：`CLASH_MIXED_PORT=9090 antigravity-proxy-setup`，不用改脚本

---

## 四、日常维护

| 场景 | 操作 |
| --- | --- |
| 改 Clash 端口后重新配置 | `CLASH_MIXED_PORT=新端口 ~/.local/bin/antigravity-proxy-setup` |
| Antigravity 正在运行需要生效 | `~/.local/bin/antigravity-proxy-setup`（会自动退出并重启它） |
| 仅配置不动应用 | `~/.local/bin/antigravity-proxy-setup --no-restart` |
| 验证 GUI 会话是否已注入 | `launchctl getenv HTTP_PROXY` |
| 彻底移除 | 见下 |

彻底移除：

```bash
launchctl unload ~/Library/LaunchAgents/com.user.antigravity-proxy.plist
rm ~/Library/LaunchAgents/com.user.antigravity-proxy.plist
launchctl unsetenv HTTP_PROXY
launchctl unsetenv HTTPS_PROXY
launchctl unsetenv ALL_PROXY
```

---

## 五、原理说明

- macOS GUI 应用启动时的环境来自 **launchd 用户会话**（`launchctl getenv` 可见），与终端 shell 完全隔离
- `launchctl setenv` 写入 launchd 会话环境，之后从 Dock/Finder/`open -a` 启动的应用都会继承
- 但已运行的应用**不会**中途刷新环境，所以改完必须 Cmd+Q 完全退出再打开
- Antigravity 的语言服务是 Go 进程，Go 的 `net/http` 只识别 `HTTP_PROXY/HTTPS_PROXY/ALL_PROXY` 环境变量，**不读 macOS 系统代理**（`scutil --proxy` 那套）
- "Authentication required" 本质是：token 验证请求 `https://www.googleapis.com/oauth2/v2/userinfo` 失败 → 客户端认为未登录

---

## 六、避坑

1. **只关窗口不算退出**：必须 `Cmd+Q`，语言服务进程还在的话不会继承新环境
2. **终端能跑 `agy` 不代表 IDE 能用**：两者环境独立，终端走 `.zshrc` 的 `proxy_on`，IDE 走 launchd 会话
3. **开机顺序**：如果开机后立刻打开 Antigravity 而 Clash 还没启动，本次会话无代理——LaunchAgent 的重试只覆盖"登录时"那一次，之后打开的应用靠当前 launchd 环境。养成"先开 Clash 再开 Antigravity"的习惯最稳
4. **换电脑/重装后**：重新部署脚本即可，钥匙串凭据不随文件迁移，需要重新 `agy` 登录一次

---

## 相关笔记

- [[Mac_Terminal_配置_Clash_Verge_代理完整笔记]]（终端侧的 `proxy_on/proxy_off` 方案，本篇是其 GUI 侧补充）
- [[使用国外AI教程]]
- [[macOS_Headscale_Tailscale_接入与避坑指南]]
