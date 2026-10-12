# VSCode Prettier 完整使用步骤

Prettier 是**代码格式化工具**，统一 JS/TS/HTML/CSS/Markdown 代码风格，自动调整缩进、引号、换行。

## 1. 安装插件

1. 左侧扩展图标（方块），搜索：**Prettier - Code formatter**（作者：esbenp），点 Install 安装
> 插件 ID：`esbenp.prettier-vscode`

## 2. 核心配置（最重要，不配置不会自动生效）

打开设置 JSON：
- Windows：`Ctrl + ,`
- Mac：`Cmd + ,` 搜索框输入 `Open Settings (JSON)`，打开 `settings.json`

新增粘贴下面配置：

```json
{
  // 保存时自动格式化
  "editor.formatOnSave": true,

  // 单独指定JS、HTML等文件使用prettier（防止冲突）
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[html]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[css]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[markdown]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

保存这个 settings.json