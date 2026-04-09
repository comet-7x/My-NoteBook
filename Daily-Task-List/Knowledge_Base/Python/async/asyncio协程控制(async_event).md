# asyncio 协程控制 全套实战教程

核心工具：

1. **`asyncio.Event`**：协程信号通知（等待、触发、广播）
2. **任务取消**：优雅取消（推荐）+ 强制取消
3. **启停控制**：封装为可复用的控制器（适配你的 Claude Code 场景）

---

## 一、基础：asyncio.Event 信号通知

`Event` 是协程之间的**信号开关**，用于：等待信号、触发信号、多协程广播。

### 1. 单协程等待 + 触发信号

```python
import asyncio

async def wait_signal(event: asyncio.Event):
    """协程：等待信号触发后执行"""
    print("[等待协程] 等待信号中...")
    await event.wait()  # 阻塞，直到信号被set()
    print("[等待协程] 收到信号！开始执行任务")

async def trigger_signal(event: asyncio.Event):
    """协程：2秒后触发信号"""
    await asyncio.sleep(2)
    print("[触发协程] 发送信号！")
    event.set()  # 触发信号

async def main():
    event = asyncio.Event()  # 1. 创建信号对象
    # 并发运行两个协程
    await asyncio.gather(wait_signal(event), trigger_signal(event))

asyncio.run(main())
```

### 2. 多协程共享信号（广播通知）

一个信号，**唤醒所有等待的协程**：

```python
import asyncio

async def worker(id: int, event: asyncio.Event):
    print(f"工人{id} → 等待开工信号")
    await event.wait()
    print(f"工人{id} → 收到信号，开始工作！")

async def main():
    event = asyncio.Event()
    # 创建3个协程，共享同一个信号
    tasks = [worker(1, event), worker(2, event), worker(3, event)]
    # 并发启动协程
    asyncio.create_task(asyncio.gather(*tasks))
    
    await asyncio.sleep(2)
    print("=== 发送开工信号 ===")
    event.set()  # 一次性唤醒所有协程
    await asyncio.sleep(1)

asyncio.run(main())
```

---

## 二、核心：协程任务 取消控制（两种方案）

### 方案 1：优雅取消（推荐 ✅）

用 `Event.is_set()` 轮询检测，**可安全清理资源、终止任务**：


```python
import asyncio

async def long_task(cancel_event: asyncio.Event):
    """可取消的耗时任务"""
    print("任务开始运行...")
    for i in range(10):
        # 核心：实时检测取消信号
        if cancel_event.is_set():
            print("检测到取消信号！任务终止，清理资源完成")
            return
        
        print(f"执行步骤 {i+1}")
        await asyncio.sleep(1)  # 模拟异步操作
    print("任务正常完成！")

async def main():
    cancel_event = asyncio.Event()
    # 启动任务
    task = asyncio.create_task(long_task(cancel_event))
    
    # 模拟：3秒后外部取消任务
    await asyncio.sleep(3)
    print("\n外部：触发取消任务信号")
    cancel_event.set()
    
    await task  # 等待任务结束

asyncio.run(main())
```

### 方案 2：强制取消（原生 asyncio 方法）

直接调用 `task.cancel()`，强制终止协程，配合异常捕获：

```python
import asyncio

async def long_task():
    try:
        print("任务开始运行...")
        for i in range(10):
            print(f"执行步骤 {i+1}")
            await asyncio.sleep(1)
    except asyncio.CancelledError:
        # 捕获取消异常，做清理操作
        print("任务被强制取消！执行清理...")

async def main():
    task = asyncio.create_task(long_task())
    await asyncio.sleep(3)
    print("强制取消任务")
    task.cancel()  # 原生强制取消
    await task

asyncio.run(main())
```

---

## 三、高级实战：生产级 协程启停控制器

封装为**类**，实现：**启动、暂停、停止、运行状态判断**，完美适配你的项目：

```python
import asyncio

class TaskController:
    """协程任务 启停/取消 控制器（生产可用）"""
    def __init__(self):
        self.run_event = asyncio.Event()  # 运行信号
        self.pause_event = asyncio.Event()  # 暂停信号
        self.is_running = False

    async def start(self):
        """启动任务"""
        self.run_event.set()
        self.pause_event.clear()
        self.is_running = True
        print("控制器：任务已启动")

    async def pause(self):
        """暂停任务"""
        self.pause_event.set()
        print("控制器：任务已暂停")

    async def resume(self):
        """恢复任务"""
        self.pause_event.clear()
        print("控制器：任务已恢复")

    def stop(self):
        """停止任务（永久终止）"""
        self.run_event.clear()
        self.is_running = False
        print("控制器：任务已停止")

async def worker(controller: TaskController):
    """业务协程"""
    while True:
        # 1. 检测是否停止
        if not controller.run_event.is_set():
            print("工作协程：收到停止信号，退出")
            break
        
        # 2. 检测是否暂停
        if controller.pause_event.is_set():
            await asyncio.sleep(0.1)
            continue

        # 3. 执行业务逻辑
        print("工作中...")
        await asyncio.sleep(1)

# 测试控制器
async def main():
    controller = TaskController()
    # 启动任务
    task = asyncio.create_task(worker(controller))
    await controller.start()

    await asyncio.sleep(2)
    await controller.pause()  # 暂停
    
    await asyncio.sleep(2)
    await controller.resume()  # 恢复
    
    await asyncio.sleep(2)
    controller.stop()  # 停止
    
    await task

asyncio.run(main())
```

---

## 四、进阶：带超时的等待（避免无限阻塞）

防止协程无限等待信号，增加**超时控制**：

```python
import asyncio

async def wait_with_timeout(event: asyncio.Event, timeout: int):
    try:
        # 等待信号，最多等待timeout秒
        await asyncio.wait_for(event.wait(), timeout=timeout)
        print("成功收到信号！")
    except asyncio.TimeoutError:
        print(f"等待超时！{timeout}秒内未收到信号")

async def main():
    event = asyncio.Event()
    # 不触发信号，测试超时
    await wait_with_timeout(event, 3)

asyncio.run(main())
```

---

# 核心知识点总结

1. **信号通知**：`asyncio.Event()` → `wait()` 等待、`set()` 触发、`is_set()` 判断
2. **任务取消**
    
    - 优雅取消：用 `Event` 轮询，安全清理资源（**推荐用于你的项目**）
    - 强制取消：`task.cancel()`，配合 `CancelledError` 异常
    
3. **启停控制**：封装多个 `Event`，实现启动 / 暂停 / 停止
4. **共享特性**：`Event` 是引用对象，跨函数 / 协程传递**状态完全同步**

所有代码都可以直接复制运行，你可以根据自己的 Claude Code 场景，直接复用**优雅取消**和**启停控制器**的代码！