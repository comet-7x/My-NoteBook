>定位：HTML 只负责**结构 / 内容**（有哪些东西），不负责长什么样（CSS）和点了会怎样（JS）。 这三者分工是整个前端的地基，先记牢。
>- HTML：内容
> - CSS：样式
>  - JS：交互

## 一、文档骨架（房子的地基）
```html
<!DOCTYPE html>           <!-- 声明这是 HTML5 现代网页 -->
<html lang="zh-CN">       <!-- 根标签；lang 告诉浏览器/翻译/读屏软件这是中文，如果网页主要针对国内市场用 `zh-CN`。如果是跨境电商或国际化项目，需要改为 `en`。 -->
  <head>                  <!-- 幕后大脑：给浏览器看的元信息，不直接显示 -->
    <meta charset="UTF-8">                                    <!-- 字符编码，防中文乱码 -->
    <!-- `width=device-width`（宽度等于设备宽）、`initial-scale=1.0`（初始缩放比例 1:1，禁止缩放） -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0"> <!-- 移动端适配开关 -->
    <title>页面标题</title>                                    <!-- 标签页上的名字 -->
    <link rel="stylesheet" href="css/style.css">             <!-- 外挂 CSS 样式表 -->
  </head>
  <body>                  <!-- 台前舞台：所有用户能看到的内容 -->
    ...
  </body>
</html>
```

**`<head>` 里几乎每页都该有的四件套**）：

|标签|作用|漏了会怎样|
|---|---|---|
|`<meta charset="UTF-8">`|字符编码|中文乱码|
|`<meta name="viewport">`|按设备宽度渲染|手机上页面缩成一团、响应式失效|
|`<title>`|标签页标题|标签页显示一串路径，对 SEO 不友好|
|`<link>`|引入 CSS / 图标|页面没样式|

## 二、HTML 常用标签速查表
### 1.文档基础结构标签

|标签|英文全称 / 释义|单 / 双标签|作用说明|
|---|---|---|---|
|`<!DOCTYPE html>`|Document Type|声明，非标签|告知浏览器当前页面使用 HTML5 标准|
|`<html>`|HyperText Markup Language|双标签|页面根节点，所有内容包裹在内|
|`<head>`|Header 头部|双标签|存放页面元数据，不展示到页面|
|`<body>`|Body 主体|双标签|页面所有可见内容存放区域|

### 2.头部元信息标签（head 内）

|标签|英文全称 / 释义|单 / 双标签|作用说明|
|---|---|---|---|
|`<meta>`|Metadata 元数据|单标签|设置字符编码、视口、页面描述等|
|`<title>`|Title 标题|双标签|浏览器标签页标题|
|`<link>`|Link 链接|单标签|引入外部 CSS、图标等资源|
|`<style>`|Style 样式|双标签|书写内部 CSS 样式表|
|`<script>`|Script 脚本|双标签|引入 / 书写 JS 脚本|
|`<noscript>`|No Script|双标签|JS 禁用时展示替代内容|
|`<base>`|Base 基准地址|单标签|设置页面所有链接默认跳转地址|
**避坑指南**：如果 `<script>` 写在 `<head>` 中，现代开发必须加上 `defer` 或 `async` 属性（推荐 `defer`，表示延迟到 DOM 解析完再执行）；或者老规矩，把 `<script>` 标签放在 `</body>` 的正上方。

### 3.文本排版标签
|标签|英文全称 / 释义|单 / 双标签|作用说明|
|---|---|---|---|
|`<h1>~<h6>`|Heading 标题 1-6 级|双标签|页面层级标题，h1 权重最高|
|`<p>`|Paragraph 段落|双标签|正文段落，自带上下间距|
|`<div>`|Division 通用区块|双标签|无语义通用容器，布局专用|
|`<span>`|Span 行内容器|双标签|行内小块文字，局部样式修改|
|`<br>`|Break 换行|单标签|强制文字换行|
|`<hr>`|Horizontal Rule 分割线|单标签|水平分割线，分隔内容区块|
|`<pre>`|Preformatted 预格式化文本|双标签|保留空格、换行、原始排版|
|`<blockquote>`|Block Quote 块级引用|双标签|大段引用文本，缩进展示|
|`<q>`|Quote 行内引用|双标签|小段行内引用，自动加引号|
|`<wbr>`|Word Break 软换行|单标签|长单词允许自动换行断点|


