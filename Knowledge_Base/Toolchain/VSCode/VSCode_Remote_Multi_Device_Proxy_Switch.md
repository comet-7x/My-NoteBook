---
title: 多设备远程开发：自动代理与排错笔记
date: 2026-07-29
tags:
  - Toolchain
  - VSCode
aliases: []
---
# 多设备远程开发：自动代理与排错笔记

## 0. 一句话目标

> 客户端只声明"我是谁"，服务器自动选对应代理；SSH 通 ≠ 应用能联网。

```text
Mac(100.64.0.40:7897)
       │ Tailscale
       ▼
Linux Server(100.64.0.15)
       │ 内部运行 .auto-proxy.zsh（自动识别客户端）
       ▲
       │ Tailscale
Windows(100.64.0.4:7890)
```

| 设备 | Tailscale IP | 本地代理 |
|---|---|---|
| MacBook | `100.64.0.40` | Clash Verge `:7897` |
| Windows | `100.64.0.4` | 代理 `:7890` |
| Linux Server | `100.64.0.15` | — |

---

## 1. 为什么不能直接用 `$SSH_CLIENT`

### 1.1 普通 SSH：可以

```bash
ssh zhihao@100.64.0.15   # 或 ssh steins-workspace
echo $SSH_CLIENT
# 100.64.0.40 57268 22
```

进程树：`sshd → zsh`，SSH 断开 zsh 就销毁。`SSH_CLIENT` 是 sshd 在这条 TCP 连接上注入的源 IP，**与当前 shell 一一对应**。

### 1.2 VS Code Remote SSH：不可靠

```
Local VS Code ──SSH──▶ Server
                        └─ vscode-server（常驻后台，一次拉起，多终端共享）
                             ├─ Extension Host
                             └─ Terminal 1 / 2 / 3  （都是它派生的 zsh）
```

关键点：
- `SSH_CLIENT` 只在**第一次建立 SSH 连接那一刻**注入给 `vscode-server` 主进程；
- 网络闪断重连后，旧 `vscode-server` 不会重启，环境变量不更新；
- 所有新终端继承的是"当年那个客户端"的 IP，不是"现在这个客户端"。

> 结论：普通 SSH 用 `$SSH_CLIENT`；VS Code / Codex / Claude Remote 这类常驻远程服务，必须由**客户端主动声明身份**。

---

## 2. 做法：客户端显式注入身份

### 2.1 VS Code 设置

Mac VS Code（`Cmd+,`）：

```json
{
  "terminal.integrated.env.linux": {
    "VSCODE_CLIENT_DEV": "mac"
  }
}
```

Windows VS Code（`Ctrl+,`）：

```json
{
  "terminal.integrated.env.linux": {
    "VSCODE_CLIENT_DEV": "win"
  }
}
```

> ⚠️ `terminal.integrated.env.linux` 只影响**集成终端**，不等于给整个 `vscode-server` / Extension Host 注入环境变量。Codex、Claude 这类远程 agent 如果不在集成终端里跑，需要在它们自己的配置里再传一次。

### 2.2 两级判断优先级

```
VSCODE_CLIENT_DEV 存在  → 用它（VS Code 集成终端）
否则                   → 回退 $SSH_CLIENT（普通 ssh / Terminal）
```

---

## 3. 服务器自动代理脚本

### 3.1 `~/.auto-proxy.zsh`

```zsh
# 客户端 Tailscale IP 与代理端口
WIN_TS_IP="100.64.0.4";  WIN_PROXY_PORT=7890
MAC_TS_IP="100.64.0.40"; MAC_PROXY_PORT=7897

_set_proxy() {
    local host="$1" port="$2"
    export all_proxy="socks5h://${host}:${port}" ALL_PROXY="$all_proxy"
    export http_proxy="http://${host}:${port}"  HTTP_PROXY="$http_proxy"
    export https_proxy="http://${host}:${port}" HTTPS_PROXY="$https_proxy"
}
_clear_proxy() {
    unset all_proxy ALL_PROXY http_proxy HTTP_PROXY https_proxy HTTPS_PROXY
}

auto_proxy_switch() {
    export no_proxy="127.0.0.1,localhost,192.168.0.0/16,100.64.0.0/10,.steins.net"
    export NO_PROXY="$no_proxy"

    local client_ip
    client_ip=$(echo "${SSH_CLIENT:-}" | awk '{print $1}')

    # 1) VS Code 显式声明
    if [[ "${VSCODE_CLIENT_DEV}" == "mac" ]]; then
        _clear_proxy; _set_proxy "$MAC_TS_IP" "$MAC_PROXY_PORT"; return
    elif [[ "${VSCODE_CLIENT_DEV}" == "win" ]]; then
        _clear_proxy; _set_proxy "$WIN_TS_IP" "$WIN_PROXY_PORT"; return
    fi

    # 2) 普通 SSH：按源 IP 回退
    if [[ "$client_ip" == "$WIN_TS_IP" || "$client_ip" =~ ^192\.168\.3\. ]]; then
        _clear_proxy; _set_proxy "$WIN_TS_IP" "$WIN_PROXY_PORT"
    elif [[ "$client_ip" == "$MAC_TS_IP" ]]; then
        _clear_proxy; _set_proxy "$MAC_TS_IP" "$MAC_PROXY_PORT"
    fi
}
auto_proxy_switch
```

### 3.2 加载位置：`~/.zshenv`，不是 `~/.zshrc`

