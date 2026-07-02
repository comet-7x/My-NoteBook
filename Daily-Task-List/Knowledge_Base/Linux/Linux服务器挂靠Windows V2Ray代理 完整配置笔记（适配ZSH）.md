## 一、核心原理

让远程 Linux 服务器外网流量，转发到本地 Windows V2Ray 代理出站，实现服务器访问 Claude / OpenAI / 所有海外资源。

正常工作链路：**Linux服务器 ➜ 局域网Windows(192.168.3.158:7890) ➜ 海外代理节点**

## 二、本机 Windows 必开条件（同局域网可用）

想要服务器走 Windows 代理，必须全部满足：

- V2RayN 正常运行、内核启动成功
- V2RayN 开启：**允许局域网连接**
- Windows 当前 WLAN 内网IP：`192.168.3.158`
- Windows 防火墙放行 7890 端口(TCP)
- 建议 V2Ray 切换为 **全局模式**（杜绝国内域名直连误判）

## 三、端口规范（固定不变）

- **7890**：Socks5 端口（Linux 统一使用，兼容性最强）
- 7891：HTTP 端口（不推荐，兼容差）

## 四、Linux ZSH 永久代理配置（仅当前用户生效）

你使用 **zsh**，配置写入`~/.zshrc`，新开终端 / 重连服务器自动生效。

**重要：仅你当前用户生效，root/其他用户不会继承代理**

```bash
# 写入永久代理
echo 'export all_proxy=socks5://192.168.3.158:7890' >> ~/.zshrc
echo 'export ALL_PROXY=socks5://192.168.3.158:7890' >> ~/.zshrc
echo 'export no_proxy=127.0.0.1,localhost,192.168.*,100.64.*' >> ~/.zshrc
echo 'export NO_PROXY=127.0.0.1,localhost,192.168.*,100.64.*' >> ~/.zshrc

# 立即生效
source ~/.zshrc
```

### 配置解释

- **大小写双写 all_proxy / ALL_PROXY**：兼容 curl、git、python、claude、openai 所有程序
- **no_proxy 内网豁免**：本机、局域网、Tailscale 流量不走代理，防止连代理机超时

## 五、正确代理验证方式

❌ 禁止使用 `cip.cc`（国内域名会被分流直连）

❌ 禁止频繁使用 `ipinfo.io`（容易 429 限流）

✅ **标准、稳定测试命令**（返回海外IP即为代理成功）

```bash
curl checkip.amazonaws.com
curl ifconfig.me
```

✅ Claude 接口连通测试（返回401即网络正常，仅缺密钥）

```bash
curl -I https://api.anthropic.com/v1/messages
```

## 六、临时关闭代理（Windows断网/关机必用）

Windows 离线后不取消代理，服务器全网外网超时。

```bash
unset http_proxy https_proxy all_proxy ALL_PROXY no_proxy NO_PROXY
```

## 七、关键问题：Windows 切换网络 / 不在同一局域网还能用吗？

### 场景1：换WiFi、同路由器、内网IP变了

**不能直接用，代理失效**

原因：你配置写死了 `192.168.3.158`，IP对不上连不上 V2Ray。

解决方法：

1. Windows 执行 `ipconfig` 查看新 WLAN IPv4
2. 修改 zsh 配置：`vim ~/.zshrc`
3. 替换所有 `192.168.3.158` 为新IP
4. `source ~/.zshrc` 重载即可恢复代理

### 场景2：Windows 和服务器 **完全不同局域网**（手机热点/异地网络）

**❌ 完全无法使用本地上网代理**

核心原因：`192.168.*` 是私有内网IP，公网服务器无法寻址、无法穿透路由访问你的家用电脑。

✅ 异地跨网替代方案（二选一）

#### 方案A：SSH反向隧道（临时最方便）

**✅ 你的结论完全正确**：可以全程使用 **SSH反向隧道方案**，彻底抛弃「同局域网限制」，是适配你长期使用的最优方案。

核心逻辑：**不管Windows在哪、什么网络、是否同局域网**，只要本机Windows开启V2Ray代理，打通反向隧道，服务器就能永久复用本机代理上网，完美解决换WiFi、异地网络代理失效问题。

**完整使用规则（反向隧道）**：

1. 前置条件：仅需 Windows 本机开启 V2Ray（无需开启局域网连接、无需放行防火墙端口）
2. 打通隧道（Windows 本地 CMD/PowerShell 执行，常驻窗口不能关）

```bash
ssh -R 7890:127.0.0.1:7890 服务器用户名@公网IP
```

3. 服务器侧统一代理配置（适配所有网络，永久生效）

```bash
# 清空旧的局域网代理（关键！）
unset http_proxy https_proxy no_proxy NO_PROXY

# 写入永久反向隧道代理（推荐覆盖zsh配置，彻底替代旧方案）
echo 'export all_proxy=socks5://127.0.0.1:7890' >> ~/.zshrc
echo 'export ALL_PROXY=socks5://127.0.0.1:7890' >> ~/.zshrc
source ~/.zshrc
```

**核心优势（对比局域网代理）**：

- 不受局域网限制：手机热点、异地网络、换WiFi 全部可用
- 无需修改IP：再也不用改 `.zshrc` 配置，一劳永逸
- 无需防火墙放行：规避局域网端口连通故障
- 开机即用：只要Windows开代理+打通隧道，服务器自动走代理

**唯一注意点**：反向隧道的SSH窗口需要保持后台运行，关闭窗口则代理断开，可搭配后台挂起命令实现常驻。

```bash
ssh -R 7890:127.0.0.1:7890 服务器用户名@公网IP
```

服务器侧临时改用本地代理：

```bash
export all_proxy=socks5://127.0.0.1:7890
export ALL_PROXY=socks5://127.0.0.1:7890
```

#### 方案B：服务器本地装Clash/V2Ray

彻底脱离Windows依赖，服务器自己常驻代理。

## 八、用户权限说明（非常重要）

- **仅当前用户生效**：配置在个人 `~/.zshrc`，root、其他用户无代理
- **sudo 会丢失代理**：sudo 默认清空环境变量
- 需要sudo走代理：        `sudo -E curl ifconfig.me`
- **不推荐全局代理**：一旦Windows关机，所有人外网炸断

## 九、各类工具生效规则

- **终端命令**：curl / git / claude / openai 自动走代理
- **VSCode Claude Code**：必须从当前代理终端 `code .` 启动才生效
- **Python 爬虫 / AI 调用**：自动读取系统 `ALL_PROXY`，无需代码写死代理

## 十、排错速查清单

- 端口不通：V2Ray未开「允许局域网连接」/ 防火墙未放行7890
- IP不变代理不走：V2Ray 规则模式分流，切全局模式
- 新开终端无代理：确认 zsh、source ~/.zshrc
- ipinfo 429：限流，换 amazon/ifconfig 测试
- 换WiFi后失效：修改 zshrc 内Windows内网IP并重载
- 异地网络：内网代理彻底失效，用反向隧道或服务器本地代理

## 十一、最终使用口诀

**同网开机即用，换网改IP重载，异地彻底失效，关机必须 unset**
