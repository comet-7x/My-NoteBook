>图床，顾名思义，就是存储图片的“床”。它是一种在线服务，允许用户上传、存储和分享图片。通过图床，用户可以将图片上传到云端，然后获得一个链接，可以在任何地方分享这个链接，而不需要担心图片的存储和带宽问题。



## GitHub配置

### 创建 GitHub 仓库

- 登录 GitHub
- 点击右上角的 “+” 号，选择 “New repository”
- 填写仓库名称，仓库描述写不写无所谓，选择仓库可见性（公开或私有）建议公开，然后点击 “Create repository”。


### 生成Token令牌
- 点击右上角的头像或照片
- 从下拉菜单中选择 “Settings”
- 在左侧菜单中，点击 “Developer settings”
- 在 “Developer settings” 页面中，点击 “Personal access tokens”
- 点击 “Generate new token” 按钮
- 在 “Note” 字段中，输入一个描述性名称，以便你记住这个令牌的用途
- 选择令牌的 “Expiration” 日期。你可以选择让令牌永不过期，或者设置一个过期日期
- 选择令牌的 “Scopes” 或权限。根据你利用令牌的目的，选择合适的权限。例如，如果你只需要访问仓库内容，可以选择 “repo” 权限。
- 一旦生成令牌，你将看到令牌的明文。请立即复制并保存这个令牌到一个安全的地方。这是你唯一一次看到这个令牌的机会。之后，你将无法查看这个令牌的明文，只能看到它是否仍然实用。

>注意:
>- 个人访问令牌非常敏感，应像密码一样保护。不要将其泄露给他人，也不要将其硬编码在代码中。
>- 如果你怀疑令牌的安全性受到了威胁，应立即在 GitHub 设置中撤销该令牌。
>- 通过GitHub 令牌能够用于执行与你的 GitHub 账户相关的各种操作，因此请谨慎选择令牌的权限。

---

## PicGo配置

### 下载  PicGo

官网：[https://molunerfinn.com/PicGo/](https://molunerfinn.com/PicGo/)


###  配置图床
- 点击打开图床设置
- 打开Github，点击 "+" 选择新建

![image.png](https://cdn.statically.io/gh/comet-7x/My-NoteBook/main/images/202604191542142.png)

```markdown
图床配置名：任意取名即可（我这里直接命名为Github）
设定仓库名：你的Github用户名/仓库名字
设定分支名：main
设定Token：你刚才复制的 token
设置存储路径：images/
设置自定义域名：https://cdn.statically.io/gh/你的Github用户名/仓库名字/分支
```
可以参考我的设置：
![image.png](https://cdn.statically.io/gh/comet-7x/My-NoteBook/main/images/202604191556888.png)

---

## Typora配置

### 配置图床
- 点击文件 -> 点击偏好设置 -> 点击图像
- 选择上传服务为PicGo（app）
- 选择PicGo路径为本地的安装路径
![image.png](https://cdn.statically.io/gh/comet-7x/My-NoteBook/main/images/202604191602929.png)