### 4.文字语义格式化标签
|标签|英文全称 / 释义|单 / 双标签|作用说明|
|---|---|---|---|
|`<strong>`|Strong 强调加粗|双标签|语义重要文本，默认粗体|
|`<b>`|Bold 粗体|双标签|仅视觉加粗，无语义|
|`<em>`|Emphasize 强调倾斜|双标签|语义侧重，默认斜体|
|`<i>`|Italic 斜体|双标签|仅视觉倾斜，无语义|
|`<u>`|Underline 下划线|双标签|文字下划线|
|`<s>`|Strike 删除线|双标签|文字中间划删除线|
|`<mark>`|Mark 标记高亮|双标签|文字底色高亮标记|
|`<small>`|Small 小号文字|双标签|缩小字号，备注小字|
|`<sup>`|Superscript 上标|双标签|文字上标（平方、次方）|
|`<sub>`|Subscript 下标|双标签|文字下标（化学式）|
|`<code>`|Code 代码行内文本|双标签|单行程序代码|
|`<kbd>`|Keyboard 键盘输入|双标签|表示键盘按键文字|
|`<var>`|Variable 变量|双标签|标记数学 / 代码变量|
|`<samp>`|Sample 示例输出|双标签|程序运行样例文本|
|`<time>`|Time 时间日期|双标签|语义化标记时间、日期|
|`<abbr>`|Abbreviation 缩写|双标签|缩写词，搭配 title 显示全称|
|`<address>`|Address 联系地址|双标签|标记作者 / 机构联系信息|
|`<cite>`|Citation 引用出处|双标签|标记书籍、文章、作品名称|
|`<dfn>`|Definition 术语定义|双标签|专业名词首次定义标注|
**纠偏 `<u>` 和 `<s>`**：现代网页开发中，下划线和删除线基本都通过 CSS（`text-decoration`）实现。`<u>` 和 `<s>` 极少在开发中使用，了解即可。


### 5.列表标签
|标签|英文全称 / 释义|单 / 双标签|作用说明|
|---|---|---|---|
|`<ul>`|Unordered List 无序列表|双标签|圆点无序号列表|
|`<ol>`|Ordered List 有序列表|双标签|数字有序编号列表|
|`<li>`|List Item 列表项|双标签|ul/ol 内部每一条列表内容|
|`<dl>`|Definition List 定义列表|双标签|术语 + 解释成对列表|
|`<dt>`|Definition Term 术语|双标签|定义列表里的名词|
|`<dd>`|Definition Description 释义|双标签|名词对应的解释内容|

### 6.多媒体与链接标签
|标签|英文全称 / 释义|单 / 双标签|作用说明|
|---|---|---|---|
|`<a>`|Anchor 锚点 / 超链接|双标签|页面跳转、锚点定位|
|`<img>`|Image 图片|单标签|展示图片资源|
|`audio`|Audio 音频|双标签|嵌入音频播放器|
|`<video>`|Video 视频|双标签|嵌入视频播放器|
|`<source>`|Source 资源源文件|单标签|audio/video 内适配多格式媒体|
|`<track>`|Track 字幕轨道|单标签|给视频添加字幕、旁白|
|`<iframe>`|Inline Frame 内联框架|双标签|嵌入另一个网页到当前页面|
|`<picture>`|Picture 响应式图片容器|双标签|多尺寸图片自适应加载容器|
- **超链接 `<a>` 补充**：
    - 如果要实现“在新标签页打开”，必须加 `target="_blank"`。
    - **安全防线**：在使用 `target="_blank"` 时，现代最佳实践是加上 `rel="noopener noreferrer"`，防止新页面通过 `window.opener` 恶意篡改你的原网页。
