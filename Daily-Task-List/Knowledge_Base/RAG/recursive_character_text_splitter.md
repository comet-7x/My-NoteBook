# RecursiveCharacterTextSplitter 深度解析

> 源码位置：`comet_rag/engines/chunkers/base_chunker.py`

---

## 一、为什么需要这个类？

RAG 系统需要把长文档切成小块送给向量模型。最简单的方法是按固定长度切：

```plaintext
文本: "The cat sat on the mat. The dog..."
每 1000 字符切一刀 → chunk1, chunk2, chunk3...
```

问题很明显：**可能在句子中间切断**，语义不完整，向量表征质量下降。

`RecursiveCharacterTextSplitter` 的核心思路是：**优先在自然语义边界切割**。段落 > 句子 > 单词 > 字符，只有在不得不切的时候才降级到更细的粒度。

---

## 二、三个关键参数

```python
RecursiveCharacterTextSplitter(
    chunk_size    = 1000,                          # 每个 chunk 的最大字符数
    chunk_overlap = 200,                           # 相邻 chunk 之间的重叠字符数
    separators    = ["\n\n", "\n", ". ", " ", ""], # 分隔符优先级列表（粗 → 细）
    keep_separator = True                          # 分隔符是否保留在输出中
)
```

### chunk_overlap 的作用

一句话可能恰好落在两个 chunk 的边界上。overlap 让前后 chunk 各拥有这句话，保证检索时不会完全丢失跨边界的上下文：

```plaintext
chunk1: [..正文内容...OVERLAP区域]
chunk2:             [OVERLAP区域...正文内容..]
```

### separators 的优先级逻辑

列表从左到右代表**粒度从粗到细**：

| 分隔符 | 语义单位 |
|--------|----------|
| `"\n\n"` | 段落 |
| `"\n"` | 行 |
| `". "` | 句子 |
| `" "` | 单词 |
| `""` | 字符（终极兜底） |

`""` 必须放在末尾，它保证任何文本最终都能被切到 `chunk_size` 以内。

### 参数约束

```python
chunk_size > 0
chunk_overlap >= 0
chunk_overlap < chunk_size   # 否则 overlap 窗口比 chunk 还大，无意义
```

---

## 三、四个方法逐一解析

### 3.1 `_split_text_with_separator` — 原子切割

这是最底层的操作，只做一件事：**用指定分隔符把文本切成原始片段**。

```python
def _split_text_with_separator(self, text: str, separator: str) -> list[str]:
    if separator == "":
        return list(text)          # "abc" → ["a", "b", "c"]

    if self._keep_separator and separator in text:
        parts = text.split(separator)
        splits = []
        for i, part in enumerate(parts):
            if i == 0:
                splits.append(part)
            else:
                if part:
                    splits.append(separator + part)  # 分隔符前置到后续片段
                elif splits:
                    splits[-1] += separator          # 空片段：把分隔符并入前一段
        return [s for s in splits if s]

    return text.split(separator)   # keep_separator=False：直接丢弃分隔符
```

**三种情况对比：**

```
text = "a\n\nb\n\nc"，separator = "\n\n"

keep_separator=False:  ["a", "b", "c"]          ← 分隔符丢失
keep_separator=True:   ["a", "\n\nb", "\n\nc"]  ← 分隔符前置保留
separator == "":       ["a", "\n", "\n", "b", "\n", "\n", "c"]  ← 逐字符
```

**分隔符前置而非后置的原因**：每个 chunk 输出后，首字符就是它在原文中所在段落/句子的自然开头（含分隔符）。多个 chunk 拼接回来等于原文，信息无损。

**边界情况：连续分隔符 / 末尾分隔符**

```
text = "a\n\n\n\nb"   (两个连续 \n\n)
parts = ["a", "", "b"]

i=0: splits = ["a"]
i=1: part="" → splits[-1] += "\n\n" → ["a\n\n"]   ← 空段合并到前一段
i=2: splits = ["a\n\n", "\n\nb"]

text = "a\n\n"   (末尾有分隔符)
parts = ["a", ""]
i=1: part="" → splits[-1] += "\n\n" → ["a\n\n"]   ← 末尾分隔符保留，不丢失
```

---

### 3.2 `_split_text` — 递归核心（flush-before-recurse 模式）

