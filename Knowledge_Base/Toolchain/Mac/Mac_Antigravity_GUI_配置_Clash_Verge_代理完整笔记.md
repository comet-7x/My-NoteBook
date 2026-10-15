---
title: Mac_Antigravity_GUI_配置_Clash_Verge_代理完整笔记
date: 2026-10-15
tags: [Toolchain, Mac, Antigravity, Proxy, Monocode]
aliases: []
status: 已修复可用（2026-10-15 端到端验证通过）
---

Antigravity（Google 的 AI IDE，含 `agy` CLI）与 monocode 在 Mac 上无法调用 Gemini 模型的完整排障与修复笔记。

> **结论先行**：表面报 "Authentication required"，实际是**三层独立故障**叠加——① GUI 应用不走代理连不上 Google；② monocode 用的 ACP 服务有独立登录态，没登录过；③ ACP 服务缺 `localharness` 二进制。三层都修好才真正可用。

## 一、核心概念：三套相互独立的认证 / 网络栈

| 组件 | 网络代理 | 认证凭据 |
| --- | --- | --- |
| Antigravity IDE（Electron + Go 语言服务） | launchd 会话环境变量 | 系统钥匙串（`gemini` / `antigravity`） |
| `agy` CLI（终端） | 终端 shell 环境变量（`.zshrc` 的 `proxy_on`） | 同 IDE，共享钥匙串 |
| monocode 的 antigravity harness（`agy_acp_server.par`） | 继承 monocode 进程的环境变量 | **独立的** `~/.gemini/antigravity-acp/`，不共享 IDE 登录态 |

关键认知：

- **`which agy` 有输出 ≠ 已登录 ≠ 模型可用**，这是三件独立的事
- **报 "Authentication required" 是误导**——它可能根本不是没登录，而是网络不通导致 token 验证失败，或 ACP 服务自己的凭据缺失
- **终端能跑 `agy` ≠ IDE 能用 ≠ monocode 能用**，三者环境完全隔离
- macOS 系统代理（`scutil --proxy`）只对浏览器等遵循系统设置的 GUI 应用生效；Go 的 `net/http` 只认 `HTTP_PROXY/HTTPS_PROXY/ALL_PROXY` 环境变量

---

## 二、故障现象

- monocode 里选 antigravity 模型，提示 "Authentication required, Run `agy` once in Terminal to sign in"
- "Gemini 3.8 Flash (High) worked for 2s" 后断开
- Antigravity IDE 本身可能正常，也可能同样报错（取决于是否配了代理）
- 终端 `agy -p "..."` 长时间无响应、挂起

---

## 三、第一层修复：GUI 应用代理（launchctl setenv）

### 原理

- macOS GUI 应用从 Dock 启动时继承 **launchd 用户会话**环境（`launchctl getenv` 可见），与终端 shell 隔离
- `launchctl setenv` 写入 launchd 会话后，从 Dock/Finder/`open -a` 启动的应用都会继承
- 但**已运行的应用不会中途刷新**，改完必须 `Cmd+Q` 完全退出再打开

### 一键脚本（已部署）

脚本路径：`~/.local/bin/antigravity-proxy-setup`

**作用**：检测代理端口 → `launchctl setenv` 注入 GUI 会话 → 写 LaunchAgent 保证重启后自动生效 → 若 Antigravity 在运行则自动重启。

```bash
#!/bin/zsh
# antigravity-proxy-setup
# 为 GUI 会话注入 Clash Verge 代理环境变量，使 Antigravity / monocode 能连接 Google 服务
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

- **LaunchAgent 内置 30 秒重试**：开机后 Clash Verge 可能比登录晚启动，脚本每秒探测端口、最多等 30 秒再注入
- **端口可通过环境变量覆盖**：`CLASH_MIXED_PORT=9090 antigravity-proxy-setup`，不用改脚本

### 验证

```bash
# GUI 会话是否已注入
launchctl getenv HTTP_PROXY   # 应输出 http://127.0.0.1:7897

