这是一份关于 **Windows 免密连接 Linux 远程服务器** 的 Markdown 笔记，涵盖了从秘钥生成到配置自动化连接的完整流程。

---

# Windows 免密连接 Linux 服务器指南

## 1. 原理简介

免密登录基于 **SSH 公钥加密算法**（如 RSA 或 ED25519）。

- **私钥 (Private Key)**：保存在本地 Windows 电脑，绝对不能泄露。
    
- **公钥 (Public Key)**：上传至 Linux 服务器，用于身份验证。
    

## 2. 操作步骤

### 第一步：在 Windows 本地生成密钥对

打开 PowerShell 或 CMD，输入以下命令：

```bash
ssh-keygen -t ed25519 -f "$HOME\.ssh\id_ed25519_remote"
```

- `-t ed25519`：推荐使用 ed25519 算法，安全性高且速度快。
    
- `-f`：指定存储路径和文件名。
    
- **提示设置密码 (Passphrase)**：直接回车（留空）即可实现真正的“免密”。
    

### 第二步：将公钥上传至 Linux 服务器

你需要将本地生成的 `.pub` 文件内容复制到服务器的 `~/.ssh/authorized_keys` 文件中。

#### 方法 A：使用 PowerShell 自动化（推荐）

在 Windows PowerShell 执行：

PowerShell

```
$pubKey = Get-Content "$HOME\.ssh\id_ed25519_remote.pub"
ssh user@remote_host "mkdir -p ~/.ssh && echo '$pubKey' >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

#### 方法 B：手动复制

1. 用记事本打开 `id_ed25519_remote.pub`，复制全部文本。
    
2. 登录 Linux 服务器，执行：
    
    Bash
    
    ```
    nano ~/.ssh/authorized_keys
    ```
    
3. 将公钥粘贴到文件末尾，保存退出。
    

### 第三步：配置本地 SSH Config（实现快捷登录）

为了避免每次输入长 IP 地址和指定私钥路径，修改 Windows 本地的 `C:\Users\用户名\.ssh\config` 文件：

Plaintext

```
Host my_server
    HostName 1.2.3.4          # 服务器 IP
    User root                 # 登录用户名
    IdentityFile ~/.ssh/id_ed25519_remote
```

## 3. 快速连接

完成上述配置后，你只需在终端输入以下命令即可秒连：

Bash

```
ssh my_server
```

---

## 4. 常见问题排查 (Troubleshooting)

- **权限问题**：Linux 上的 `.ssh` 目录权限必须为 `700`，`authorized_keys` 必须为 `600`。
    
- **SSH 服务配置**：确保服务器 `/etc/ssh/sshd_config` 中 `PubkeyAuthentication` 设置为 `yes`。
    
- **VS Code 支持**：如果你使用 VS Code 的 Remote SSH 插件，它会自动读取上述 `config` 文件，点击侧边栏即可连接。
    

---

**需要我为你补充如何在 VS Code 中进一步优化远程开发环境（如插件配置或本地代理设置）吗？**