这是整个类的大脑。理解它需要掌握两个步骤：**分隔符选择** 和 **flush-before-recurse 模式**。

#### Step 1：选择分隔符

```python
separator = separators[-1]      # 默认兜底
new_separators = []

for i, sep in enumerate(separators):
    if sep == "":
        separator = sep
        break                   # "" 是终点，后面没有更低级的了
    if sep in text:
        separator = sep
        new_separators = separators[i + 1:]  # 剩余分隔符留给递归
        break
```

**例**：text 含 `"\n\n"`，separators = `["\n\n", "\n", " ", ""]`

→ `separator = "\n\n"`，`new_separators = ["\n", " ", ""]`

#### Step 2：flush-before-recurse 循环

```python
merge_sep = "" if self._keep_separator else separator
good_splits = []

for split in splits:
    if len(split) <= self._chunk_size:
        good_splits.append(split)      # 小片段积累
    else:
        if good_splits:
            # ① 先把已积累的小片段输出，清空窗口
            final_chunks.extend(self._merge_splits(good_splits, merge_sep))
            good_splits = []
        if new_separators:
            # ② 大片段递归处理，结果直接追加，绕过本层 merge
            final_chunks.extend(self._split_text(split, new_separators))
        else:
            final_chunks.extend(self._split_by_characters(split))

if good_splits:
    final_chunks.extend(self._merge_splits(good_splits, merge_sep))
```

#### 为什么不能把递归结果放回 good_splits 再统一合并？（双重合并问题）

```
❌ 旧做法（错误）：
  递归返回已合并好的 chunk（如 800 字符）
  → 放入 good_splits
  → 再次经过 _merge_splits
  → 两个 800 字符的 chunk 被合并成 1600 字符
  → 严重超过 chunk_size

✅ flush-before-recurse（正确）：
  每层只合并本层产生的小片段
  递归结果是"黑盒"，直接透传到输出
  → 每层的 merge 各司其职，互不干扰
```

#### 完整执行可视化

以 `chunk_size=20`，text = `"A\n\nVERY_LONG_PARAGRAPH\n\nB"` 为例：

```
_split_text(text, ["\n\n", "\n", " ", ""])
│
├─ separator = "\n\n"，new_separators = ["\n", " ", ""]
├─ splits = ["A", "\n\nVERY_LONG_PARAGRAPH", "\n\nB"]
│
├─ "A" (len=1) ≤ 20 → good_splits = ["A"]
│
├─ "\n\nVERY_LONG_PARAGRAPH" (len=22) > 20
│   ├─ flush: _merge_splits(["A"], "") → ["A"]  追加到 final_chunks
│   └─ 递归: _split_text("\n\nVERY_LONG_PARAGRAPH", ["\n", " ", ""])
│       ├─ separator = "\n"，继续向下找...
│       └─ 最终返回多个小 chunk，直接追加到 final_chunks
│
└─ "\n\nB" (len=4) ≤ 20 → good_splits = ["\n\nB"]
   └─ end: _merge_splits(["\n\nB"], "") → ["\n\nB"]  追加到 final_chunks
```

**注意**："A" 和 "\n\nB" 因为被大段落隔开，分属不同批次，不会被合并——这是 flush-before-recurse 的已知限制，也是正确性的代价。

---

### 3.3 `_merge_splits` — 滑动窗口合并

接收一批**已经足够小**（≤ chunk_size）的片段，将它们合并成带 overlap 的最终 chunk。

核心是一个**滑动窗口**算法：

```python
def _merge_splits(self, splits: list[str], separator: str) -> list[str]:
    docs = []
    current_doc = []   # 当前窗口
    total = 0          # 窗口内的总字符数

    for split in splits:
        split_len = len(split)

        if total + split_len > self._chunk_size and current_doc:
            # 窗口已满 → 输出当前 chunk
            doc = "".join(current_doc)           # keep_separator=True
            # doc = separator.join(current_doc)  # keep_separator=False
            docs.append(doc)

            # 从左侧移除片段，保留 overlap 窗口
            while total > self._chunk_overlap and len(current_doc) > 0:
                removed = current_doc.pop(0)
                total -= len(removed)
                # keep_separator=False 时还要减去分隔符长度

        current_doc.append(split)
        total += split_len
        # keep_separator=False 时还要加上分隔符长度（片段间）

    if current_doc:
        docs.append("".join(current_doc))   # 最后一个 chunk

    return docs
```