```zsh
# ~/.zshenv
[ -f ~/.auto-proxy.zsh ] && source ~/.auto-proxy.zsh
```

| 文件 | 作用范围 |
|---|---|
| `.zshrc` | 仅交互式 shell |
| `.zshenv` | **所有** zsh（包括非交互、scp、远程脚本） |

> 因为放到了 `.zshenv`，脚本不能"一启动就 `_clear_proxy`"——必须识别到客户端再改，否则会把 cron、非交互任务里的代理也清掉。

---

## 4. 案例：Codex 一直 `Reconnecting... waiting for network`

### 4.1 拆成两条链路

```
链路 A：Mac  ──SSH──▶ Server          （这条是通的）
链路 B：Server ──Proxy──▶ Mac Clash ──▶ OpenAI   （这条断了）
```

**"SSH Connected" 不代表远程 Codex 能访问 OpenAI。**

### 4.2 逐层下钻

```bash
# ① 当前代理是谁？
env | grep -i proxy
# → http_proxy=http://100.64.0.40:7897   （选对了 Mac）

# ② Tailscale 机器可达？
tailscale ping 100.64.0.40
# → pong from cometmacbookpro   ✅

# ③ TCP 端口可达？
nc -vz 100.64.0.40 7897
# → ❌ connection refused

# ④ Mac 上代理监听在哪？（在 Mac 本地执行）
sudo lsof -nP -iTCP:7897 -sTCP:LISTEN
# → verge-mihomo ... TCP 127.0.0.1:7897 (LISTEN)
```

### 4.3 根因

Clash Verge 只监听 `127.0.0.1:7897`——**只有 Mac 本机能用**。Server 走 Tailscale IP `100.64.0.40:7897` 当然连不上。

修复：Clash Verge 打开 **Allow LAN / 允许局域网连接**，让它监听 `0.0.0.0:7897`（或 Tailscale 网卡）。

> 安全提示：开了 Allow LAN 不代表只有 Server 能访问。配合 Tailscale ACL 或系统防火墙，把来源限制在 Server 的 Tailscale IP。

---

## 5. 踩过的反方案：SSH `RemoteForward`

为了绕过 Clash 只监听 localhost，曾试：

```sshconfig
Host steins-workspace
    RemoteForward 17897 127.0.0.1:7897
    ExitOnForwardFailure yes
```

原理：Server 的 `127.0.0.1:17897` 通过 SSH 隧道打到 Mac 的 `127.0.0.1:7897`。

验证确实通：
```bash
curl -x http://127.0.0.1:17897 -I https://api.openai.com/v1/models
# HTTP/1.1 200 Connection established
# HTTP/2 401   ← 没带 API Key，正常；网络层已通
```

**但踩了第二个坑**：VS Code、Codex、Terminal 各自建立独立 SSH 连接，第二条连接再绑 `:17897` 会失败：

```
Authenticated to 100.64.0.15 using "publickey"
Error: remote port forwarding failed for listen port 17897
```

配合 `ExitOnForwardFailure yes`，整条 SSH 直接被拒。

> 这条报错很值钱：`Authenticated` 说明密钥认证没问题，**真正失败的是 RemoteForward**。看完整日志，别被 UI 上的 "SSH connection failed" 带偏。

长期用 RemoteForward 需要单独维护一条专用隧道（`steins-workspace-proxy`），对当前需求过度设计。最终回到 Tailscale + Allow LAN 方案。

---

## 6. 固定排障流程（下次照抄）

症状：SSH 正常，但 Codex / curl / git / pip 上不了外网。

```bash
# 1. SSH 本身通不通
ssh steins-workspace

# 2. 服务器认为自己是谁
echo $VSCODE_CLIENT_DEV
echo $SSH_CLIENT

# 3. 当前代理变量
env | grep -i proxy

# 4. 机器可达？
tailscale ping 100.64.0.40

# 5. 端口可达？
nc -vz 100.64.0.40 7897

# 6. 客户端代理监听在哪？（在客户端本机跑）
sudo lsof -nP -iTCP:7897 -sTCP:LISTEN

# 7. 强制指定代理测真实出口
curl -x http://100.64.0.40:7897 -I https://api.openai.com/v1/models

# 8. 最后才回到应用本身
codex / git / pip / uv
```

对应心智模型：

```
SSH → 身份识别 → 环境变量 → Tailscale → TCP Port → Proxy → Internet → Codex
```

**从底往上查，不要从 Codex 往回猜。**

---

## 7. VS Code Server 状态异常时

`Remote 环境变量不对 / 终端残留旧环境 / 扩展宿主抽风`：

1. `Cmd/Ctrl+Shift+P` → `Remote-SSH: Kill VS Code Server on Host...`
2. 完全退出 VS Code 再重连。

命令行兜底：
```bash
ps aux | grep vscode
pkill -f vscode-server
```

只有确认 `~/.vscode-server` 安装损坏才 `rm -rf`，那会触发远程扩展重装，不要当日常手段。

---

## 8. 最终原则

> **客户端声明身份，服务器选择代理，Tailscale 负责互通，代理软件负责真的把端口开出来。**

四条经验压成一行：
1. SSH 通 ≠ 应用层通——这是两条链路。
2. IP 能 ping ≠ 端口能连——还要看服务监听在哪个网卡。
3. 先 `tailscale ping` → `nc -vz` → `lsof` → `curl -x`，再怀疑应用。
4. 修最小问题（开 Allow LAN），别上来就重构成 RemoteForward + 专用隧道。