# 确认目标进程真的带上了代理变量（PID 换成实际进程）
ps eww <PID> | tr ' ' '\n' | grep -E '^(HTTP_PROXY|HTTPS_PROXY|ALL_PROXY)='
```

---

## 四、第二层修复：monocode 的 ACP 服务独立登录（关键，最易遗漏）

### 现象

第一层代理配好、monocode 重启后**仍然**报 "Authentication required"。

### 根因

monocode 的 antigravity harness 走的是 `~/.local/share/agy-acp/agy_acp_server.par`（ACP 协议服务），它：

1. 有**自己独立的凭据存储**（`~/.gemini/antigravity-acp/`），**不共享** IDE / CLI 的钥匙串登录态
2. 必须在 `~/.gemini/antigravity-acp/settings.json` 里指定 `auth.type`，否则直接报 `No authentication method selected`

### 诊断

直接对 ACP 服务发 JSON-RPC，看真实报错：

```bash
export HTTP_PROXY=http://127.0.0.1:7897 HTTPS_PROXY=http://127.0.0.1:7897 ALL_PROXY=socks5://127.0.0.1:7897
( printf '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":1,"clientCapabilities":{}}}\n'; sleep 2; \
  printf '{"jsonrpc":"2.0","id":2,"method":"session/new","params":{"cwd":"/tmp","mcpServers":[]}}\n'; sleep 15 ) \
  | ~/.local/share/agy-acp/agy_acp_server.par 2>&1 | grep -E '^\{'
```

- 报 `No authentication method selected ... set auth.type in settings.json` → 还没配 auth
- 配了之后日志出现 `Credentials missing or invalid. Launching browser login flow...` 并打印一条 `https://accounts.google.com/o/oauth2/...` 链接 → 需要手动浏览器登录一次

### 修复步骤

**1. 创建 ACP 的 settings.json，指定认证方式**

```bash
mkdir -p ~/.gemini/antigravity-acp
cat > ~/.gemini/antigravity-acp/settings.json <<'EOF'
{
  "auth": {
    "type": "oauth-personal"
  }
}
EOF
```

**2. 触发浏览器登录（只需一次）**

ACP 服务启动 `session/new` 时会自己打开浏览器。若没自动打开，手动 `open` 日志里打印的那条 Google OAuth 链接，完成登录 + 授权（用 Antigravity 对应的 Google 账号）。成功后浏览器跳到 `127.0.0.1:xxxxx` 页面即完成。

**3. 验证凭据已生效**

再次 `session/new`，日志应显示 `Access token expired. Refreshing silently...`（静默刷新成功）+ `User is already onboarded with project ID: 'aicode-consumers'`，而不是又打印登录链接。

---

## 五、第三层修复：补齐缺失的 localharness 二进制（关键）

### 现象

ACP 服务登录后，`session/new` 卡在 `Starting connection and creating conversation...` 不再返回；或日志开头就有 `Localharness not found.`。

### 根因

ACP 服务运行会话需要调用一个名为 `localharness`（或 `localharness_external`）的专用二进制（讲 GAOS Interactions 协议）。它的查找顺序：

1. 环境变量 `ANTIGRAVITY_HARNESS_PATH`
2. `.par` 同目录 / Python 可执行目录下的 `localharness`
3. pip 包 `google-antigravity` 内嵌的 `google/antigravity/bin/localharness`

本机器三层都没命中（全盘搜不到该文件），而 `agy` CLI 是单文件全内嵌、不提供独立 localharness。

### 修复步骤（从官方 PyPI 包提取）

```bash
export HTTP_PROXY=http://127.0.0.1:7897 HTTPS_PROXY=http://127.0.0.1:7897 ALL_PROXY=socks5://127.0.0.1:7897
cd /tmp && rm -rf agy_sdk && mkdir agy_sdk && cd agy_sdk

# 取 macOS arm64 的 wheel 直链
curl -sS https://pypi.org/pypi/google-antigravity/json -o meta.json
URL=$(python3 -c "
import json
d = json.load(open('meta.json'))
v = d['info']['version']
for u in d['releases'][v]:
    if 'macosx' in u['filename'] and 'arm64' in u['filename']:
        print(u['url']); break
")
curl -sSL -m 120 -o sdk.whl "$URL"

# 解出 localharness（约 115MB），放到 ACP 服务能找到的位置
unzip -o -j sdk.whl "google/antigravity/bin/localharness" -d .
cp localharness ~/.local/share/agy-acp/localharness
chmod +x ~/.local/share/agy-acp/localharness
```

> x86_64 机器把上面的 `arm64` 改成 `x86_64`（wheel 文件名 `..._x86_64.whl`）。

### 端到端验证（确认三层全通）

```bash
export HTTP_PROXY=http://127.0.0.1:7897 HTTPS_PROXY=http://127.0.0.1:7897 ALL_PROXY=socks5://127.0.0.1:7897
# 发 initialize -> session/new -> session/prompt，最终应收到模型实际回复
```

2026-10-15 实测：`session/new` 返回 `sessionId` + 模型列表，`session/prompt` 发消息后模型回复 `E2E-OK`，全链路打通。

### 可用模型

free-tier 账号当前只有 **Gemini 3.8 Flash** 一个模型，3 个推理档位：