#### 数字演示

`chunk_size=10`，`chunk_overlap=3`，splits = `["ab", "cde", "fg", "hij", "kl"]`

```
初始: current_doc=[], total=0

① add "ab":   total=2,  current_doc=["ab"]
② add "cde":  total=5,  current_doc=["ab","cde"]
③ add "fg":   total=7,  current_doc=["ab","cde","fg"]
④ add "hij":  7+3=10 ≤ 10，total=10，current_doc=["ab","cde","fg","hij"]
⑤ add "kl":  10+2=12 > 10 → 输出！
   → docs = ["abcdefghij"]
   → overlap loop: total=10 > 3，pop "ab"，total=8
                   total=8  > 3，pop "cde"，total=5
                   total=5  > 3，pop "fg"，total=3
                   total=3 ≤ 3，停止
   → current_doc=["hij"]，total=3
   → add "kl": total=5，current_doc=["hij","kl"]

end: docs.append("hijkl")

结果: ["abcdefghij", "hijkl"]
       overlap 区域:  "hij" (3字符) ✓
```

#### `keep_separator` 对合并的影响

| 条件 | join 方式 | 原因 |
|------|-----------|------|
| `keep_separator=True` | `"".join(current_doc)` | 分隔符已内嵌在片段中 |
| `keep_separator=False` + `separator=""` | `"".join(current_doc)` | 空分隔符没东西可插 |
| `keep_separator=False` + 实际分隔符 | `separator.join(current_doc)` | 需要把分隔符重新插回 |

同理，length tracking 只在 `keep_separator=False` 时才额外加减 `len(separator)`，因为 `True` 时分隔符长度已经计入了 `split_len`。

---

### 3.4 `_split_by_characters` — 无分隔符兜底

```python
def _split_by_characters(self, text: str) -> list[str]:
    chunks = []
    for i in range(0, len(text), self._chunk_size):
        chunks.append(text[i : i + self._chunk_size])
    return chunks
```

**触发条件**：调用者提供了**不含 `""` 的自定义 separators**，且所有分隔符用尽后仍有超大片段。

**无 overlap 的原因**：结果直接追加到 `final_chunks`，不经过 `_merge_splits`。标准 separators 列表（以 `""` 结尾）不会触发此方法——`""` 分隔符产生逐字符 splits，全部 ≤ chunk_size，由 `_merge_splits` 正常处理。

---

## 四、完整调用链路

```
BaseChunker.chunk(text)
    │ 空文本守卫
    └─ RecursiveCharacterTextSplitter.split_text(text)
           └─ _split_text(text, separators)          ← 递归入口
                  ├─ _split_text_with_separator()    ← 原子切割
                  ├─ _merge_splits()                 ← 滑动窗口合并（每层独立）
                  ├─ _split_text()                   ← 递归（大片段）
                  └─ _split_by_characters()          ← 兜底（极少触发）
```

---

## 五、中文 / 日文支持

默认 separators 含 `" "`（空格），对中日文无效（无词间空格）。当所有分隔符都未命中时，文本直接降级到逐字符拆分，性能退化为 O(n)。

解决方案：使用语言专属 separators，加入句末标点作为语义边界：

```python
SEPARATORS_ZH = ["\n\n", "\n", "。", "！", "？", "；", "，", "、", ""]
SEPARATORS_JA = ["\n\n", "\n", "。", "！", "？", "；", "、", ""]
```

通过 `BaseChunker(language=Language.CHINESE)` 自动选择，无需手动传入。

---

## 六、核心设计权衡

| 设计决定 | 解决的问题 | 代价 |
|----------|------------|------|
| 分隔符优先级列表 | 语义边界优先，避免中间截断 | 需要按语言配置不同列表 |
| flush-before-recurse | 避免双重合并导致 chunk 超出 chunk_size | 大片段两侧的小片段无法跨边界合并 |
| chunk_overlap 滑动窗口 | 防止跨 chunk 边界的信息丢失 | 存储量增加约 `overlap / chunk_size` |
| `keep_separator=True` | chunk 拼回来 = 原文，信息无损 | join 和 length tracking 逻辑更复杂 |
| `""` 作为最终 separator | 任意文本都能切到 chunk_size 以内 | 密集文本性能退化为 O(n) 字符列表 |

---

## 七、源码全文（含注释）

