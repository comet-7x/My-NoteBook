---
title: 多设备使用 VSCode 远程开发配置自动代理笔记
date: 2026-07-29
tags:
  - Toolchain
  - VSCode
aliases: []
---

# 多设备使用 VSCode 远程开发配置自动代理笔记

## 一、目标与环境

我有 Mac 和 Windows 两台客户端，通过 Tailscale 连接同一台 Linux 开发服务器，希望：

- 使用 Mac 的 VSCode / Terminal 连接服务器时，服务器自动使用 Mac 上的代理；
    
- 使用 Windows 的 VSCode / CMD 连接服务器时，服务器自动使用 Windows 上的代理；
    
- 不需要每次登录服务器后手动 `export http_proxy=...`；
    
- VSCode 和普通 SSH 都能正确判断当前使用的是哪台客户端。

当前设备：

|设备|Tailscale IP|本地代理端口|
|---|--:|--:|
|MacBook|`100.64.0.40`|`7897`|
|Windows|`100.64.0.4`|`7890`|
|Linux Server|`100.64.0.15`|—|

最终链路：

```text
Mac(100.64.0.40:7897)
       │
       │ Tailscale
       │
       ▼
Linux Server(100.64.0.15) ——— 内部运行 .auto-proxy.zsh（自动识别客户端） 
       ▲
       │
       │ Tailscale
       │
Windows(100.64.0.4:7890)
```

核心原则：

> **客户端负责提供代理，Tailscale 负责设备互通，服务器负责根据当前客户端自动选择代理。**

---

# 第一步：配置 Clash Verge

## 1. 安装并确认 Clash Verge 正常工作

首先在客户端安装并启动 Clash Verge。

以 Mac 为例，确认系统代理：

```bash
scutil --proxy
```

例如：

```bash
<dictionary> {
  ExceptionsList : <array> {
    0 : 127.0.0.1
    1 : 192.168.0.0/16
    2 : 10.0.0.0/8
    3 : 172.16.0.0/12
    4 : localhost
    5 : *.local
    6 : *.crashlytics.com
    7 : <local>
  }
  FTPPassive : 1
  HTTPEnable : 1
  HTTPPort : 7897
  HTTPProxy : 127.0.0.1
  HTTPSEnable : 1
  HTTPSPort : 7897
  HTTPSProxy : 127.0.0.1
  ProxyAutoConfigEnable : 0
  SOCKSEnable : 1
  SOCKSPort : 7897
  SOCKSProxy : 127.0.0.1
}
```

说明当前 Clash 使用：

```text
127.0.0.1:7897
```

作为代理端口。

也可以：

```bash
sudo lsof -nP -iTCP:7897 -sTCP:LISTEN
```

查看真实监听情况。

---

## 2. 开启「允许局域网连接」

这里是整个方案非常关键的一步。

默认情况下 Clash 很可能只监听：

```text
127.0.0.1:7897
```

它意味着：

```text
Mac 自己
    ↓
127.0.0.1:7897
    ↓
Clash
```

可以正常使用。

但是 Linux Server 访问的是：

```text
Server
    ↓
100.64.0.40:7897
    ↓
Mac
```

如果 Clash 只监听 `127.0.0.1`，服务器即使能通过 Tailscale 找到 Mac，也无法使用这个代理。

所以需要在 Clash Verge 中打开：

```text
Allow LAN
允许局域网连接
```

对应配置通常类似：

```yaml
allow-lan: true
```

开启以后，再查看：

```bash
sudo lsof -nP -iTCP:7897 -sTCP:LISTEN
```

应该不再只是：

```text
127.0.0.1:7897
```

而能够接受其他网络接口进入的连接。

---

## 3. 为什么「Tailscale 能 ping 通」仍然可能无法使用代理？

这是这次排错过程中非常重要的一个经验。

服务器执行：

```bash
tailscale ping 100.64.0.40
```

可能得到：

```text
pong from cometmacbookpro
```

这只能证明：

```text
Server → Mac
```

网络可达。

但不能证明：

```text
Server → Mac:7897
```

也可达。

需要继续：

```bash
nc -vz 100.64.0.40 7897
```

因此应该区分三个层次：

```text
机器可达
↓
端口可达
↓
代理服务可用
```

