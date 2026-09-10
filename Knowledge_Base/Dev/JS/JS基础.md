## 1. 变量：存东西
用 `const` / `let` 声明变量

```js
// const：常量，一旦赋值不能改（优先用）
const name = "Hello JavaScript";
// let：变量，后面可以重新赋值
let age = 22;

console.log(name); // 打印到终端，相当于python print()
```

> 类比 Python：`name = "JavaScript"`

## 2. 注释
```js
// 单行注释（这一行不会执行）
/*
多行注释
*/
```


## 3. 数据类型

```js
const num = 100;        // 数字 number
const str = "hello";    // 字符串，双引号/单引号都行
const flag = true;      // 布尔：true / false
const arr = [1,2,3];    // 数组，和python list类似
// 对象 Object，类似python dict
const user = {
  name: "abc",
  age: 18
};
console.log(user.name); // 取对象属性，输出 abc
```

## 4. 函数：封装一段可执行代码

两种写法，先记住第一种

```js
// 定义函数
function sayHello(msg) {
  console.log(msg);
}
// 调用函数
sayHello("你好");
```

> 类比 Python
> 
> ```python
> def sayHello(msg):
>     print(msg)
> sayHello("你好")
> ```

## 5. 数组遍历 forEach

数组.forEach (每个元素 => {执行逻辑})

```js
const list = ["a","b","c"];

list.forEach((item, index) => {
  console.log(index, item);
})
```

- item：数组里当前的元素
- index：当前下标，从 0 开始

> 等价 Python：
> `for index, item in enumerate(["a","b","c"]): print(index, item)`

## 6. 条件判断 if

```js
const score = 80;
if(score >= 60){
  console.log("及格");
}else{
  console.log("不及格");
}
```

## 7. try ... catch 捕获错误

```js
try {
  // 尝试执行这里的代码，可能报错
  const a = undefinedVariable;
} catch(err) {
  // 如果上面报错，进入catch，err是错误信息
  console.error("出错了", err.message);
}
```

> Python 等价 `try ... except`

## 8. 模块导入 `require()`（Node 独有，重点！）

Node 里，想要用内置工具（读文件、路径），需要`require`加载模块

```js
// 加载node内置文件模块fs
const fs = require('fs');
// 加载路径处理模块path
const path = require('path');
```

> 类比 Python `import fs`

## 9. 方法调用

```js
// 对象.方法()
const text = 'abc';
text.toUpperCase() // 转大写

fs.readFileSync(文件路径, 'utf-8')
// fs是模块对象，readFileSync是它里面的函数
```

## 10. JSON.parse()

JSON 字符串转 JS 对象。 ipynb 文件存的就是一大段 JSON 文本。

```js
const jsonStr = '{"name":"test"}';
const obj = JSON.parse(jsonStr);
console.log(obj.name) // test
```

> Python：`json.loads()`