```python
# comet_rag/engines/chunkers/base_chunker.py

class RecursiveCharacterTextSplitter:
    """基于分隔符层次结构递归拆分文本的文本分割器"""

    def __init__(
        self,
        chunk_size: int = 4000,
        chunk_overlap: int = 200,
        separators: list[str] | None = None,
        keep_separator: bool = True,
    ) -> None:
        if chunk_size <= 0:
            raise ValueError(...)
        if chunk_overlap < 0:
            raise ValueError(...)
        if chunk_overlap >= chunk_size:
            raise ValueError(...)

        self._chunk_size = chunk_size
        self._chunk_overlap = chunk_overlap
        self._keep_separator = keep_separator
        self._separators = separators or ["\n\n", "\n", " ", ""]

    def split_text(self, text: str) -> list[str]:
        """公开入口，直接委托给 _split_text"""
        return self._split_text(text, self._separators)

    def _split_text(self, text: str, separators: list[str]) -> list[str]:
        """
        递归核心：flush-before-recurse 模式
        - 找到当前层最高优先级的有效分隔符
        - 小片段积累到 good_splits，统一交给 _merge_splits
        - 大片段在 flush 之后递归处理，结果直接追加，不经过本层 merge
        """
        final_chunks: list[str] = []
        separator = separators[-1]
        new_separators: list[str] = []

        for i, sep in enumerate(separators):
            if sep == "":
                separator = sep
                break
            if sep in text:
                separator = sep
                new_separators = separators[i + 1:]
                break

        splits = self._split_text_with_separator(text, separator)
        merge_sep = "" if self._keep_separator else separator

        good_splits: list[str] = []
        for split in splits:
            if len(split) <= self._chunk_size:
                good_splits.append(split)
            else:
                if good_splits:
                    final_chunks.extend(self._merge_splits(good_splits, merge_sep))
                    good_splits = []
                if new_separators:
                    final_chunks.extend(self._split_text(split, new_separators))
                else:
                    final_chunks.extend(self._split_by_characters(split))

        if good_splits:
            final_chunks.extend(self._merge_splits(good_splits, merge_sep))

        return final_chunks

    def _split_text_with_separator(self, text: str, separator: str) -> list[str]:
        """
        原子切割：
        - separator="" → 逐字符
        - keep_separator=True → 分隔符前置到后续片段
        - keep_separator=False → text.split(separator)，分隔符丢弃
        """
        if separator == "":
            return list(text)

        if self._keep_separator and separator in text:
            parts = text.split(separator)
            splits = []
            for i, part in enumerate(parts):
                if i == 0:
                    splits.append(part)
                else:
                    if part:
                        splits.append(separator + part)
                    elif splits:
                        splits[-1] += separator
            return [s for s in splits if s]

        return text.split(separator)

    def _split_by_characters(self, text: str) -> list[str]:
        """
        兜底切割（不含 overlap）：
        仅在自定义 separators 不含 "" 时触发
        """
        chunks = []
        for i in range(0, len(text), self._chunk_size):
            chunks.append(text[i: i + self._chunk_size])
        return chunks

    def _merge_splits(self, splits: list[str], separator: str) -> list[str]:
        """
        滑动窗口合并：
        - 积累片段直到超过 chunk_size → 输出 chunk
        - 从左侧移除片段直到 total ≤ chunk_overlap → 保留 overlap 窗口
        - keep_separator=False 时需额外追踪分隔符长度
        """
        docs: list[str] = []
        current_doc: list[str] = []
        total = 0

        for split in splits:
            split_len = len(split)

            if total + split_len > self._chunk_size and current_doc:
                doc = ("".join(current_doc)
                       if (separator == "" or self._keep_separator)
                       else separator.join(current_doc))
                if doc:
                    docs.append(doc)

                while total > self._chunk_overlap and len(current_doc) > 0:
                    removed = current_doc.pop(0)
                    total -= len(removed)
                    if separator != "" and not self._keep_separator:
                        total -= len(separator)

            current_doc.append(split)
            total += split_len
            if separator != "" and not self._keep_separator and len(current_doc) > 1:
                total += len(separator)

        if current_doc:
            doc = ("".join(current_doc)
                   if (separator == "" or self._keep_separator)
                   else separator.join(current_doc))
            if doc:
                docs.append(doc)

        return docs
```