分别对应：

```bash
tailscale ping 100.64.0.40

nc -vz 100.64.0.40 7897

curl -x http://100.64.0.40:7897 https://example.com
```

---

# 第二步：配置 VSCode

## 1. SSH 配置保持简单

Mac：

```text
~/.ssh/config
```

配置：

```sshconfig
Host steins-workspace
  HostName 100.64.0.15
  User zhihao

  IdentityFile ~/.ssh/steins_workspace_remote

  ConnectTimeout 10

  ServerAliveInterval 30
  ServerAliveCountMax 5

  TCPKeepAlive no
```

这里不需要再配置：

```sshconfig
RemoteForward ...
```

因为当前方案直接通过 Tailscale：

```text
Server
↓
100.64.0.40:7897
↓
Mac Clash
```

访问代理。

---

## 2. Mac VSCode 配置客户端身份

Mac VSCode 的 `settings.json`：

```json
{
  "terminal.integrated.env.linux": {
    "VSCODE_CLIENT_DEV": "mac"
  },

  "http.proxySupport": "off",

  "remote.SSH.useLocalServer": false
}
```

Windows 对应：

```json
{
  "terminal.integrated.env.linux": {
    "VSCODE_CLIENT_DEV": "win"
  }
}
```

这样 VSCode 新建远程终端以后：

Mac：

```bash
echo $VSCODE_CLIENT_DEV
```

应该得到：

```text
mac
```

Windows：

```text
win
```

---

## 3. 为什么不能只使用 `$SSH_CLIENT` 判断设备？

普通 SSH：

```bash
ssh steins-workspace
```

通常可以通过：

```bash
echo $SSH_CLIENT
```

得到：

```text
100.64.0.40 57268 22
```

因此普通 SSH 中：

```text
100.64.0.40 → Mac
100.64.0.4 → Windows
```

很好判断。

但是 VSCode Remote SSH 不只是：

```text
SSH → Shell
```

它大致存在这样的结构：

```text
VSCode Client
      │
      │ SSH
      ▼
Linux Server
      │
      └── VSCode Server
              │
              ├── Extension Host
              │
              └── Terminal
                     └── zsh
```

VSCode Server 是长期存在的后台进程。

因此可能出现：

```text
Mac 首先连接
↓
启动 VSCode Server
↓
VSCode Server 继承了 Mac 当时的 SSH 环境
```

之后 Windows 再连接时，某些远程进程、终端恢复或后台服务并不一定重新经历完整的 Shell 初始化。

曾经实际遇到过：

```bash
echo $SSH_CLIENT
```

在 Windows VSCode 中仍然显示：

```text
100.64.0.40 ...
```

也就是之前 Mac 的信息。

因此：

> `$SSH_CLIENT` 非常适合普通 SSH，但不应该成为 VSCode 多客户端环境中的唯一设备判断依据。

最终策略应该是：

```text
VSCode
↓
优先 VSCODE_CLIENT_DEV

普通 SSH
↓
使用 SSH_CLIENT
```

也就是：

```text
应用层明确声明身份
+
网络层作为 fallback
```

---

## 4. 为什么设置 `http.proxySupport = off`

以前遇到过另一个问题：

Mac Terminal SSH：

```bash
echo $SSH_CLIENT
```

正常显示 Mac Tailscale IP。

但 Mac VSCode Remote SSH 登录后，却出现异常来源信息。

因此对于：

```text
Tailscale / WireGuard / 内网 SSH
```

不希望 VSCode 自身的 HTTP 代理逻辑干扰远程连接。

所以设置：

```json
"http.proxySupport": "off"
```

同时 Clash 中建议确保：

```text
100.64.0.0/10
```

走 Direct。

需要注意：

> `http.proxySupport = off` 并不会给远程 Linux Server 配置代理。

真正设置服务器：

```text
http_proxy
https_proxy
all_proxy
```

的是后面的 `~/.auto-proxy.zsh`。

---

# 第三步：编写服务器自动代理脚本

服务器创建：

```bash
nano ~/.auto-proxy.zsh
```

内容：

