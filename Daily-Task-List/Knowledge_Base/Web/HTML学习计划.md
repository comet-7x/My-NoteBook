## HTML结构速览
- **文档声明** (`<!DOCTYPE html>`)：告诉浏览器这是一个现代的 HTML5 网页。
- `<html>`（树根）
    
    - 📄 `<head>`（网页的幕后大脑）
        
        - 🏷️ `<meta>`：设置编码（如 `charset="UTF-8"`，防止中文乱码）。
            
        - 🏷️ `<title>`：标签页的名字。
            
        - 🔗 `<link>`：把写好的 CSS 样式表像外挂一样引进来。
            
    - 🎨 `<body>`（网页的台前舞台）
        
        - 🏗️ **内容骨架**：`<h1>`~`<h6>`（大标题）、`<p>`（文本段落）。
            
        - 📊 **数据排列**：`<ul>` 嵌套 `<li>`（列表项）。
            
        - 🔗 **多媒体与交互**：`<a>`（负责跳走）、`<img>`（负责看）、`<button>`（负责点）、`<input>`（负责打字）。
            
        - 📦 **现代大管家**：`<div>`。


## HTML 源码 与 DOM 树的区别
我们编写的HTML只是文本内容，它在被浏览器加载时会被**解析成一棵内存里的树形对象结构**，这棵树就叫 **DOM（Document Object Model） 树**。
1. HTML 只是一段纯字符串文本

```html
<html>
  <head>
    <title>页面</title>
  </head>
  <body>
    <div class="box">
      <p>文字</p>
    </div>
  </body>
</html>
```

这段代码本质就是一堆字符，浏览器没法直接读取标签层级、父子关系、属性。

 2. 浏览器解析后生成 DOM 树（树形结构）

把标签转换成**节点对象**，按照嵌套关系分出：根、父、子、兄弟节点，树形分层：

```plaintext
Document（文档根节点，最顶层）
└── html 元素节点
    ├── head 元素节点
    │   └── title 元素节点
    │       └── #text 文本节点 "页面"
    └── body 元素节点
        └── div 元素节点 class="box"
            └── p 元素节点
                └── #text 文本节点 "文字"
```

这就是 DOM 树，**内存里的对象树**，JS 能直接读写每一个节点。


### 四、为什么一定要有 DOM 树？

#### 1. HTML 文本不可操作，DOM 对象可编程：

HTML 只是静态文字；DOM 是**JS 可访问的对象**。

举例：想修改 div 文字

- 直接改 HTML 字符串：要匹配截取、拼接，极其麻烦
- 操作 DOM：`document.querySelector('div').innerText = '新文字'`
    
    DOM 树提供一套标准 API，任意增删改查页面结构。

### 2. 树形结构天然匹配 HTML 嵌套语法

HTML 本身就是层层嵌套，树结构完美对应层级：

- 查找父 / 子 / 兄弟元素逻辑清晰
- 批量遍历页面所有标签（循环子节点）非常方便

### 3. 浏览器渲染的中间桥梁

完整渲染流程：

1. 下载 HTML 字符串
2. 解析生成 **DOM 树**
3. 同时解析 CSS 生成 CSSOM 样式树
4. DOM + CSSOM 合并生成 **渲染树 Render Tree**
5. 布局 Layout → 绘制 Paint → 合成 Composite

没有 DOM 树，浏览器无法计算每个元素的样式、位置、大小。

## 五、举前端开发日常场景，直观感受 DOM 树

### 场景 1：获取链接、批量修改（对应你刚才 Excel 批量链接需求）

页面一堆 `<a href="url">`，想批量读取所有链接地址：

js

运行

```
// 获取DOM树上所有a元素节点
const aList = document.querySelectorAll('a')
aList.forEach(a => {
  console.log(a.href) // 读取DOM节点的href属性
  a.style.color = 'red' // 修改DOM节点样式
})
```

底层就是在遍历 DOM 树的子节点。

### 场景 2：动态新增页面元素

js

运行

```
// 在DOM树上创建新节点
const div = document.createElement('div')
div.textContent = '新增内容'
// 挂载到DOM树body下（添加子节点）
document.body.appendChild(div)
```

`createElement` 是在内存创建 DOM 节点，`appendChild` 是把节点插入 DOM 树，页面才会渲染出来；只创建不挂载树上，页面看不见。

### 场景 3：删除元素

js

运行

```
document.querySelector('.box').remove()
```

本质：从 DOM 树中移除该节点及其所有后代子树。