# Python @dataclass 详解

`@dataclass` 是 Python 3.7+ 引入的一个装饰器，可以自动为类生成常用的“样板方法”，让开发者能更专注于数据本身，而不是重复编写基础方法。

## 核心功能

当你在一个类上使用 `@dataclass` 装饰器时，它会根据你定义的类属性，自动生成以下一个或多个方法：

- `__init__(self, ...)`: 构造函数，用于初始化实例属性。
- `__repr__(self)`: 提供一个清晰的、可供开发者阅读的字符串表示形式。
- `__eq__(self, other)`: 实现基于实例属性的相等性比较 (`==`)。
- `__ne__(self, other)`: 实现不等比较 (`!=`)。
- `__lt__(self, other)`, `__le__(self, other)`, `__gt__(self, other)`, `__ge__(self, other)`: 实现排序比较 (`<`, `<=`, `>`, `>=`)，但需要将 `@dataclass(order=True)` 设置为 `True`。
- `__hash__`: 如果合适，会生成哈希方法，使数据类的实例可以被添加到集合（`set`）或用作字典（`dict`）的键。

## 基础示例

```python
from dataclasses import dataclass

@dataclass
class InventoryItem:
    """一个简单的库存物品数据类"""
    name: str
    unit_price: float
    quantity_on_hand: int = 0

# 无需手动编写 __init__，可以直接创建实例
item1 = InventoryItem(name="Laptop", unit_price=1200.0, quantity_on_hand=5)
item2 = InventoryItem(name="Laptop", unit_price=1200.0, quantity_on_hand=5)
item3 = InventoryItem(name="Mouse", unit_price=25.0)

# __repr__ 是自动生成的
print(item1)
# 输出: InventoryItem(name='Laptop', unit_price=1200.0, quantity_on_hand=5)

# __eq__ 是自动生成的
print(item1 == item2)
# 输出: True

print(item1 == item3)
# 输出: False
```

`@dataclass` 极大地简化了以存储数据为主要目的的类的代码，使其更简洁、更易读。