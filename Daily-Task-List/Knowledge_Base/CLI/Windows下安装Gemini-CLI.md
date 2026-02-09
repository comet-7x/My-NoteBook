# 在Windows上通过代理安装 Gemini CLI

本指南介绍了如何在 Windows 的命令提示符（`cmd.exe`）或 PowerShell 环境下，通过设置临时代理来安装 `@google/gemini-cli`。

## 安装步骤

### 1. 设置临时代理
打开一个新的命令提示符窗口，然后执行以下命令来设置 HTTP 和 HTTPS 代理。请将 `7890` 替换为你的代理软件实际监听的端口。

**在 cmd.exe 中:**
```powershell
:: 设置 HTTP 和 HTTPS 代理
set http_proxy=http://127.0.0.1:7890
set https_proxy=http://127.0.0.1:7890
```

**在 PowerShell 中:**
```powershell
# 设置 HTTP 和 HTTPS 代理
$env:http_proxy="http://127.0.0.1:7890"
$env:https_proxy="http://127.0.0.1:7890"
```

### 2. 测试代理连通性 (可选)
你可以使用 `curl` 或其他网络工具来测试代理是否已成功连接到外部网络。

```powershell
# 如果返回了 Google 的服务器信息，则代表代理设置成功
curl -I https://www.google.com
```

### 3. 安装 Gemini CLI
在**同一个窗口**中，使用 `npm` 执行安装命令。由于代理已设置，`npm` 将通过代理下载包。

```powershell
npm install -g @google/gemini-cli
```

### 4. 首次运行与身份验证
安装成功后，你可以运行 `gemini` 命令进行验证，并根据提示完成首次的身份验证流程。

```powershell
# 检查版本号
gemini --version

# 启动并根据引导进行认证
gemini
```

### 5. 清除代理设置 (重要)
完成安装后，为了避免影响其他网络操作，建议关闭当前命令行窗口，或在当前窗口中手动清除代理设置。

**在 cmd.exe 中:**
```powershell
:: 清除代理设置
set http_proxy=
set https_proxy=
```

**在 PowerShell 中:**
```powershell
# 清除代理设置
Remove-Item env:http_proxy
Remove-Item env:https_proxy
```