# Windows端SSH免密登录新服务器（更换服务器后）
## 背景说明
服务器更换后，IP（223.4.6.21）、端口（5040）、登录用户（zhihao）未变，但新服务器的`zhihao`用户为全新创建，未配置SSH免密登录公钥；同时本地Windows保留了旧服务器的SSH指纹，导致首次连接触发指纹冲突报错。

## 核心问题
1. 本地`known_hosts`文件记录的旧服务器指纹与新服务器不匹配，触发SSH安全校验失败；
2. 新服务器`zhihao`用户无`.ssh/authorized_keys`文件，无法免密登录。

## 解决步骤
### 步骤1：清理本地旧服务器SSH指纹（解决指纹冲突）
打开Windows PowerShell，执行以下命令删除旧指纹：
```powershell
ssh-keygen -R "[223.4.6.21]:5040"
```
执行成功会提示：
```
# Host [223.4.6.21]:5040 found: line 9
C:\Users\19334/.ssh/known_hosts updated.
Original contents retained as C:\Users\19334/.ssh/known_hosts.old
```

### 步骤2：密码登录新服务器
执行登录命令，按提示确认新服务器指纹并输入用户密码：
```powershell
ssh zhihao@223.4.6.21 -p 5040
```
- 当出现指纹确认提示时，输入`yes`（必须输全，仅输`y`无效）；
- 输入`zhihao`用户的密码，完成登录。

### 步骤3：在新服务器配置免密登录公钥
登录服务器后，逐行执行以下命令（配置公钥及权限）：
```bash
# 1. 创建.ssh目录（新用户默认无此目录）
mkdir -p ~/.ssh

# 2. 设置.ssh目录权限（SSH要求严格权限，否则免密登录失败）
chmod 700 ~/.ssh

# 3. 编辑authorized_keys文件，写入本地公钥
vim ~/.ssh/authorized_keys
```
vim编辑器操作：
- 按`i`进入插入模式；
- 粘贴本地`C:\Users\19334\.ssh\id_rsa_wenji.pub`文件的全部内容；
- 按`Esc`退出插入模式，输入`:wq`回车保存并退出。

```bash
# 4. 设置authorized_keys文件权限
chmod 600 ~/.ssh/authorized_keys

# 5. （可选）重启SSH服务确保配置生效（Ubuntu系统）
sudo systemctl restart ssh
```

### 步骤4：验证免密登录
退出当前服务器连接（输入`exit`），重新执行登录命令：
```powershell
ssh zhihao@223.4.6.21 -p 5040
```
若无需输入密码直接登录，说明免密登录配置成功。

## 关键注意事项
1. 本地私钥文件`C:\Users\19334\.ssh\id_rsa_wenji`无需删除，是免密登录的核心凭证；
2. SSH配置文件中`IdentityFile`路径需修正多余引号：
   ```
   # 错误写法
   IdentityFile "C:\Users\19334\.ssh\id_rsa_wenji"""
   # 正确写法
   IdentityFile "C:\Users\19334\.ssh\id_rsa_wenji"
   ```
3. `.ssh`目录权限必须为`700`，`authorized_keys`文件权限必须为`600`，权限错误会直接导致免密登录失败；
4. 公钥内容需完整复制，不可遗漏字符或多空格。

## 常见问题排查
若配置后仍需输入密码登录：
1. 检查本地私钥路径是否正确（`IdentityFile`配置）；
2. 检查服务器端`.ssh`目录和`authorized_keys`文件权限；
3. 检查`authorized_keys`文件中的公钥内容是否完整、无格式错误；
4. 重启服务器端SSH服务后再次验证。