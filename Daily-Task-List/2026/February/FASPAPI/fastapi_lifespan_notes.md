# `FastAPI Lifespan` 与 `asynccontextmanager`

## 为什么FastAPI的lifespan函数需要使用`asynccontextmanager`这个装饰器来进行装饰？

FastAPI 使用 `asynccontextmanager` 来装饰 `lifespan` 函数，是为了提供一个清晰、可靠且结构化的方式来管理应用程序生命周期中的 **“启动”** 和 **“关闭”** 事件。

这个装饰器巧妙地利用了Python的上下文管理器（Context Manager）协议，将其应用于应用的整个生命周期。

### 工作原理详解

1.  **什么是上下文管理器？**
    在Python中，你最熟悉的上下文管理器就是 `with` 语句，比如 `with open("file.txt") as f:`。它保证无论 `with` 代码块内部发生什么（即使是异常），都会执行一个“进入”（`__enter__`）操作和一个“退出”（`__exit__`）操作。这对于确保资源（如文件、数据库连接）被正确释放至关重要。

2.  **`@asynccontextmanager` 的作用**
    这个装饰器来自于Python标准库 `contextlib`，它允许你用一个更简单的方式（使用 `yield`）来创建一个异步上下文管理器，而无需手动编写一个包含 `__aenter__` 和 `__aexit__` 方法的类。

    它的工作模式如下：
    *   `yield` 关键字之前的所有代码，都相当于“进入”逻辑（`__aenter__`）。
    *   `yield` 关键字本身，是程序执行权交出的地方。
    *   `yield` 关键字之后的所有代码，都相当于“退出”逻辑（`__aexit__`），并且这部分代码被隐式地放在一个 `try...finally` 结构中，确保它总能被执行。

3.  **FastAPI 如何应用它？**
    你可以把整个FastAPI应用想象成是运行在 `lifespan` 上下文管理器的 `with` 代码块内部。

    *   **应用启动时**：FastAPI会执行 `lifespan` 函数中 `yield` 之前的部分。这是你放置初始化代码的理想位置，例如：
        *   创建数据库连接池。
        *   加载机器学习模型到内存。
        *   连接到消息队列。
        *   初始化后台任务。

    *   **应用运行时**：`yield` 执行后，FastAPI开始接收和处理HTTP请求。你的API端点会正常工作。

    *   **应用关闭时**：当FastAPI应用停止时（例如，你停止了Uvicorn服务器），它会继续执行 `lifespan` 函数中 `yield` 之后的部分。这是你放置清理代码的理想位置，例如：
        *   关闭数据库连接池。
        *   释放模型占用的资源。
        *   断开与外部服务的连接。

### 代码示例

下面是一个典型的 `lifespan` 用法，可以让你更清晰地理解这个流程：

```python
# main.py
from contextlib import asynccontextmanager
from fastapi import FastAPI
import time

# 假设这是一个需要加载的昂贵资源，比如AI模型
ml_models = {}

@asynccontextmanager
async def lifespan(app: FastAPI):
    # --- 应用启动时执行 ---
    print("应用启动中...开始加载模型...")
    # 模拟加载模型
    time.sleep(2) 
    ml_models["model_a"] = "这是一个非常大的模型A"
    ml_models["model_b"] = "这是一个非常大的模型B"
    print("模型加载完成！")

    yield  # 将执行权交给FastAPI应用

    # --- 应用关闭时执行 ---
    print("应用关闭中...开始清理资源...")
    ml_models.clear()
    print("资源清理完成！")


app = FastAPI(lifespan=lifespan)

@app.get("/")
async def root():
    # 在请求中可以使用启动时加载的资源
    return {"message": "应用已启动", "loaded_models": list(ml_models.keys())}
```

当你用 `uvicorn main:app --reload` 启动这个应用时，你会在终端看到：

```bash
应用启动中...开始加载模型...
模型加载完成！
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
...
```

当你用 `CTRL+C` 停止应用时，你会看到：

```bash
...
INFO:     Shutting down
应用关闭中...开始清理资源...
资源清理完成！
```

### 总结：为什么这种方式更好？

*   **结构清晰**：将启动和关闭逻辑集中在一个函数中，代码更内聚，易于理解和维护。
*   **保证执行**：`asynccontextmanager` 的设计保证了无论应用如何退出，清理代码（`yield`之后的部分）都会被执行，防止资源泄露。
*   **状态共享**：可以通过 `yield` 传递一个状态字典，供应用在不同部分（如依赖注入）中使用，这比使用全局变量更安全、更明确。
*   **符合Pythonic风格**：它遵循了Python中管理资源的最佳实践（上下文管理器模式）。