```zsh
# ============================================================
# Mac / Windows 多设备自动代理
# ============================================================

WIN_TS_IP="100.64.0.4"
MAC_TS_IP="100.64.0.40"

WIN_PROXY_PORT=7890
MAC_PROXY_PORT=7897


_set_proxy() {
    local host="$1"
    local port="$2"

    export all_proxy="socks5h://${host}:${port}"
    export ALL_PROXY="${all_proxy}"

    export http_proxy="http://${host}:${port}"
    export HTTP_PROXY="${http_proxy}"

    export https_proxy="http://${host}:${port}"
    export HTTPS_PROXY="${https_proxy}"

    export socks5_proxy="socks5h://${host}:${port}"
}


_clear_proxy() {
    unset all_proxy ALL_PROXY
    unset http_proxy HTTP_PROXY
    unset https_proxy HTTPS_PROXY
    unset socks5_proxy SOCKS5_PROXY
}


auto_proxy_switch() {
    local client_ip
    client_ip=$(echo "${SSH_CLIENT}" | awk '{print $1}')

    # 内网地址不经过代理
    export no_proxy="127.0.0.1,localhost,192.168.0.0/16,100.64.0.0/10,.steins.net"
    export NO_PROXY="${no_proxy}"

    # ========================================================
    # 1. VSCode 环境：优先使用客户端主动声明的设备信息
    # ========================================================

    if [[ "${VSCODE_CLIENT_DEV}" == "mac" ]]; then
        _clear_proxy
        _set_proxy "${MAC_TS_IP}" "${MAC_PROXY_PORT}"
        return
    fi

    if [[ "${VSCODE_CLIENT_DEV}" == "win" ]]; then
        _clear_proxy
        _set_proxy "${WIN_TS_IP}" "${WIN_PROXY_PORT}"
        return
    fi

    # ========================================================
    # 2. 普通 SSH：根据 SSH 来源 IP 判断
    # ========================================================

    if [[ "${client_ip}" =~ ^192\.168\.3\. ]] || \
       [[ "${client_ip}" == "${WIN_TS_IP}" ]]; then

        _clear_proxy
        _set_proxy "${WIN_TS_IP}" "${WIN_PROXY_PORT}"
        return
    fi

    if [[ "${client_ip}" == "${MAC_TS_IP}" ]]; then

        _clear_proxy
        _set_proxy "${MAC_TS_IP}" "${MAC_PROXY_PORT}"
        return
    fi
}


auto_proxy_switch
```

---

## 为什么分成 `_set_proxy()` 和 `auto_proxy_switch()`？

不建议这样重复：

```zsh
export all_proxy=...
export http_proxy=...
export https_proxy=...

# Windows 再重复一遍

export all_proxy=...
export http_proxy=...
export https_proxy=...
```

更好的结构是：

```text
谁在连接？
↓
得到 host + port
↓
统一调用 _set_proxy()
```

这样以后修改：

```text
HTTP_PROXY
HTTPS_PROXY
ALL_PROXY
```

只需要修改一个地方。

---

## 为什么同时设置大小写变量？

因为不同程序读取代理环境变量的行为不完全一致。

所以同时设置：

```text
http_proxy
HTTP_PROXY

https_proxy
HTTPS_PROXY

all_proxy
ALL_PROXY
```

可以减少兼容性问题。

---

## 为什么使用 `socks5h`？

这里：

```bash
all_proxy=socks5h://...
```

相比：

```text
socks5://
```

多出来的 `h` 表示：

> 主机名解析也交给 SOCKS 代理完成。

因此：

```text
DNS
+
TCP
```

都可以通过代理端完成。

---

## 为什么 `100.64.0.0/10` 要加入 `NO_PROXY`

Tailscale 使用：

```text
100.64.0.0/10
```

范围。

因此：

```bash
export no_proxy=...,100.64.0.0/10,...
```

是为了避免：

```text
访问 Tailscale 内部设备
↓
反而再经过 Clash
```

造成不必要的代理绕路。

---

# 第四步：配置代理脚本加载位置

服务器使用 zsh，因此需要自动加载：

```text
~/.auto-proxy.zsh
```

以前使用：

```zsh
# ~/.zshrc
[ -f ~/.auto-proxy.zsh ] && source ~/.auto-proxy.zsh
```

后来调整为：

```zsh
# ~/.zshenv

[ -f ~/.auto-proxy.zsh ] && source ~/.auto-proxy.zsh
```

