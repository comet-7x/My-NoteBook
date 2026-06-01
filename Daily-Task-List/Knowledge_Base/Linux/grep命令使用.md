## grep 读代码技巧

### 基本格式

```bash

grep [选项] "要找的内容" 文件或目录

```


### 常用选项

| 选项                 | 作用          |
| ------------------ | ----------- |
| `-r`               | 递归搜索子目录     |
| `-n`               | 显示行号        |
| `-i`               | 忽略大小写       |
| `-l`               | 只显示文件名      |
| `--include="*.py"` | 只搜索特定后缀     |
| `-v`               | 排除匹配行（反向搜索） |



### 常用组合

```bash

# 基本搜索（几乎每次都用这个）

grep -rn "关键词" src/ --include="*.py"

  

# 限制输出行数，避免刷屏

grep -rn "关键词" src/ | head -20

  

# 排除干扰项

grep -rn "prompt" src/ | grep -v "system_prompt"

  

# 精确匹配单词（\b 是单词边界）

grep -rn "\.prompt\b" src/

```

  

### 读代码的固定追踪套路

遇到不认识的东西，按顺序执行：
```bash

# 1. 找定义

grep -rn "class Foo\|def Foo" src/

  

# 2. 找 import 来源

grep -rn "import.*Foo" src/

  

# 3. 找调用方

grep -rn "\.foo\b" src/ --include="*.py"

```

  

### 三个核心动作
**定位 → 追踪 → 验证**

- 定位：从行为差异出发，找到入口（类、函数、参数）

- 追踪：顺着 import 往上找类型来源，顺着调用往下找执行路径

- 验证：用自己的话解释，能讲清楚说明真正理解了