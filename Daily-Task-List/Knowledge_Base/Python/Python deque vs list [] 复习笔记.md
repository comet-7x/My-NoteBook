# Python deque vs list [] 复习笔记

## 一、核心结论（必记）

✅ list []：动态数组 → 擅长「随机访问、尾部增删」，头部增删极慢

✅ deque：双向链表 → 擅长「两端快速增删」，随机访问极慢

❌ 禁忌：用list做「队列、头部频繁增删」，性能灾难！

## 二、底层实现对比（根本区别）

|对比维度|list []|deque|高频考点|
|---|---|---|---|
|数据结构|动态数组（内存连续）|双向链表（内存分散）|底层决定性能差异|
|随机访问（按索引）|O(1)（极速）|O(n)（很慢，不推荐）|list核心优势|
|头部增删（insert(0)/pop(0)）|O(n)（全量移动元素）|O(1)（仅修改指针）|deque核心优势，面试高频|
|尾部增删（append/pop）|O(1)|O(1)|两者无差异|
## 三、核心操作对比（实战必背）

### 1. 尾部操作（两者一致，均高效）

```python
# list
lst = []
lst.append(1)  # 尾部添加 O(1)
lst.pop()      # 尾部删除 O(1)

# deque
dq = deque()
dq.append(1)   # 尾部添加 O(1)
dq.pop()       # 尾部删除 O(1)
```

### 2. 头部操作（天差地别，易错点）

```python
# ❌ list 头部操作（极慢，禁止用）
lst.insert(0, 1)  # O(n)，元素全量后移
lst.pop(0)        # O(n)，元素全量前移

# ✅ deque 头部操作（极速，推荐）
dq.appendleft(1)  # O(1)，deque专属方法
dq.popleft()      # O(1)，deque专属方法
```

### 3. 随机访问（list专属优势）

```python
lst[100]    # O(1) 极速（推荐）
dq[100]     # O(n) 很慢（禁止用）
```

## 四、功能差异（高频考点）

### 1. deque 独有功能（重点）

- 固定长度 `maxlen`：自动丢弃旧元素，适合滑动窗口、缓存、日志队列

- 专属方法：`appendleft()`、`popleft()`、`extendleft()`

```python
# 固定长度示例（实战常用）
dq = deque(maxlen=3)  # 最多存3个元素
dq.append(1)
dq.append(2)
dq.append(3)
dq.append(4)  # 自动丢弃最老元素1 → [2,3,4]
```

### 2. list 独有功能

- 按索引插入：`insert(index, value)`

- 排序/反转：`sort()`、`reverse()`（deque不适合排序）

- 切片操作：`lst[1:5]`（deque切片性能差，不推荐）

## 五、适用场景（直接照抄，开发不踩坑）

### ✅ 用 list [] 的场景

- 需要按索引随机访问（如 `data[5]`）

- 仅在尾部增删数据

- 需要排序、切片、遍历静态数据

### ✅ 用 deque 的场景

- 实现队列 FIFO（先进先出，工业级标准）

- 实现栈 LIFO（两端均可操作）

- 头部频繁增删数据

- 滑动窗口、固定长度缓存、日志队列（用maxlen）

- 生产者-消费者模型

## 六、实战案例（易错点对比）

### ❌ 错误：用list实现队列（性能灾难）

```python
q = []
q.append(1)   # 入队
q.pop(0)      # 出队 → O(n)，数据量大卡死！
```

### ✅ 正确：用deque实现队列（推荐）

```python
from collections import deque
q = deque()
q.append(1)    # 入队 O(1)
q.popleft()    # 出队 O(1) → 极速
```

## 七、记忆口诀（快速背诵）

列表数组查得快，头部增删慢如龟
队列链表两头快，随机访问别碰它
队列缓存用deque，普通存储用列表

## 八、复习重点（浓缩版）

1. 底层：list=动态数组，deque=双向链表

2. 性能：list随机访问O(1)，deque两端增删O(1)

3. 禁忌：list不做头部增删，deque不做随机访问

4. 亮点：deque的maxlen适合滑动窗口

5. 实战：队列必须用deque，普通存储用list
> （注：文档部分内容可能由 AI 生成）