- `gemini-3.8-flash-high`（默认）
- `gemini-3.8-flash-medium`
- `gemini-3.8-flash-low`

更多模型（Gemini 3 Pro 等）需升级付费订阅（https://codeassist.google.com/upgrade）。

---

## 六、日常维护

| 场景 | 操作 |
| --- | --- |
| 改 Clash 端口后重新配置 | `CLASH_MIXED_PORT=新端口 ~/.local/bin/antigravity-proxy-setup` |
| Antigravity 正在运行需要生效 | `~/.local/bin/antigravity-proxy-setup`（自动退出并重启） |
| 仅配置不动应用 | `~/.local/bin/antigravity-proxy-setup --no-restart` |
| monocode 改动后不生效 | `Cmd+Q` 完全退出 monocode 再打开（让它重新拉起 ACP 服务） |
| 验证 GUI 会话已注入 | `launchctl getenv HTTP_PROXY` |
| ACP 又报 Authentication | 检查 `~/.gemini/antigravity-acp/settings.json` 是否在、token 是否过期需重新浏览器登录 |
| localharness 丢失/升级 | 重跑第五步，从 PyPI 重新提取并覆盖 |
| 彻底移除代理配置 | 见下 |

彻底移除代理：

```bash
launchctl unload ~/Library/LaunchAgents/com.user.antigravity-proxy.plist
rm ~/Library/LaunchAgents/com.user.antigravity-proxy.plist
launchctl unsetenv HTTP_PROXY
launchctl unsetenv HTTPS_PROXY
launchctl unsetenv ALL_PROXY
```

---

## 七、原理与日志速查

### 关键路径

| 项目 | 路径 |
| --- | --- |
| ACP 服务二进制 | `~/.local/share/agy-acp/agy_acp_server.par` |
| ACP 登录凭据 / 设置 | `~/.gemini/antigravity-acp/`（`settings.json`） |
| localharness（需手动补） | `~/.local/share/agy-acp/localharness` |
| IDE 主日志 | `~/Library/Logs/Antigravity/main.log` |
| IDE 语言服务日志 | `~/Library/Logs/Antigravity/language_server.log` |
| CLI 日志 | `~/.gemini/antigravity-cli/log/` |
| monocode 数据 | `~/Library/Application Support/com.monocode.desktop/monocode.db` |

### 日志判读

- 语言服务日志反复 `Get "https://www.googleapis.com/oauth2/v2/userinfo": EOF` / `net::ERR_CONNECTION_CLOSED` → **第一层**网络不通
- ACP 日志 `Localharness not found.` → **第三层**缺二进制
- ACP 日志 `Credentials missing or invalid. Launching browser login flow...` → **第二层**未登录
- ACP 日志 `Refreshing silently...` + `already onboarded` → 凭据正常，看后续

### monocode.db 排查

```bash
DB=~/Library/Application\ Support/com.monocode.desktop/monocode.db
# 看最近的会话用的哪个 harness、模型
sqlite3 "$DB" "SELECT datetime(updated_at/1000,'unixepoch','localtime'), harness, model FROM sessions ORDER BY updated_at DESC LIMIT 5;"
# 看失败会话的具体报错块
sqlite3 "$DB" "SELECT substr(blocks_json,1,2000) FROM sessions WHERE id='<session_id>';"
```

---

## 八、避坑清单

1. **只关窗口不算退出**：必须 `Cmd+Q`，否则语言服务 / ACP 子进程还在跑旧环境
2. **终端能跑 `agy` ≠ IDE 能用 ≠ monocode 能用**：终端走 `.zshrc`，IDE 走 launchd，monocode 的 ACP 还要独立登录
3. **"Authentication required" 别只当没登录**：先按第一/二/三层依次排查，别急着反复 `agy` 登录
4. **开机顺序**：先开 Clash Verge 再开 Antigravity / monocode，最稳
5. **换电脑 / 重装后**：代理脚本重新部署；钥匙串凭据不随文件迁移，需重新 `agy` 登录；ACP 的 `settings.json` + 浏览器登录 + localharness 都要重做
6. **localharness 必须与 `.par` 匹配**：升级 `google-antigravity` SDK 版本时留意，协议不兼容可能导致会话建不起来

---

## 相关笔记

- [[Mac_Terminal_配置_Clash_Verge_代理完整笔记]]（终端侧 `proxy_on/proxy_off` 方案，本篇是其 GUI 侧 + monocode 侧补充）
- [[使用国外AI教程]]
- [[macOS_Headscale_Tailscale_接入与避坑指南]]
