# 在 WSL 上进行 Hailo8 的  onnx 到 hef 的转换

本文参考链接：

>[Windows Subsystem for Linux 文档 | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/)
>
>[使用 WSL 运行 Linux GUI 应用 | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/gui-apps)
>
>[请问下大家，怎么修改配置文件，使得我的 v2rayN 不止能监听本地回环地址 · 2dust/v2rayN · Discussion #6844](https://github.com/2dust/v2rayN/discussions/6844)
>
>
>[开始使用 WSL 来体验 Linux | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/linux)
>
>[什么是适用于 Linux 的 Windows 子系统 | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/about)
>



## 1. 安装 WSL：

### 1.1 了解 WSL 与 Linux：

这里给出连接供读者阅读，笔者认为对于入门非常有用：

[什么是适用于 Linux 的 Windows 子系统 | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/about)

[开始使用 WSL 来体验 Linux | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/linux)

本文很多部分都参考了微软提供的学习教程，我强烈建议你前去官网 [Windows Subsystem for Linux 文档 | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/) 学习。



### 1.2 安装条件：

必须运行 Windows 10 版本 2004 及更高版本（内部版本 19041 及更高版本）或 Windows 11

> 注意：若要检查 Windows 版本及内部版本号，选择 Windows 徽标键 + R，然后键入“winver”，选择“确定” 。

Win+R 输入 cmd：

![image-20250703204902722](https://s2.loli.net/2025/07/03/ioPUcZkaqExhOWS.png)

执行 `optionalfeatures` 打开「Windows 功能」，选择我这里框住的部分。



### 1.3 从未安装：

现在可以在管理员 PowerShell 或 Windows 命令提示符中输入此命令，然后重启计算机，安装运行适用于 Linux 的 Windows 子系统（WSL）所需的所有内容。

```powershell
wsl --install
```

这个命令将启用运行 WSL 所需的特性，并安装 Linux 的 Ubuntu 发行版。（这个默认发行版可以更改）。



### 1.4 已安装需更新：

选择 **“开始**”，键入 **PowerShell**，右键单击 **Windows PowerShell**，然后选择“ **以管理员身份运行**”。

输入 WSL 更新命令：

```powershell
wsl --update
```

需要重启 WSL 才能使更新生效。 可以通过在 PowerShell 中运行关闭命令来重启 WSL。

```
wsl --shutdown
```

> 注意：Linux GUI 应用仅受 WSL 2 支持，不适用于为 WSL 1 配置的 Linux 分发版。 了解如何 [将分发版从 WSL 1 更改为 WSL 2](https://learn.microsoft.com/zh-cn/windows/wsl/basic-commands#set-wsl-version-to-1-or-2)。

你可以通过输入命令 `wsl -l -v` 在 PowerShell 或 Windows 命令提示符中列出已安装的 Linux 发行版，并检查每个 WSL 设置的版本。



安装 WSL 的详细教程参照：https://learn.microsoft.com/en-us/windows/wsl/install



## 2. 安装 Linux 系统：



### 2.2 进入 Linux 系统：

```shell
wsl -d Ubuntu
```

如果可以成功进入，那么我们使用 `exit` 退出 WSL Ubuntu 会话。



### 2.3 移动 WSL：

WSL 默认安装在 C 盘，它的文件系统会随着你安装软件、下载文件而逐渐增大，这会导致 C 盘空间紧张。将它移动到其他盘可以有效解决这个问题。



#### **① 检查 WSL 发行版状态并关闭它：**

在 Windows 的 PowerShell 或命令提示符中，首先确认你的 Ubuntu 发行版名称，并确保它已关闭（处于 `Stopped` 状态）。

```shell
wsl -l -v
```

```shell
# 方法一
C:\Users\19334>wsl -l -v
  NAME              STATE           VERSION
* docker-desktop    Stopped         2
  Ubuntu            Running         2
```

```shell
# 方法二
C:\Users\19334>wsl --list --verbose
  NAME              STATE           VERSION
* docker-desktop    Stopped         2
  Ubuntu            Stopped         2
```



我可以看到安装的 Ubuntu 处于 `Running` 状态，使用：

```shell
# 方法一，关闭一个实例
wsl --terminate Ubuntu
```

```shell
# 还有一种方式可以关闭全部实例
wsl --shutdown
```

关闭系统。



#### **② 导出 WSL Ubuntu 到一个文件：**

```shell
# 示例：将导出文件放在 E:\1_Learning_Record\17_WSL\WSL_Backup\ubuntu.tar
# 请根据你的实际情况修改路径
C:\Users\19334>wsl --export Ubuntu E:\1_Learning_Record\17_WSL\WSL_Backup\ubuntu.tar
正在导出，这可能需要几分钟时间。 (1477 MB)

操作成功完成。
```

**注意：** 这里的 `Ubuntu` 是你的发行版名称（根据 `wsl -l -v` 的结果来）。`E:\1_Learning_Record\17_WSL\WSL_Backup` 只是一个示例路径，你可以选择任何你希望的目录来存放这个临时的 `tar` 文件。这个过程可能需要一些时间，取决于你的 WSL 大小。



#### **③ 注销当前的 WSL Ubuntu 发行版** ：

导出成功后，你可以安全地注销（卸载）C 盘上当前的 Ubuntu 发行版。这会释放 C 盘上的空间。

```shell
C:\Users\19334>wsl --unregister Ubuntu
正在注销。
操作成功完成。
```

**警告：** 这一步会删除 C 盘上原有 WSL Ubuntu 的所有数据，所以务必确认第 3 步的导出已成功完成。



#### **④ 将 WSL Ubuntu 导入到新的位置**：

现在，你可以将之前导出的 `.tar` 文件导入到你希望的新盘符和新文件夹。

```shell
# 示例：将 Ubuntu 导入到 E:\1_Learning_Record\17_WSL\WSL_Instances\Ubuntu
# 请根据你的实际情况修改路径
wsl --import Ubuntu E:\1_Learning_Record\17_WSL\WSL_Instances\Ubuntu E:\1_Learning_Record\17_WSL\WSL_Backup\ubuntu.tar --version 2
操作成功完成。
```

**解释：**

- `Ubuntu`: 新导入发行版的名称。你可以保持和原来一样。
- `E:\1_Learning_Record\17_WSL\WSL_Instances\Ubuntu`: 这是你希望将 WSL Ubuntu 文件系统实际存放的新路径。**务必确保这个文件夹在执行命令前是空的或者不存在**，WSL 会自动创建它。
- `E:\1_Learning_Record\17_WSL\WSL_Backup\ubuntu.tar`: 这是你之前导出的 `.tar` 文件的完整路径。
- `--version 2`: 强烈建议指定 WSL 2 版本，以获得更好的性能和兼容性。



**（可选）删除临时的导出文件** 导入成功后，你可以删除之前创建的 `E:\1_Learning_Record\17_WSL\WSL_Backup\ubuntu.tar` 文件，因为你已经不再需要它了。



#### **⑤ 启动新的 WSL Ubuntu：**

现在，你可以通过以下命令启动你的 WSL Ubuntu：

```sh
wsl -d Ubuntu
```

它会从你指定的新位置加载，并且你的所有数据、用户、配置都会和之前一样。


### 2.4 新增用户：
#### ① 创建用户
```bash
adduser 你的用户名
```
系统会提示你输入密码（输入时屏幕不显示，输完回车即可），剩下的全名等信息直接一路回车。

#### ② 给 新用户 管理员权限：
```bash
usermod -aG sudo 你的用户名
```

#### ③ 退出并重启 WSL：
```bash
exit
```
然后在 Windows 终端执行：
```bash
wsl --shutdown
wsl
```

示例：
```bash
```bash
root@localhost:~# adduser comet
info: Adding user `comet' ...
info: Selecting UID/GID from range 1000 to 59999 ...
info: Adding new group `comet' (1000) ...
info: Adding new user `comet' (1000) with group `comet (1000)' ...
info: Creating home directory `/home/comet' ...
info: Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for comet
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y
info: Adding new user `comet' to supplemental / extra groups `users' ...
info: Adding user `comet' to group `users' ...
root@localhost:~# usermod -aG sudo comet
root@localhost:~# exit
logout


```
```

## 3. 在 WSL 中使用 GPU：

### 3.1 检查GPU是否可用

#### **① 更新软件包列表：**

```shell
sudo apt-get update
sudo apt-get upgrade
```



#### **② 安装 `mesa-utils` 和 `libgl1-mesa-glx` (通常用于基本的 OpenGL/Vulkan 测试)：**

```shell
sudo apt install mesa-utils libglx-mesa0
```



#### **③ 验证 GPU 是否被识别 (通用检查)：**

```shell
glxinfo | grep -i "opengl renderer"
```

```shell
pi@localhost:/mnt/c/Users/19334$ glxinfo | grep -i "opengl renderer"
OpenGL renderer string: D3D12 (NVIDIA GeForce RTX 3060 Laptop GPU)
```



## 4. WSL 共享 Windows 系统的网络代理：


### 4.1 镜像网络模式 (Mirrored Mode)：
这是微软在 2023 年底推出的新特性，最推荐使用。它让 WSL2 和 Windows 彻底处于同一个网络层级，WSL2 会直接继承 Windows 的代理设置，连 IP 地址都和 Windows 一模一样。
#### ① 操作步骤
1. 在 Windows 中，按下 `Win + R`，输入 `%UserProfile%` 并回车。
2. 在打开的文件夹中新建一个文件，命名为 `.wslconfig`（如果已有则直接打开）。
3. 粘贴以下带注释的代码：
```Ini
[wsl2]
# 开启镜像网络模式，让 WSL 共享 Windows 的网络堆栈
networkingMode=mirrored

# 开启 DNS 隧道，让 WSL 使用 Windows 的 DNS 解析，解决 DNS 污染
dnsTunneling=true

# 自动应用 Windows 的代理设置到 WSL
autoProxy=true

# 允许 WSL 穿透 Windows 防火墙（镜像模式必需）
firewall=true
```

4. **保存并重启**：在 PowerShell 中执行 `wsl --shutdown`，然后再次打开 Ubuntu。

#### ② 优点与局限
- **优点**：配置简单，几乎不需要在 Linux 内部改任何代码；解决了很多网络协议（如 UDP、IPv6）的兼容性问题。
- **局限**：需要 Windows 11 22H2 以上版本，且 WSL 内核版本较新。


### 4.2 环境变量手动模式 (Environment Variables)：
如果你的系统不支持镜像模式，或者你希望通过命令手动控制开关，使用这种方法。它通过设置 `http_proxy` 等环境变量引导 Linux 流量走向宿主机代理端口。
#### ① 前置条件 (重要)
- **允许局域网连接**：在你的代理软件（如 Clash/V2ray/小火箭）设置中，必须打开 **"Allow LAN"** 或 **“允许局域网连接”**。
- **防火墙放行**：确保 Windows 防火墙没有拦截你的代理软件。


#### ② 配置步骤：
打开你的 **Zsh** 配置文件：`nano ~/.zshrc`（如果你用的是默认 Bash，则打开 `nano ~/.bashrc`），在文件末尾添加以下内容：
```bash
# --- WSL 代理配置开始 ---

# 1. 动态获取宿主机的 IP 地址
# 因为每次重启后宿主机 IP 可能会变，所以需要从 /etc/resolv.conf 自动提取
export hostip=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}')

# 2. 设置代理软件的端口（请根据你软件的设置修改，通常是 7890, 1080, 或 10809）
export hostport=7890 

# 3. 定义一键开启代理的命令 (alias)
alias proxy="
    export https_proxy=http://${hostip}:${hostport};
    export http_proxy=http://${hostip}:${hostport};
    export all_proxy=socks5://${hostip}:${hostport};
    echo '代理已开启：已指向 http://${hostip}:${hostport}'
"

# 4. 定义一键关闭代理的命令
alias unproxy="
    unset https_proxy;
    unset http_proxy;
    unset all_proxy;
    echo '代理已关闭'
"

# --- WSL 代理配置结束 ---
```

#### ③ 刷新配置：
执行 `source ~/.zshrc`。



#### ④ 手动开关：
- 需要安装 `uv` 或 `vllm` 时，输入 `proxy`。
- 不需要时，输入 `unproxy`。


代理验证：
```bash
# 如果看到一串网页 HTML 代码或成功字样，说明通了
curl -vv https://www.google.com
```

### 4.1 修改软件配置：

我这里使用的是 `V2rayN` 这款代理软件：

步骤：打开软件  -->  打开自动配置系统代码  -->  打开设置，选择参数设置

![image-20250702214636230](https://s2.loli.net/2025/07/02/tLqbSOUjPXGCenH.png)

按照如图配置：
1.设置本地混合监听端口为 `10810`，等会要用。

2.打开允许来自局域网的连接。



同时我们要保证在 Windows 中验证端口监听（Win+R 输入 CMD）：
输入命令 `netstat -ano | findstr :10810`：

```powershell
(base) PS C:\Users\19334> netstat -ano | findstr :10810
  TCP    0.0.0.0:10810          0.0.0.0:0              LISTENING       38616
  TCP    127.0.0.1:10810        127.0.0.1:61719        TIME_WAIT       0
  TCP    127.0.0.1:10810        127.0.0.1:62010        TIME_WAIT       0
  TCP    127.0.0.1:10810        127.0.0.1:62012        TIME_WAIT       0
  TCP    127.0.0.1:10810        127.0.0.1:62056        TIME_WAIT       0
  TCP    127.0.0.1:10810        127.0.0.1:62065        TIME_WAIT       0
  TCP    127.0.0.1:10810        127.0.0.1:62083        TIME_WAIT       0
  TCP    127.0.0.1:10810        127.0.0.1:62084        TIME_WAIT       0
  TCP    127.0.0.1:10810        127.0.0.1:62086        TIME_WAIT       0
  TCP    127.0.0.1:10810        127.0.0.1:62088        TIME_WAIT       0
  TCP    127.0.0.1:10810        127.0.0.1:62114        TIME_WAIT       0
  TCP    127.0.0.1:10810        127.0.0.1:62115        TIME_WAIT       0
  TCP    127.0.0.1:10810        127.0.0.1:62116        TIME_WAIT       0
  TCP    127.0.0.1:10810        127.0.0.1:62117        TIME_WAIT       0
  TCP    127.0.0.1:10810        127.0.0.1:62123        ESTABLISHED     38616
  TCP    127.0.0.1:10810        127.0.0.1:62131        ESTABLISHED     38616
  TCP    127.0.0.1:10810        127.0.0.1:62134        ESTABLISHED     38616
  TCP    127.0.0.1:10810        127.0.0.1:62137        ESTABLISHED     38616
  TCP    127.0.0.1:10810        127.0.0.1:62149        ESTABLISHED     38616
  TCP    127.0.0.1:10810        127.0.0.1:62151        ESTABLISHED     38616
  TCP    127.0.0.1:62052        127.0.0.1:10810        TIME_WAIT       0
  TCP    127.0.0.1:62123        127.0.0.1:10810        ESTABLISHED     15440
  TCP    127.0.0.1:62124        127.0.0.1:10810        TIME_WAIT       0
  TCP    127.0.0.1:62129        127.0.0.1:10810        TIME_WAIT       0
  TCP    127.0.0.1:62131        127.0.0.1:10810        ESTABLISHED     21036
  TCP    127.0.0.1:62134        127.0.0.1:10810        ESTABLISHED     21036
  TCP    127.0.0.1:62136        127.0.0.1:10810        TIME_WAIT       0
  TCP    127.0.0.1:62137        127.0.0.1:10810        ESTABLISHED     42564
  TCP    127.0.0.1:62149        127.0.0.1:10810        ESTABLISHED     21036
  TCP    127.0.0.1:62151        127.0.0.1:10810        ESTABLISHED     15440
  TCP    [::]:10810             [::]:0                 LISTENING       38616
  UDP    0.0.0.0:10810          *:*                                    38616
  UDP    [::]:10810             *:*                                    38616
```

必须看到：

```
TCP    0.0.0.0:10810          0.0.0.0:0              LISTENING       38616 # 38616是V2RayN的进程ID
```



### 4.2 在 WSL 里编写代理脚本：

#### **① 首先，进入当前用户的家目录：**

```shell
cd ~
```

```shell
pi@localhost:/mnt/c/Users/19334$ cd ~
pi@localhost:~$
```



#### **② 编写 `proxy.sh` 脚本：**

```shell
vim ~/proxy.sh
```

```shell
pi@localhost:~$ vim ~/proxy.sh
```

 **粘贴以下内容：**

```sh
#!/bin/sh
# 核心配置：Windows的IP（或用localhost）和V2RayN的SOCKS5端口
hostip="localhost"  # 用localhost避免IP冲突（WSL与Windows共享回环）
port=10810          # 替换为V2RayN的实际端口（必须与V2RayN一致）

# SOCKS5代理地址（匹配V2RayN的协议）
PROXY_SOCKS="socks5://${hostip}:${port}"

set_proxy(){
    # 清除旧代理，只设置必要的变量
    unset http_proxy https_proxy ALL_PROXY
    export http_proxy="${PROXY_SOCKS}"
    export https_proxy="${PROXY_SOCKS}"
    export ALL_PROXY="${PROXY_SOCKS}"
    echo "代理已设置为: ${PROXY_SOCKS}"
}

unset_proxy(){
    unset http_proxy https_proxy ALL_PROXY
    echo "代理已取消"
}

test_proxy(){
    echo "当前代理: $https_proxy"
    echo -e "\n测试谷歌访问（通过代理）:"
    curl -s -m 5 https://www.google.com -o /dev/null && echo "✅ 成功" || echo "❌ 失败"
}

# 根据参数执行对应功能
case "$1" in
    set) set_proxy ;;
    unset) unset_proxy ;;
    test) test_proxy ;;
    *) echo "用法: $0 [set|unset|test]" ;;
esac
```



#### **③ 保存并添加执行权限：**

```shell
chmod +x ~/proxy.sh
```



#### **④ 使用：**

##### **1.设置代理：**

```shell
# 用source执行脚本（. 是source的简写）
. ./proxy.sh set  # 或 source ./proxy.sh set
```

```shell
pi@localhost:~$ ./proxy.sh set
代理已设置为: socks5://localhost:10810
```

设置完成后我们可以查看一下代理是否设置成功：

```shell
pi@localhost:~$ echo $https_proxy
socks5://localhost:10810
```



##### **2.测试代理连通性：**

```shell
./proxy.sh test
# 若显示“✅ 成功”，说明代理基本正常
```

```
pi@localhost:~$ . ./proxy.sh test
当前代理: socks5://localhost:10810

测试谷歌访问（通过代理）:
✅ 成功
```



##### **3.取消代理：**

```shell
# 用source执行脚本（. 是source的简写）
. ./proxy.sh unset  # 或 source ./proxy.sh unset
```

```shell
pi@localhost:~$ . ./proxy.sh unset
代理已取消
```



#### **⑤ 手动修改 DNS，优先使用公共 DNS（解决域名解析失败）：**

由于 WSL 会自动覆盖 `/etc/resolv.conf`，需先禁用自动生成，再手动设置 DNS：

1. **创建 `/etc/wsl.conf`，禁用自动生成 `resolv.conf`**

```bash
sudo vim /etc/wsl.conf
```

添加以下内容（阻止 WSL 自动修改 DNS）：

```ini
[network]
generateResolvConf = false  # 关键：禁用自动生成resolv.conf
```

2. **删除旧的 `resolv.conf`，创建新文件**

```bash
sudo rm /etc/resolv.conf  # 删除自动生成的文件
sudo vim /etc/resolv.conf  # 新建手动配置的文件
```

3. **添加公共 DNS（推荐谷歌或阿里云 DNS）**

在新的 `resolv.conf` 中添加：

```ini
nameserver 8.8.8.8        # 谷歌公共DNS（全球通用，需代理可访问）
nameserver 223.5.5.5      # 阿里云公共DNS（国内访问稳定）
search wifi               # 保留原搜索域，不影响本地网络
```

4. **重启 WSL 使 DNS 生效（Windows PowerShell 中执行）**

```powershell
wsl --shutdown  # 完全关闭WSL
wsl -d Ubuntu   # 重新启动WSL
```

5. **备份原有软件源列表**

```bash
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak  # 备份防止出错
```

6. **编辑软件源列表，替换为阿里云源**

```bash
sudo vim /etc/apt/sources.list
```

删除文件中所有内容，粘贴以下阿里云源（适用于 Ubuntu 24.04，代号 `noble`）：

```ini
# 阿里云Ubuntu 24.04 (noble) 源
deb http://mirrors.aliyun.com/ubuntu/ noble main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ noble main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ noble-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ noble-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ noble-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ noble-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ noble-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ noble-backports main restricted universe multiverse
```

7. **更新源列表并测试**

```bash
sudo apt update  # 刷新源列表，此时应能连接到阿里云源
```

- 若成功，会显示 “获取” 阿里云源的包信息，无 “无法连接” 错误。

- 之后即可安装所需工具（如 `nslookup`、`polipo`）

  ```bash
  sudo apt install -y bind9-dnsutils  # 安装nslookup
  sudo apt install -y privoxy  # 安装SOCKS5转HTTP工具（如需）
  ```

8. **配置 `privoxy` 转发 SOCKS5 代理**

```bash
sudo vim /etc/privoxy/config
```

在文件末尾添加以下内容（将 SOCKS5 代理转为 HTTP 代理）：

```conf
# 监听WSL的8119端口（作为HTTP代理出口，privoxy默认端口）
listen-address 0.0.0.0:8119

# 转发到V2RayN的SOCKS5代理（localhost:10810）
forward-socks5 / localhost:10810 .  # 注意末尾的点不能省略
```

 **验证端口是否可用**：

```shell
# 检查8119端口是否被占用（应无输出）
sudo lsof -i :8119
```

 **启动 `privoxy` 并验证**

```shell
sudo privoxy --no-daemon /etc/privoxy/config
```

正常输出（无错误，显示监听 8119 端口）：

```shell
2025-07-02 22:51:30.357 7f05262d3b80 Info: Privoxy version 3.0.34
2025-07-02 22:51:30.357 7f05262d3b80 Info: Program name: privoxy
```

此时按 `Ctrl+C` 停止前台运行，改用 `systemd` 后台启动：

```shell
sudo systemctl start privoxy
sudo systemctl status privoxy  # 应显示active (running)
```

```shell
pi@localhost:/mnt/c/Users/19334$ sudo systemctl status privoxy
● privoxy.service - Privacy enhancing HTTP Proxy
     Loaded: loaded (/usr/lib/systemd/system/privoxy.service; enabled; preset: enabled)
     Active: active (running) since Wed 2025-07-02 22:51:48 CST; 6s ago
       Docs: man:privoxy(8)
             https://www.privoxy.org/user-manual/
    Process: 1499 ExecStart=/usr/sbin/privoxy --pidfile $PIDFILE --user $OWNER $CONFIGFILE (code=exited, status=0/SUCCESS)
   Main PID: 1500 (privoxy)
      Tasks: 1 (limit: 9436)
     Memory: 1.6M ()
     CGroup: /system.slice/privoxy.service
             └─1500 /usr/sbin/privoxy --pidfile /run/privoxy.pid --user privoxy /etc/privoxy/config

Jul 02 22:51:47 localhost systemd[1]: Starting privoxy.service - Privacy enhancing HTTP Proxy...
Jul 02 22:51:48 localhost systemd[1]: Started privoxy.service - Privacy enhancing HTTP Proxy.
```

9. **重启 `privoxy` 生效**

```bash
sudo systemctl restart privoxy
```

10. **为 `apt` 配置 HTTP 代理（通过 `privoxy` 中转）**

```bash
sudo vim /etc/apt/apt.conf.d/proxy.conf
```

添加以下内容（使用 `privoxy` 提供的 HTTP 代理）：

```ini
Acquire::http::Proxy "http://localhost:8119";
Acquire::https::Proxy "http://localhost:8119";
```

11. **验证 `apt` 是否能通过代理更新**

```bash
sudo apt update
```

若成功，说明 `privoxy` 已正常转发 SOCKS5 代理，`apt` 可通过 HTTP 代理访问网络。







## 5. 安装有用的软件：

### 5.1 安装 X-server：

这个软件是安装在 Windows 上的，用于打开 WSL 上的任何窗口，例如 Chrome 浏览器，命令行窗口等。

#### **① 从 VcXsrv Windows X Server 下载并安装 VxXsrv**

[VcXsrv Windows X Server 下载 | SourceForge.net --- VcXsrv Windows X Server download | SourceForge.net](https://sourceforge.net/projects/vcxsrv/)



#### **② 创建桌面快捷方式：**

安装完成后，创建一个新的桌面快捷方式，指向 C:\Program Files\VcXsrv\vcxsrv.exe，并在快捷方式的属性中的目标字段中设置以下命令： `"C:\Program Files\VcXsrv\vcxsrv.exe" :0 -ac -terminate -lesspointer -multiwindow -clipboard -wgl -dpi auto`



#### **③ 双击快捷方式以启动 X 服务器：**



#### **④ 通过 Windows 开始菜单启动 WSL-Ubuntu 终端并编辑.bashrc 文件：**

```bash
wsl -d Ubuntu
vim .bashrc
```

按“i”键，添加以下行：

```ini
export DISPLAY="172.28.112.1:0"
```

注意：你需要使用 `ipconfig` 查看你电脑上的 IP 地址，然后替换为你的。



#### **⑤ 为测试目的，安装 terminator，它是一个类似于 xterm 的终端模拟器：**

```bash
sudo apt-get install terminator
```

通过启动一个终端窗口来测试 xserver：

```bash
terminator &
```



### 5.2 安装 Google-chrome：

在 window 里打开 Chrome 网站：https://www.google.com/chrome/

翻到最底部：

![image-20250703203902355](https://s2.loli.net/2025/07/03/IPXJb2NndB9FfWK.png)

点击其他平台，选择 Linux：

![image-20250703203959492](https://s2.loli.net/2025/07/03/QzOT38H4qDhdxae.png)

默认选择第一个，然后下载：

![image-20250703204041798](https://s2.loli.net/2025/07/03/xt5ELIzlpnOT4hm.png)



然后我们现在可以将我们下载的文件放在显眼的位置，我则合理放在了 `"E:\google-chrome-stable_current_amd64.deb"`：

为什么可以随便放呢？

WSL（Windows Subsystem for Linux）默认的共享文件夹并不是固定在某个特定盘符，而是可以访问 Windows 系统中的任何盘符的内容。

在 WSL 中，Windows 的所有盘符会自动挂载到 `/mnt` 目录下，你可以通过 `/mnt/<盘符>` 的路径来访问相应盘符中的文件。例如，要访问 Windows 的 C 盘，可以使用 `/mnt/c` 路径；访问 D 盘，则使用 `/mnt/d` 路径等。同时，Windows 也可以通过 `\\wsl$` 来访问 WSL 中的文件系统。

```bash
pi@localhost:/$ cd /mnt
pi@localhost:/mnt$ ls
c  d  e  wsl  wslg
pi@localhost:/mnt$ 
```

不过，虽然 WSL 可以访问 Windows 任何盘符的文件，但在实际使用中可能会遇到一些权限方面的问题。因为 Windows 盘符挂载到 WSL 中时，默认的权限是 777，这可能不符合一些安全要求或特定的开发场景需求，需要根据实际情况进行权限调整。

我们我直接进入 Chrome 安装包所在的位置，然后安装：

```bash
# 进入 Windows 的 E 盘
cd /mnt/e  
# 使用 dpkg 安装
sudo dpkg -i google-chrome-stable_current_amd64.deb
```

安装完成后打开 Chrome 浏览器：

```bash
google-chrome &
```





### 5.3 安装 Sublime Text 3：

Sublime Text 3 是一个优秀的代码编辑工具。

#### **① 导入 GPG 密钥**：

使用以下命令导入 Sublime Text 3 的 GPG 密钥，以确保软件包的完整性和安全性。

```bash
wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
```



#### **② 添加软件源**：

将 Sublime Text 3 的官方软件源添加到系统的软件源列表中。

```bash
echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
```



#### **③ 更新软件包列表**：

运行以下命令，让系统更新软件包列表，以便获取到 Sublime Text 3 的相关信息。

```bash
sudo apt update
```



#### **④ 安装 Sublime Text 3**：

使用下面的命令进行安装。

```bash
sudo apt install sublime-text
```



#### **⑤ 在终端中启动 Sublime Text**：

```bash
subl
```



### 5.4 安装 Nautilus：

Nautilus 也称为 GNOME 文件，是 GNOME 桌面的文件管理器。 （类似于 Windows 文件资源管理器）。

```bash
sudo apt install nautilus -y
```

若要启动，请输入： `nautilus`


### 5.5 安装 ZSH 终端：
#### ① 安装宿主工具：Windows Terminal (预览版或正式版)：
如果你还在用 Windows 自带的 CMD 黑色窗口，建议立刻替换。
- **安装方法**：打开 Windows 的 **Microsoft Store**，搜索并安装 **"Windows Terminal"**（或者叫“终端”）。
- **优点**：支持多标签页、高度自定义配色、完美支持透明度、支持 Unicode 字符（显示图标）。

#### ② 安装高效 Shell：Zsh + Oh My Zsh：
Ubuntu 默认的 `bash` 虽然稳定，但 Tab 补全不够智能。我们换成 `zsh`。
1. **在 WSL 中安装 Zsh**：
```bash
sudo apt update
sudo apt install zsh -y
```

2. **安装 Oh My Zsh（管理配置的框架）**：
```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
- 安装过程中会问你是否设为默认 Shell，输入 `y`。_

3. 实现“地表最强” Tab 补全和历史建议
这是你最需要的功能，需要安装两个神级插件：
**A. 自动建议 (zsh-autosuggestions)**
它会根据你的历史记录，用灰色文字帮你预测命令，按 **→** 键即可一键补全。
```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

 **B. 语法高亮 (zsh-syntax-highlighting)**
命令输对了是绿色，输错了是红色，一眼就能看出单词拼错没。
```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

 **C. 启用插件**
1. 打开配置文件：`nano ~/.zshrc`

2. 找到 `plugins=(git)` 这一行，修改为：
```bash
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

3. 保存并退出（Ctrl+O, Enter, Ctrl+X），然后刷新配置：
```bash
source ~/.zshrc
```


#### ③ 让 VS Code 终端也变漂亮
1. 在 VS Code 中按下 `Ctrl + Shift + P`。
2. 搜索并选择 `Terminal: Select Default Profile`。
3. 选择 **WSL (Ubuntu)**。
4. 由于 Zsh 使用了一些特殊符号，建议在 Windows 上安装 **Nerd Fonts**（如 _MesloLGS NF_），并在 VS Code 设置中搜索 `Terminal Integrated Font Family` 修改为该字体，否则可能会有乱码小方块。


如果发现搜索`Terminal: Select Default Profile`无结果时：


1. 确认 Zsh 是否安装成功
首先回到你的 WSL 终端（那个黑色的命令行窗口），输入：
```bash
which zsh
```
- **如果返回** `/usr/bin/zsh`，说明安装成功。
- **如果返回空**，请重新执行 `sudo apt install zsh -y`。

2. 在 VS Code 中手动指定 Zsh 路径
如果下拉菜单里还是没有，我们可以直接在 VS Code 的设置里“写死”它：
	- 在 VS Code 中按 `Ctrl + ,` (逗号) 打开设置。
	- 在搜索框输入 `automationShell.linux`。
	- 点击 **“在 settings.json 中编辑”**。
	- 在打开的 JSON 文件中，添加或修改以下配置（确保在大括号 `{}` 内部）：
```json
	"terminal.integrated.defaultProfile.linux": "zsh",
	"terminal.integrated.profiles.linux": {
	"zsh": {
		"path": "/usr/bin/zsh"
	}
}
```
3. **重启 VS Code**（彻底关掉再开）

## 6. Python 以及依赖项安装：

### 6.1 安装 Python 以及创建环境：

####  **① 添加 deadsnakes PPA 源：**

```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
```



####  **② 安装 Python 3.10：**

```bash
sudo apt install python3.10 python3.10-dev python3.10-venv
```



####  **③ 验证安装**：

```bash
python3.10 --version  # 应显示 Python 3.10.x
```



####  **④ 创建 Python 3.10 虚拟环境**：

```bash
# 使用 python3.10-venv 模块创建虚拟环境（推荐方式）
python3.10 -m venv ~/workspace/hailo

# 或使用 virtualenv 指定 Python 3.10
virtualenv --python=$(which python3.10) ~/workspace/hailo
```



####  **⑤ 激活虚拟环境并验证**：

```bash
# 激活虚拟环境
source ~/workspace/hailo/bin/activate
# 显示 Python 3.10.18
python --version                        
```



#### **⑥ 更新 pip**：

```bash
pip install --upgrade pip
```


## 8. 使用 VS Code 开发 WSL：

### 8.1 在 Windows 上安装 VsCode：

- 访问 [VS Code 安装页](https://code.visualstudio.com/download) 并选择 32 或 64 位安装程序。 在 Windows 上安装 Visual Studio Code（不在 WSL 文件系统中）。

- 当系统提示在安装过程中 **选择其他任务** 时，请务必选中 **“添加到 PATH** ”选项，以便使用代码命令轻松在 WSL 中打开文件夹。

- 安装 [远程开发扩展包](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)。 除了远程 - SSH 和开发容器扩展之外，此扩展包还包括 WSL 扩展，使你能够在容器、远程计算机上或 WSL 中打开任何文件夹。

> 注意：若要安装 WSL 扩展，需要 [1.35 年 5 月版本](https://code.visualstudio.com/updates/v1_35) 或更高版本的 VS Code。 不建议在没有 WSL 扩展的情况下在 VS Code 中使用 WSL，因为你将失去对自动完成、调试、linting 等的支持。有趣的事实：此 WSL 扩展安装在 $HOME/.vscode/extensions（在 PowerShell 中输入命令 `ls $ HOME\.vscode\extensions\` ）。

- 安装远程连接的插件：
  ![image-20250705160542386](https://s2.loli.net/2025/07/05/RGPuaoAU8kKNZhm.png)





### 8.2 更新 Linux 分发版：

某些 WSL Linux 分发版缺少 VS Code 服务器启动所需的库。 可以使用其包管理器将其他库添加到 Linux 分发版中。

例如，若要更新 Debian 或 Ubuntu，请使用：

```bash
sudo apt-get update
```

若要添加 wget（要从 Web 服务器检索内容）和 ca-certificates（若要允许基于 SSL 的应用程序检查 SSL 连接的真实性），请输入：

```bash
sudo apt-get install wget ca-certificates
```



### 8.3 在 Vs Code 中打开 WSL 项目：

若要从 WSL 分发版打开项目，请打开分发的命令行并输入： `code .`

![使用 VS Code 远程服务器打开 WSL 项目](https://learn.microsoft.com/zh-cn/windows/wsl/media/wsl-open-vs-code.gif)

等待加载完成，就会自动打开 VSCode，然后输入 **ctrl+`**打开命令行窗口：

![image-20250705161518356](https://s2.loli.net/2025/07/05/1DElvKeYBnru79Q.png)



### 8.4 上传数据集：

#### ① 安装解压工具：

**（1）安装 ZIP/UNZIP（处理 .zip 文件）（可选）：**

ZIP 是 Linux 最常用的压缩格式，使用 `zip` 和 `unzip` 工具处理：

```bash
# Ubuntu/Debian 系统
sudo apt install zip unzip
```

 **常用命令：**

```bash
# 解压 ZIP 文件
unzip file.zip -d /path/to/extract/  # -d 指定解压目录

# 创建 ZIP 文件
zip -r archive.zip folder/  # -r 递归压缩目录
```



 **（2）安装 RAR/UNRAR（处理 .rar 文件）（可选）：**

RAR 是闭源格式，需要安装 `unrar` 工具解压，`rar` 工具创建压缩包：

```bash
# Ubuntu/Debian 系统
sudo apt install unrar
```

 **常用命令：**

```bash
# 解压 RAR 文件
unrar x file.rar -d /path/to/extract/  # -d 指定解压目录

# 创建 RAR 文件（需先安装 rar 工具）
rar a archive.rar folder/  # a 表示添加到压缩包
```



 **（3）其他压缩格式支持：**

 **7-Zip（处理 .7z、.tar.7z 等格式）（可选）：**

```bash
# Ubuntu/Debian 系统
sudo apt install p7zip-full

# Arch Linux 系统
sudo pacman -S p7zip
```

 **常用命令：**

```bash
# 解压 7z 文件
7z x file.7z -o/path/to/extract/  # -o 指定解压目录

# 创建 7z 文件
7z a archive.7z folder/
```



 **tar.gz/tar.bz2 等（Linux 原生压缩格式）（可选）：**

```bash
# 解压 tar.gz
tar -xzvf file.tar.gz -C /path/to/extract/  # -C 指定解压目录

# 解压 tar.bz2
tar -xjvf file.tar.bz2 -C /path/to/extract/

# 创建 tar.gz
tar -czvf archive.tar.gz folder/
```



**通用解压命令（推荐）：**

如果不想记具体格式，可以使用 `dtrx`（智能解压工具）：

```bash
# 安装 dtrx
sudo apt install dtrx  # Ubuntu/Debian

# 解压任意格式
dtrx file.zip  # 或 file.rar、file.7z 等
```



#### ② 解压数据集文件：

