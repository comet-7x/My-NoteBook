# VsCode 格式化配置完整笔记

---
## 一、HTML / CSS / JavaScript / Markdown 格式化（Prettier）

> Prettier：**纯格式化工具**，统一缩进、引号、换行；不做代码质量检查。

### 1. 安装插件

扩展商店搜索 **Prettier - Code formatter**（作者 esbenp），点 Install

> 插件 ID：`esbenp.prettier-vscode` 支持：`js/ts/html/css/scss/json/markdown`

### 2. 核心配置

打开设置 JSON：Windows `Ctrl + ,`，Mac `Cmd + ,` → 搜索 `Open Settings (JSON)`

```json
{
  // 保存文件自动格式化
  "editor.formatOnSave": true,
  // 为各文件类型单独指定默认格式化器，避免与其他插件冲突
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

> ⚠️ 注意：这里没有设置全局 `editor.defaultFormatter`，只为明确列出的语言指定了 Prettier，避免影响 Python 等其他语言。

### 3. 项目自定义规则（团队协作推荐）

项目根目录新建 `.prettierrc`（JSON），提交进 Git 统一团队规则

```json
{
  "printWidth": 100,
  "tabWidth": 2,
  "singleQuote": true,
  "trailingComma": "es5"
}
```

### 4. 手动格式化
- 快捷键：Mac `Shift + Option + F`；Windows `Shift + Alt + F`
- 命令面板 `Cmd/Ctrl + Shift + P` → `Format Document`（整文件）/ `Format Selection`（选中代码）

---

## 二、Prettier + ESLint 配套配置（JS / TS 项目）

> **职责分工（核心原则）**：格式版式交给 Prettier，代码质量交给 ESLint，二者各管各的，用 `eslint-config-prettier` 消除重叠。

|工具|管什么|
|---|---|
|Prettier|版式：缩进、引号、换行、尾逗号|
|ESLint|质量：未使用变量、`==`、`any`、未定义变量|
|eslint-config-prettier|关闭 ESLint 中与 Prettier 冲突的格式规则|

### 1. 安装插件

扩展商店安装 **ESLint**（作者 Microsoft）

> 插件 ID：`dbaeumer.vscode-eslint`

### 2. 项目根目录安装 npm 依赖

```bash
npm install -D eslint prettier eslint-config-prettier typescript-eslint @typescript-eslint/parser
```

|依赖|作用|
|---|---|
|`eslint`|代码检查核心|
|`prettier`|格式化|
|`eslint-config-prettier`|**关键**：关掉 ESLint 里和 Prettier 冲突的格式规则|
|`typescript-eslint` / `@typescript-eslint/parser`|TS 语法解析与规则（纯 JS 项目可去掉）|

### 3. 配置文件 `eslint.config.js`（新版 flat config）

```js
import js from '@eslint/js'
import tseslint from 'typescript-eslint'
import prettier from 'eslint-config-prettier'

export default tseslint.config(
  { ignores: ['dist', 'node_modules'] },
  js.configs.recommended,
  ...tseslint.configs.recommended,
  prettier, // ⚠️ 必须放最后：覆盖关闭格式类冲突规则
)
```

### 4. VSCode settings.json 追加

```json
{
  "eslint.validate": ["javascript", "typescript", "html"],
  // 保存时自动修复 ESLint 可自动修正的问题
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  }
}
```

### 5. 终端手动命令

```json
npx eslint .            # lint 检查
npx eslint . --fix      # 自动修复可修复问题
npx prettier --write .  # 批量格式化
```

### 6. 配合要点

1. `eslint-config-prettier` 必须放在 config **最后**，才能覆盖前面格式规则
2. ESLint 做质量检查，Prettier 做排版，**不要用 ESLint 规则去格式化**
3. TS 项目：`no-explicit-any` 等质量规则在 ESLint；缩进、引号在 Prettier

---

## 三、Python 格式化 + Lint 检查（Ruff）

> Ruff：**Lint 检查器 + 格式化器**，替代 Black + isort；不只调格式，还能自动修复代码问题、排序 import。

### 1. 前置准备

先装官方 **Python** 插件（Microsoft Python），再装扩展商店的 **Ruff**（作者 Astral）

> 插件 ID：`charliermarsh.ruff`

### 2. 项目本地安装 Ruff（推荐，统一版本）

```bash
# uv（优先）
uv add ruff --dev
# pip 备选
pip install ruff --dev
```

### 3. VSCode settings.json 追加

```json
{
  "[python]": {
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "charliermarsh.ruff",
    // 保存自动：排序 import + 修复可自动修正的 lint 问题
    "editor.codeActionsOnSave": {
      "source.organizeImports.ruff": true,
      "source.fixAll.ruff": true
    }
  }
}
```

### 4. 项目自定义 Ruff 规则（项目根目录）

新建 `ruff.toml`（也可写进 `pyproject.toml`），提交进 Git

```json
line-length = 88          # 单行最大字符长度
indent-width = 4          # Python 缩进 4 空格

[lint]
select = ["E", "W", "F", "I"] # 规则：错误、警告、语法检查、import 排序
```

### 5. 终端手动命令

```bash
ruff format .       # 格式化全部 py 文件
ruff check . --fix  # 检查并自动修复问题
```

### 6. ⚠️ 冲突提醒

旧项目若装了 Black / isort，建议禁用/卸载。Ruff 内置实现了两者功能，一起使用会冲突。
