# JavaScript 零基础入门语法速成

本手册专为有 C/Python 基础或零基础快速上手 JS / Node.js 脚本编写设计。


## 安装Node.js

### 第一步：安装 Node.js

根据操作系统选择最直接的安装方式（均建议安装 **LTS（长期支持）** 版本）：

- **Windows / macOS（安装包方式，最简单）**：
    1. 访问官网 [nodejs.org](https://nodejs.org/)。
    2. 下载标注为 **LTS** 的安装包（`.msi` 或 `.pkg`）。
    3. 双击运行安装程序，一路点击“下一步（Next）”直到完成（默认配置即可，它会自动配置环境变量）。

- **macOS（命令行方式）**：
    ```bash
    brew install node
    ```

- **Linux (Ubuntu / Debian)**：
    ```bash
    sudo apt update
    sudo apt install -y nodejs npm
    ```


### 第二步：验证安装是否成功

打开终端（Windows 使用 `PowerShell` 或 `CMD`，Mac / Linux 使用 `Terminal`），输入以下命令：

```bash
node -v
npm -v
```

若能正确输出版本号（例如 `v20.x.x` 或 `v22.x.x`），说明环境已经就绪。

### 第三步：创建并运行第一个脚本

**1. 创建项目文件夹并进入**

```bash
mkdir js-demo
cd js-demo
```

**2. 创建脚本文件**

新建一个名为 `app.js` 的文件（可以用 VS Code 等编辑器打开，也可以直接在终端创建）：

```bash
# Mac / Linux 可使用 touch，Windows 可使用 ni
touch app.js
```

**3. 在 `app.js` 中写入代码**

打开 `app.js`，填入以下测试代码并保存：

```JavaScript
const message = "你好，Node.js 已经跑起来了！";
const numbers = [1, 2, 3, 4, 5];

console.log(message);
console.log("计算结果:", numbers.map(x => x * 10));
```

**4. 在终端运行脚本**

确保终端所在的路径是包含 `app.js` 的目录，输入：

```bash
node app.js
```

终端将立即输出：

```plaintext
你好，Node.js 已经跑起来了！
计算结果: [ 10, 20, 30, 40, 50 ]
```

### 进阶提示：临时测试的交互式终端（REPL）
如果只想快速验证几行 JS 语法或简单的数组计算，不需要创建文件：
1. 终端直接输入 `node` 回车，即可进入交互模式（类似 Python 的 `python` REPL）。
2. 输入 `const a = 1; a + 2;` 直接回车看结果。
3. 连续按两次 `Ctrl + C` 退出交互模式。

## 语法速查

### 1. 变量：存东西

用 `const` 或 `let` 声明变量，严禁使用旧式的 `var`。

```JavaScript
// const：常量，声明后不可重新赋值（优先默认使用）
const name = "Hello JavaScript";

// let：变量，后续需要修改值时使用
let age = 22;
age = 23; // 正确

console.log(name, age); // 打印到终端，相当于 Python 的 print()
```

> **类比 Python**：`name = "Hello JavaScript"`

### 2. 注释

```JavaScript
// 单行注释（这一行不会执行）

/*
  多行注释
  中间写几行都可以
*/
```

### 3. 数据类型

```JavaScript
const num = 100;         // 数字 Number（整数和小数都是 Number）
const str = "hello";     // 字符串 String，单引号/双引号均可
const flag = true;       // 布尔 Boolean：true / false（首字母全小写）
const emptyVal = null;   // 空值 Null（主动赋值为空）
let notAssigned;         // undefined（已声明但未赋值）

// 数组 Array，等价于 Python 的 list
const arr = [1, 2, 3];
console.log(arr[0]);     // 取第一个元素，输出 1

// 对象 Object，等价于 Python 的 dict
const user = {
  name: "abc",
  age: 18
};
console.log(user.name);  // 取属性写法 1：输出 abc
console.log(user["age"]); // 取属性写法 2（当键名是变量时用这种）
```

### 4. 字符串进阶：模板字符串

用反引号 `` ` `` 包裹，通过 `${}` 嵌入变量或运算。

```JavaScript
const name = "Tom";
const score = 95;

// 推荐写法：模板字符串
console.log(`学生 ${name} 的得分是: ${score}`);

// 传统写法（繁琐易错，不推荐）: "学生 " + name + " 的得分是: " + score
```

> **类比 Python**：`f"学生 {name} 的得分是: {score}"`

### 5. 函数定义（普通函数与箭头函数）

```JavaScript
// 1. 普通函数声明
function sayHello(msg) {
  return `收到消息: ${msg}`;
}

// 2. 现代箭头函数（主流写法，必须掌握）
const add = (a, b) => {
  return a + b;
};

// 极简写法：若只有一行返回值，可省略大括号与 return
const multiply = (a, b) => a * b;

console.log(sayHello("你好"));
console.log(multiply(3, 4)); // 输出 12
```

> **类比 Python**：`multiply = lambda a, b: a * b`


### 6. 条件判断与全等比较

判断必须使用全等号 `===` 和不全等号 `!==`，避免类型隐式转换导致的 Bug。

```JavaScript
const score = 80;

if (score >= 90) {
  console.log("优秀");
} else if (score >= 60) {
  console.log("及格");
} else {
  console.log("不及格");
}

// 严格相等判断
const val = "100";
if (val === 100) {
  console.log("相等"); // 不会执行，因为类型不同（字符串 vs 数字）
}
```

### 7. 数组常用操作

```JavaScript
const list = ["a", "b", "c"];

// 1. push：向末尾添加元素（类似 Python 的 list.append()）
list.push("d"); 

// 2. forEach：单纯遍历
list.forEach((item, index) => {
  console.log(`下标 ${index}: ${item}`);
});

// 3. map：逐项映射生成新数组（类似列表推导式 [x * 2 for x in nums]）
const nums = [1, 2, 3];
const doubled = nums.map(x => x * 2); // [2, 4, 6]

// 4. filter：按条件筛选生成新数组（类似 [x for x in nums if x > 1]）
const filtered = nums.filter(x => x > 1); // [2, 3]

// 5. includes：检查是否存在某元素（类似 Python 的 "a" in list）
console.log(list.includes("a")); // true
```

### 8. 解构赋值与扩展运算符

快速提取对象/数组属性，或展开拷贝。

```JavaScript
// 1. 对象解构
const config = { host: "localhost", port: 8080, debug: true };
const { host, port } = config; 
console.log(host, port); // "localhost" 8080

// 2. 数组解构
const [first, second] = [10, 20];
console.log(first); // 10

// 3. 展开运算符（...）：浅拷贝或合并对象
const updatedConfig = { ...config, port: 9000 }; // 复制 config 并覆盖 port
```

### 9. 异常捕获 try ... catch

```JavaScript
try {
  // 尝试执行可能报错的逻辑
  const result = JSON.parse("invalid json string");
} catch (err) {
  // 捕获异常，err 是错误对象
  console.error("发生错误:", err.message);
} finally {
  // 无论是否报错都会执行（常用于清理资源）
  console.log("执行完毕");
}
```

> **类比 Python**：`try ... except Exception as err: ... finally: ...`
> 
>   

### 10. 异步操作：Promise 与 async / await

Node.js 绝大多数耗时操作（网络请求、非阻塞文件读写）都是异步的。

```JavaScript
// 模拟一个异步耗时任务
const wait = (ms) => new Promise(resolve => setTimeout(resolve, ms));

async function runTask() {
  console.log("开始任务...");
  await wait(1000); // 等待 1 秒
  console.log("1 秒后完成！");
}

runTask();
```

> **类比 Python**：`async def run_task(): await asyncio.sleep(1)`

### 11. JSON 互转（处理文本与数据）

```JavaScript
const data = { name: "Tom", age: 18 };

// 1. 对象 -> JSON 文本（保存/写入文件用）
const jsonStr = JSON.stringify(data, null, 2); // 第 3 个参数 2 代表缩进 2 格格式化

// 2. JSON 文本 -> 对象（读取/解析文件用）
const parsedObj = JSON.parse(jsonStr);
console.log(parsedObj.name); // "Tom"
```

> **类比 Python**：`json.dumps()` 与 `json.loads()`

### 12. Node.js 模块导入与文件读写实战

Node.js 环境提供核心系统模块（`fs` 操作文件，`path` 拼接路径）。

```JavaScript
// 加载内置模块（CommonJS 规范）
const fs = require('fs');
const path = require('path');

// 构造安全的文件跨平台路径
const filePath = path.join(__dirname, 'data.json');

// 1. 同步写入文件（推荐小型自动化脚本使用，直观易懂）
const content = { title: "JS 笔记", done: true };
fs.writeFileSync(filePath, JSON.stringify(content, null, 2), 'utf-8');

// 2. 同步读取文件
if (fs.existsSync(filePath)) {
  const rawText = fs.readFileSync(filePath, 'utf-8');
  const fileData = JSON.parse(rawText);
  console.log("成功读取文件标题:", fileData.title);
}
```

### 核心避坑指南

|**场景**|**错误/不推荐写法**|**正确/规范写法**|**说明**|
|---|---|---|---|
|**等号比较**|`if (a == 1)`|`if (a === 1)`|`==` 会隐式转类型，`'1' == 1` 为 `true`；必须使用 `===`|
|**变量声明**|`var a = 1;`|`const a = 1;` 或 `let a = 1;`|`var` 存在变量提升和函数级作用域污染，现代 JS 一律不用|
|**布尔判断**|`if (str.length > 0)`|`if (str)`|空字符串 `""`、数字 `0`、`null`、`undefined` 在条件判断中自动视为 `false`|
|**路径拼接**|`__dirname + '/data.json'`|`path.join(__dirname, 'data.json')`|手动拼斜杠在 Windows 和 Linux 上极易引发路径分隔符错误|


## 包管理器与项目工程化
Node.js 生态拥有多款包管理器工具。目前社区推荐首选更轻量、高效的 **pnpm**（类似 Python 现代生态中的 **uv**），最基础通用的为内置的 **npm**（类似 **pip**）。

### 包管理器生态与核心概念类比

|**JS 生态（Node）**|**Python 生态**|**简要说明**|
|---|---|---|
|**npm**|`pip`|官方内置，最基础的包管理器（装 Node 自带）。|
|**pnpm**|`uv` / `poetry`|新一代主流，硬链接机制极度节省磁盘空间、安装速度极快。|
|`package.json`|`pyproject.toml` / `requirements.txt`|项目清单文件，声明项目元信息、依赖包版本及脚本命令。|
|`node_modules/`|`.venv/lib/site-packages`|本地缓存与依赖代码的存放目录（**严禁提交至 Git 仓库**）。|
|`pnpm-lock.yaml` / `package-lock.json`|`uv.lock` / `poetry.lock`|依赖锁定文件，记录每个依赖项的具体安装版本，保证团队环境一致。|
|`npx` / `pnpm dlx`|`uvx`|临时拉取远端工具执行，执行完即销毁，无需全局安装。|

### 标准工程上手全流程
**第一步：初始化项目**

```bash
# 方式 A：使用 pnpm
pnpm init

# 方式 B：使用内置 npm（-y 表示全部使用默认配置）
npm init -y
```

执行后会在当前目录生成基础的 `package.json` 清单。

**第二步：安装依赖库**

```bash
# 安装网络请求库 axios
pnpm add axios
# 对应 npm：npm install axios
```

**第三步：配置 `.gitignore`（避坑必备）**

在项目根目录新建 `.gitignore` 文件并写入：

```plaintext
node_modules
```


### 常用命令对照速查

日常开发以操作 `pnpm` 为主；若未安装 pnpm，使用内置 `npm` 亦可无缝替换：

|**动作目标**|**pnpm（现代推荐）**|**npm（内置默认）**|**Python (uv / pip)**|**说明**|
|---|---|---|---|---|
|**初始化项目**|`pnpm init`|`npm init -y`|`uv init`|生成 `package.json` 项目清单|
|**同步全部依赖**|`pnpm install`|`npm install`|`uv sync`|克隆他人项目后一键安装 `node_modules`|
|**新增运行依赖**|`pnpm add <包名>`|`npm i <包名>`|`uv add <包名>`|记录在 `dependencies` 中（生产运行必需）|
|**新增开发依赖**|`pnpm add -D <包名>`|`npm i -D <包名>`|`uv add --dev <包名>`|记录在 `devDependencies`（仅用于打包、测试等开发环节）|
|**删除依赖**|`pnpm remove <包名>`|`npm uninstall <包名>`|`uv remove <包名>`|移除本地包并自动从清单中删除|
|**运行项目内命令**|`pnpm exec <命令>`|`npx <命令>`|`uv run <命令>`|调用当前项目中已安装的本地 CLI 工具|
|**临时免安装执行**|`pnpm dlx <包名>`|`npx <包名>`|`uvx <包名>`|临时下载、执行一次后自动销毁|
