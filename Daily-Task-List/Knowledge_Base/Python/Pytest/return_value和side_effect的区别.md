
## Python Mock 进阶：return_value vs side_effect 核心笔记
### 前置基础
`unittest.mock`（或pytest-mock）中的Mock对象用于模拟代码中的依赖（如函数、方法、接口），而`return_value`和`side_effect`是控制Mock对象调用行为的两个核心属性，也是Mock进阶使用的关键。

---

### 1. return_value：固定返回值
#### 核心定义
为Mock对象设置**固定的返回值**，无论Mock被调用时传入什么参数，都会返回这个预设值。

#### 基础用法示例
```python
from unittest.mock import Mock

# 1. 基础场景：返回简单类型
mock_add = Mock()
mock_add.return_value = 42  # 预设固定返回值
print(mock_add(1, 2))  # 输出：42（忽略入参，始终返回42）
print(mock_add(10, 20))  # 输出：42

# 2. 进阶场景：返回复杂对象（字典/列表/另一个Mock）
mock_api = Mock()
mock_api.get_user.return_value = {"id": 1, "name": "张三", "age": 25}
print(mock_api.get_user())  # 输出：{'id': 1, 'name': '张三', 'age': 25}
```

#### 适用场景
- 被模拟的函数/方法无状态，调用后始终返回同一结果（如简单的工具函数、静态数据查询）；
- 测试中只需要验证“调用行为”，无需关注返回值的动态变化。

---

### 2. side_effect：自定义执行逻辑（副作用）
#### 核心定义
字面意为“副作用”，让Mock对象被调用时执行**自定义逻辑**（而非仅返回固定值），支持三类核心场景：
1. 可迭代对象：每次调用依次返回迭代器中的下一个值；
2. 可调用对象（函数）：透传调用参数，执行自定义逻辑并返回结果；
3. 异常类/实例：调用时直接抛出指定异常。

#### 全场景示例
```python
from unittest.mock import Mock

# 场景1：可迭代对象（依次返回多值）
mock_poll = Mock()
mock_poll.side_effect = ["pending", "pending", "success"]  # 迭代取值
print(mock_poll())  # 输出：pending（第1次调用）
print(mock_poll())  # 输出：pending（第2次调用）
print(mock_poll())  # 输出：success（第3次调用）
# print(mock_poll())  # 迭代耗尽，抛出StopIteration

# 场景2：可调用对象（动态逻辑+参数透传）
def custom_calc(a, b):
    """自定义逻辑：根据入参返回结果"""
    return a * b  # 模拟根据参数动态计算

mock_calc = Mock()
mock_calc.side_effect = custom_calc
print(mock_calc(3, 4))  # 输出：12（透传参数，执行自定义逻辑）
print(mock_calc(5, 6))  # 输出：30

# 场景3：抛出异常（模拟错误场景）
mock_db = Mock()
mock_db.query.side_effect = ValueError("数据库连接失败")
try:
    mock_db.query()
except ValueError as e:
    print(e)  # 输出：数据库连接失败
```

#### 适用场景
- 需要Mock对象**每次调用返回不同值**（如分页接口、轮询接口返回不同状态）；
- 需要Mock对象**根据入参动态返回结果**（如不同用户ID返回不同信息）；
- 需要模拟函数/方法**抛出异常**（测试异常处理逻辑，如网络错误、参数错误）。

---

### 3. return_value vs side_effect 核心对比
| 特性                | return_value                | side_effect                          |
|---------------------|-----------------------------|--------------------------------------|
| 核心行为            | 返回固定值                  | 执行自定义逻辑/依次返回/抛异常       |
| 参数处理            | 忽略调用参数                | 透传参数到自定义函数（若为可调用对象）|
| 多次调用表现        | 始终返回同一值              | 迭代取值/按逻辑动态变化              |
| 优先级              | 低（被side_effect覆盖）     | 高（同时设置时优先生效）            |

#### 优先级验证示例
```python
from unittest.mock import Mock

mock_func = Mock()
mock_func.return_value = 100  # 低优先级
mock_func.side_effect = [1, 2]  # 高优先级

print(mock_func())  # 输出：1（side_effect生效）
print(mock_func())  # 输出：2（side_effect生效）
```

---

### 4. 进阶实战场景
#### 场景1：模拟接口重试逻辑（失败后成功）
```python
from unittest.mock import Mock

# 模拟接口：第1次调用抛异常，第2次返回成功
mock_api = Mock(side_effect=[ConnectionError("网络超时"), {"code": 200, "msg": "成功"}])

# 测试重试逻辑
try:
    mock_api()  # 第1次调用：抛异常
except ConnectionError:
    print("重试中...")
    result = mock_api()  # 第2次调用：返回成功
    print(result)  # 输出：{'code': 200, 'msg': '成功'}
```

#### 场景2：模拟根据参数动态返回用户信息
```python
from unittest.mock import Mock

def get_user_by_id(user_id):
    """模拟根据用户ID返回不同信息"""
    user_map = {
        1: {"name": "Alice", "age": 25},
        2: {"name": "Bob", "age": 30}
    }
    return user_map.get(user_id, {"error": "用户不存在"})

mock_get_user = Mock(side_effect=get_user_by_id)
print(mock_get_user(1))  # 输出：{'name': 'Alice', 'age': 25}
print(mock_get_user(3))  # 输出：{'error': '用户不存在'}
```

---

### 总结
1. `return_value`适用于**固定返回值**场景，调用Mock时忽略入参、始终返回同一值，用法简单；
2. `side_effect`功能更灵活，支持**依次返回多值、动态逻辑（透传参数）、抛出异常**，优先级高于`return_value`；
3. 实战中，简单固定返回用`return_value`，需要动态行为/异常模拟用`side_effect`。