---

## 为什么从 `.zshrc` 移到 `.zshenv`

`.zshrc` 主要面向：

```text
交互式 zsh
```

也就是我们手动打开 Terminal 时。

但 Remote 开发工具除了交互式 Shell，还可能启动：

```text
后台命令
非交互 Shell
Remote Agent
```

因此只放：

```text
~/.zshrc
```

可能造成：

```text
VSCode Terminal curl 正常
```

但某些 Remote 后台工具：

```text
Codex
Remote Agent
后台 CLI
```

拿不到相同的代理环境。

`.zshenv` 的覆盖范围更广，所以这里选择：

```text
~/.zshenv
```

加载。

---

## 为什么脚本不能一启动就无条件 `_clear_proxy`

错误设计：

```zsh
auto_proxy_switch() {
    _clear_proxy

    # 然后再开始判断是谁
}
```

如果当前 Shell：

```text
既不是 VSCode
也没有 SSH_CLIENT
```

最终就会发生：

```text
先把原代理清空
↓
又没有识别到客户端
↓
代理彻底消失
```

因此现在的原则是：

> **识别成功以后，再清除旧代理并设置新代理。**

例如：

```zsh
if [[ "${VSCODE_CLIENT_DEV}" == "mac" ]]; then
    _clear_proxy
    _set_proxy ...
fi
```

---

# 第五步：验证代理联通性

配置完成以后，不应该直接：

```text
打开 Codex
↓
看看能不能用
```

这样出了问题很难判断是哪一层。

正确方式是逐层验证。

---

## 1. 验证设备识别

VSCode：

```bash
echo $VSCODE_CLIENT_DEV
```

Mac：

```text
mac
```

Windows：

```text
win
```

普通 SSH：

```bash
echo $SSH_CLIENT
```

例如 Mac：

```text
100.64.0.40 XXXXX 22
```

---

## 2. 验证代理环境变量

```bash
env | grep -iE 'http_proxy|https_proxy|all_proxy|no_proxy'
```

Mac 应该看到：

```text
http_proxy=http://100.64.0.40:7897
https_proxy=http://100.64.0.40:7897
all_proxy=socks5h://100.64.0.40:7897
```

Windows 应该变成：

```text
http_proxy=http://100.64.0.4:7890
https_proxy=http://100.64.0.4:7890
all_proxy=socks5h://100.64.0.4:7890
```

---

## 3. 验证 Tailscale

服务器：

```bash
tailscale ping 100.64.0.40
```

正常应该：

```text
pong from ...
```

但注意：

> ping 成功只代表设备可达，不代表代理可用。

---

## 4. 验证代理端口

服务器：

```bash
nc -vz 100.64.0.40 7897
```

正常：

```text
Connection to 100.64.0.40 7897 port [tcp/*] succeeded!
```

如果：

```text
ping ✅
nc ❌
```

优先检查 Clash：

```bash
sudo lsof -nP -iTCP:7897 -sTCP:LISTEN
```

特别注意是不是只监听：

```text
127.0.0.1:7897
```

如果是，就检查：

```text
Clash Verge
→ Allow LAN
```

---

## 5. 强制代理访问 OpenAI

服务器：

```bash
curl -x http://100.64.0.40:7897 \
  -I \
  --connect-timeout 10 \
  https://api.openai.com/v1/models
```

曾经成功得到：

```text
HTTP/1.1 200 Connection established

HTTP/2 401
```

这里：

```text
401 Unauthorized
```

并不是代理失败。

它说明：

```text
Server
↓
Tailscale
↓
Mac
↓
Clash
↓
OpenAI
```

整条网络链路已经成功。

只是这个 `curl` 请求没有提供 OpenAI API Key，所以 OpenAI 返回 `401`。

---

## 6. 最后才测试环境变量代理

取消显式 `-x`：

```bash
curl -I --connect-timeout 10 \
  https://api.openai.com/v1/models
```

如果仍然：

```text
HTTP/2 401
```

说明：

```text
.auto-proxy.zsh
↓
HTTP_PROXY
↓
Mac Clash
↓
OpenAI
```

自动代理完整生效。

这时候才测试：

```bash
codex
```

或：

```bash
git
pip
uv
npm
```