- **图片 `<img>` 补充**：
    - `alt` 属性非常关键，除了图片加载失败时显示文字，更是 **SEO（让谷歌百度知道图片内容）和盲人读屏软件** 的核心依据。

### 7.语义化布局标签（HTML5 新增）
|标签|英文全称 / 释义|单 / 双标签|作用说明|
|---|---|---|---|
|`<main>`|Main 页面主体核心内容|双标签|页面唯一主要内容区域，一个页面仅用一次，排除导航、侧边栏、页脚|
|`<nav>`|Navigation 导航栏|双标签|存放页面导航链接区域|
|`<header>`|Header 头部区域|双标签|区块 / 页面头部（标题、logo）|
|`<footer>`|Footer 底部区域|双标签|页面 / 区块页脚，版权信息|
|`<section>`|Section 内容区块|双标签|独立主题内容分块|
|`<article>`|Article 独立文章|双标签|完整独立内容（帖子、新闻）|
|`<aside>`|Aside 侧边栏|双标签|侧边辅助内容、侧边栏|
|`<figure>`|Figure 图文组合|双标签|包裹图片 + 图注整体|
|`<figcaption>`|Figure Caption 图注|双标签|figure 内图片说明文字|
**提示**：`<section>` 和 `<article>` 的区别：`<article>` 的内容必须是**完全独立且完整的**（例如一篇博客、一则新闻、一条评论），拿出来放在任何地方都能读懂；而 `<section>` 仅仅是页面的一个**章节或版块**（如“产品特点”、“关于我们”）。


### 8.表格标签
|标签|英文全称 / 释义|单 / 双标签|作用说明|
|---|---|---|---|
|`<table>`|Table 表格|双标签|表格最外层容器|
|`<thead>`|Table Head 表头|双标签|表格头部分组|
|`<tbody>`|Table Body 表主体|双标签|表格数据行分组|
|`<tfoot>`|Table Foot 表尾|双标签|表格底部汇总行|
|`<tr>`|Table Row 表格行|双标签|表格一行单元格|
|`<th>`|Table Header 表头单元格|双标签|表头，默认加粗居中|
|`<td>`|Table Data 数据单元格|双标签|普通表格内容格子|
|`<caption>`|Caption 表格标题|双标签|表格顶部总标题|
|`<col>`|Column 列定义|单标签|统一设置一列样式宽度|
|`<colgroup>`|Column Group 列分组|双标签|包裹多个 col，批量管理列|

### 9.表单输入标签
|标签|英文全称 / 释义|单 / 双标签|作用说明|
|---|---|---|---|
|`<form>`|Form 表单容器|双标签|包裹所有输入控件，用于提交数据|
|`<input>`|Input 输入框|单标签|万能输入控件（文本 / 密码 / 按钮等）|
|`<label>`|Label 输入标签|双标签|绑定输入框，点击文字聚焦输入|
|`<textarea>`|Text Area 多行文本域|双标签|大段多行文字输入框|
|`<select>`|Select 下拉选择框|双标签|下拉菜单外层容器|
|`<option>`|Option 下拉选项|双标签|select 内部每一个下拉选项|
|`<optgroup>`|Option Group 选项分组|双标签|下拉菜单给选项分组归类|
|`<button>`|Button 按钮|双标签|可内嵌图文的功能按钮|
|`<fieldset>`|Field Set 表单分组框|双标签|给表单控件加边框分组|
|`<legend>`|Legend 分组标题|双标签|fieldset 内分组标题文字|
|`<datalist>`|Data List 输入联想列表|双标签|input 输入时下拉联想候选值|
|`<output>`|Output 计算输出|双标签|展示表单计算结果|
- **表单 `<form>` 与 `<input>` 的配合（黄金搭档）**：
    - `<input>` 必须要写 `name` 属性！因为**没有 `name` 属性的输入框，它的数据是无法被 `form` 提交给后端的**。
    - `<label>` 标签的 `for` 属性一定要和 `<input>` 的 `id` 属性一致，这样点击文字时才能激活输入框。