---

# 今天遇到的 Codex 问题

今天遇到的现象：

```text
Codex Remote SSH
Connected
```

但真正发送消息以后：

```text
Reconnecting... waiting for network
```

最初容易认为：

```text
Codex 与服务器 SSH 连接不稳定
```

但实际上：

```text
Mac
↓
SSH
↓
Server
```

完全正常。

失败的是另一条链路：

```text
Server
↓
100.64.0.40:7897
↓
Mac Clash
↓
OpenAI
```

服务器测试：

```bash
curl -I https://api.openai.com/v1/models
```

得到：

```text
Failed to connect to 100.64.0.40 port 7897
```

继续：

```bash
tailscale ping 100.64.0.40
```

却正常。

最终通过：

```bash
sudo lsof -nP -iTCP:7897 -sTCP:LISTEN
```

发现：

```text
verge-mihomo ... 127.0.0.1:7897
```

也就是说：

> Clash 本身正常，但只允许 Mac 自己访问，Linux Server 无法通过 `100.64.0.40:7897` 使用它。

这是今天真正的根因。

---

# 曾经尝试过但最终没有采用：SSH RemoteForward

排障期间曾使用：

```sshconfig
RemoteForward 17897 127.0.0.1:7897
```

把：

```text
Server 127.0.0.1:17897
```

通过 SSH 隧道转发到：

```text
Mac 127.0.0.1:7897
```

这样确实能成功：

```bash
curl -x http://127.0.0.1:17897 \
  -I https://api.openai.com/v1/models
```

得到：

```text
HTTP/2 401
```

证明网络正常。

但是后来又遇到了：

```text
remote port forwarding failed for listen port 17897
```

原因是：

```text
VSCode
Codex
Terminal
```

都可能创建独立 SSH 连接。

如果：

```sshconfig
RemoteForward 17897 ...
```

直接写在所有工具共用的：

```text
Host steins-workspace
```

里面，那么：

```text
第一条 SSH
→ 占用 17897

第二条 SSH
→ 再次绑定 17897
→ 失败
```

甚至在：

```sshconfig
ExitOnForwardFailure yes
```

存在时，整个 SSH 连接都会失败。

因此最终没有采用这个方案。

当前场景中：

```text
Tailscale + Clash Allow LAN
```

已经足够简单。

---

# 最终排障思路

以后如果再次出现：

```text
SSH 可以连
但 Codex / curl / git 无法联网
```

按照下面的顺序检查：

```text
① 当前客户端是谁？
        ↓
② 自动代理变量是否正确？
        ↓
③ Tailscale 是否能找到客户端？
        ↓
④ 客户端代理端口是否可达？
        ↓
⑤ Clash 是否正在正确接口监听？
        ↓
⑥ 强制指定代理能否访问 Internet？
        ↓
⑦ 自动代理能否访问 Internet？
        ↓
⑧ 最后才检查 Codex / VSCode / git
```

对应命令：

```bash
echo $VSCODE_CLIENT_DEV
echo $SSH_CLIENT

env | grep -i proxy

tailscale ping 100.64.0.40

nc -vz 100.64.0.40 7897

curl -x http://100.64.0.40:7897 \
  -I https://api.openai.com/v1/models

curl -I https://api.openai.com/v1/models
```

客户端：

```bash
sudo lsof -nP -iTCP:7897 -sTCP:LISTEN
```

---

# 最终经验

整个配置最终可以理解成四个独立职责：

```text
VSCode
↓
告诉服务器：我是谁

.auto-proxy.zsh
↓
根据设备选择哪个代理

Tailscale
↓
负责 Server 与 Mac / Windows 之间互通

Clash
↓
负责真正把服务器流量代理到 Internet
```

因此以后排错也应该遵循：

```text
身份识别
↓
代理变量
↓
网络可达
↓
端口可达
↓
代理服务
↓
外网
↓
具体应用
```

而不是看到：

```text
Codex Reconnecting
```

就直接认为 Codex 本身出了问题。

这次最重要的经验是：

> **SSH 能连接、Tailscale 能 ping、代理地址看起来正确，都不能单独证明代理链路真正可用。最终一定要验证“端口是否可达”和“代理程序到底监听在哪个网络接口”。**