- **`<button>` 的默认行为坑**：
    > **🔥 惊天大坑**：在 `<form>` 表单内的 `<button>`，如果没有指定 `type`，它默认是 `type="submit"`（提交按钮）！点击它会直接导致页面刷新。如果不希望它刷新页面，记得显式声明 `type="button"`。

### 10.其他特殊标签
|标签|英文全称 / 释义|单 / 双标签|作用说明|
|---|---|---|---|
|`<canvas>`|Canvas 画布|双标签|JS 绘制图形、动画、游戏画布|
|`<svg>`|Scalable Vector Graphics|双标签|矢量图形，内嵌图标图形|
|`<details>`|Details 折叠面板|双标签|可展开 / 收起折叠内容|
|`<summary>`|Summary 折叠标题|双标签|details 面板的显示标题|
|`<dialog>`|Dialog 弹窗|双标签|原生模态弹窗对话框|
|`<template>`|Template 模板|双标签|存放模板 DOM，页面不渲染，JS 克隆使用|
|`<portal>`|Portal 跨页面嵌入|双标签|新页面内联预加载（兼容性有限）|

补充区分小总结：
1. **单标签（空元素，无闭合、不能包裹内容）**：`meta` `link` `img` `input` `br` `hr` `source` `wbr`
2. **双标签**：其余全部标签，均可包裹文字、子标签；
3. **语义化标签**：nav/article/section/header/footer/aside 替代纯 div，利于 SEO （Search Engine Optimization，搜索引擎优化）与可读性。
4. `<main>` 是 HTML5 核心语义标签，**一个页面只能出现一次**，代表页面核心正文，区分 header/aside/footer 辅助区域；

## 三、块级元素与行内元素
每个元素默认要么「块级」要么「行内」，决定了它怎么占位：
||块级元素 (block)|行内元素 (inline)|
|---|---|---|
|占位|**独占一行**，从上往下排|不换行，**并排**排列|
|宽高|可设 `width`/`height`|设宽高**通常无效**，由内容撑开|
|例子|`div` `p` `h1` `ul` `li` `header` `section`|`span` `a` `strong` `em`|

> 特例：`<img>`、`<input>` 是「可替换元素」，虽是行内但**可以设宽高**。 进阶：这套默认行为以后能用 CSS 的 `display`（`block`/`inline`/`inline-block`/`flex`/`grid`）随意改——这是第 3 阶段布局的核心开关。

**嵌套红线**：块级元素可以包裹行内元素，但**行内元素绝对不能包裹块级元素**（`<a>` 标签除外，`<a>` 是个特例，它在 HTML5 中可以包裹 `<div>` 等块级盒子）。 另外，**`<p>` 标签内部绝对不能嵌套 `<div>` 或其他的 `<p>`**，否则浏览器会自动强制切断段落，导致 DOM 树解析错乱！


## 四、属性系统
通用写法：`<标签 属性名="值" 属性名="值">`。属性是给标签附加的「配置」。

**全局属性（几乎所有标签都能用）：**
- `class="..."`：**可重复**，给「一类」元素贴标签，供 CSS/JS 批量选中。
- `id="..."`：**页面内唯一**，像主键，用来精确定位单个元素。
- `style="..."`：行内样式，**能不用就不用**（样式应交给 CSS 文件）。
**`id` vs `class`（关键区别）：**

- `id` 唯一 → 一个页面里 `id="lightbox"` 只能有一个 → 对应 JS 的 `getElementById`。
- `class` 可重复 → 一堆卡片都能 `class="cat-card"` → 对应 `querySelectorAll('.cat-card')`。

**常见专用属性：** `href`(a) · `src`/`alt`(img) · `type`(input) · `lang`(html)。
**相对路径（写 `href`/`src` 必懂）：**
- `images/cat.svg`：相对**当前 HTML 文件**所在目录找。
- `./` 当前目录 · `../` 上一级目录 · `/` 网站根目录。
- `https://...`：绝对路径（外部资源）。
- 你项目里 `src="images/logo.svg"` 就是相对路径，所以图片文件夹必须跟 HTML 在对的相对位置。

## 五、书写规范与常见坑

- **标签要闭合**：`<p>` 必须有 `</p>`（单标签除外），漏了会布局错乱。
- **正确嵌套、不能交叉**：
	- `<p><strong>对</strong></p>` ✅；
	- `<p><strong>错</p></strong>` ❌。
- **属性值用引号**：`class="box"`。
- **标签用小写**，养成习惯。
- **注释**：`<!-- 这是注释 -->`，浏览器忽略，只给人看。
- **一个 `<h1>`、一个 `<main>`** 是推荐的最佳实践。


## 六、HTML 源码 与 DOM 树的区别
我们写的 HTML 只是**纯文本字符串**，浏览器加载时会把它**解析成一棵内存里的树形对象结构**，这就是 **DOM（Document Object Model）树**。
源码：
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

解析后的 DOM 树：
```plaintext
Document（文档根节点）
└── html
    ├── head
    │   └── title → #text "页面"
    └── body
        └── div.box
            └── p
                └── #text "文字"
```

**为什么一定要有 DOM 树：**
1. **文本不可操作，对象可编程**：改 HTML 字符串要做截取拼接，极麻烦；操作 DOM 有一套标准 API，`document.querySelector('div').innerText = '新文字'` 一行搞定。
2. **树形天然匹配 HTML 嵌套**：查父/子/兄弟节点、批量遍历都清晰。
3. **它是渲染的中间桥梁**：没有 DOM，浏览器无法计算每个元素的样式和位置。

## 七、浏览器渲染流程
1. 下载 HTML 字符串
2. 解析生成 **DOM 树**
3. 解析 CSS 生成 **CSSOM 样式树**
4. DOM + CSSOM 合并成 **渲染树 (Render Tree)**
5. **布局 Layout（算位置大小）→ 绘制 Paint（上色）→ 合成 Composite（显示）**
> 记住这条流水线：第 4 阶段讲「为什么动画要用 transform」时会回头用到——transform/opacity 只触发合成，不触发布局，所以流畅。


## 八、DOM 日常操作场景
```js
// 1. 批量读取/修改：遍历 DOM 树上所有 a 节点
document.querySelectorAll('a').forEach(a => {
  console.log(a.href);     // 读属性
  a.style.color = 'red';   // 改样式
});

// 2. 动态新增：createElement 在内存建节点，appendChild 才挂上树（不挂载页面看不见）
const div = document.createElement('div');
div.textContent = '新增内容';
document.body.appendChild(div);

// 3. 删除：从 DOM 树移除该节点及其整棵子树
document.querySelector('.box').remove();
```


## 九、关于 DOM 的误区
- **VS Code 只管写代码**，没有浏览器内核，不存在 DOM 树，无法实际操作 DOM。
- **浏览器打开页面后才生成 DOM**；你写的 DOM 操作 JS，到浏览器里才生效。
- **Console** 是浏览器的调试控制台，用来临时手动操作 DOM，和业务代码分离。

## 十、一页速记 / 自测清单
学完 HTML，你应该能不看资料回答：
- [ ] HTML / CSS / JS 在网页开发中分别扮演什么角色？
- [ ] `<head>` 里的“四件套”是什么？如果漏掉 `viewport`，在手机上访问会发生什么？
- [ ] 块级元素、行内元素、行内块元素（可替换元素）在布局和宽高上有何区别？
- [ ] 为什么 `<p>` 标签里不能嵌套 `<div>`？如果嵌套了会怎么样？
- [ ] 在 `<form>` 表单里写一个点击触发 JS 函数的普通按钮，需要注意什么属性？（提示：`type="button"`）
- [ ] 超链接 `<a>` 在新窗口打开时，如何防止安全漏洞？（提示：`rel="noopener"`）
- [ ] HTML 源码文本和浏览器内存中的 DOM 树有什么区别？为什么 JS 必须操作 DOM 树而不是源码？
- [ ] 为什么建议把 `<script>` 标签写在 `</body>` 前面，或者加上 `defer` 属性？




