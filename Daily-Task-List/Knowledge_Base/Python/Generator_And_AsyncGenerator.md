# Python 生成器（Generator）与异步生成器（AsyncGenerator）实战笔记

## 一、核心知识点总结

### 1. 生成器类型注解规则

| 类型注解             | 格式                                           | 参数含义                         | 核心差异              |
| ---------------- | -------------------------------------------- | ---------------------------- | ----------------- |
| `Generator`      | `Generator[YieldType, SendType, ReturnType]` | ① 产出值类型<br>② 接收值类型 ③ 最终返回值类型 | 支持定义最终 return 值类型 |
| `AsyncGenerator` | `AsyncGenerator[YieldType, SendType]`        | ① 产出值类型 ② 接收值类型              | 无返回值类型（PEP 规范省略）  |

- **YieldType**：生成器通过 `yield` 产出的值的类型（外部遍历获取的结果类型）；
- **SendType**：生成器通过 `gen.send(xxx)`/`agen.asend(xxx)` 接收的输入值类型（`None` 表示不接收输入）；
- **ReturnType**：仅普通生成器有，标记生成器结束时 `return` 的值类型（实际极少使用）。

### 2. 生成器函数的关键特性

- 函数体内只要包含 `yield`，调用后**必返回生成器对象**，而非 `return` 语句的值；
- 普通生成器函数的 `return` 值会被包装在 `StopIteration` 异常的 `value` 属性中，无法直接获取；
- 实现 “分支返回普通对象 / 生成器”：需将 `yield` 逻辑抽离到**内部生成器函数**，主函数仅返回普通对象或生成器对象。

### 3. 同步 / 异步生成器使用差异

|维度|同步生成器|异步生成器|
|---|---|---|
|定义方式|普通函数 + `yield`|`async def` 函数 + `yield`|
|遍历方式|普通 `for` 循环|`async for` 循环|
|休眠 / IO 模拟|`time.sleep()`|`asyncio.sleep()`|
|启动 / 发送值|`next(gen)`/`gen.send()`|`agen.asend(None)`|
|运行方式|直接调用|需通过 `asyncio.run()` 运行|

## 二、完整可运行代码

```python
from typing import Generator, AsyncGenerator
import asyncio
from time import sleep


# 基础响应类定义
class ChatAgentResponse:
    """普通聊天响应类（非流式）"""
    def __init__(self, content: str):
        self.content = content


class StreamingChatAgentResponse:
    """流式聊天响应类（逐个返回片段）"""
    def __init__(self, chunk: str):
        self.chunk = chunk


# 同步 ChatAgent 实现
class ChatAgent:
    def run(
        self,
        message: str,
        stream: bool = False,
    ) -> ChatAgentResponse | Generator[StreamingChatAgentResponse, None, None]:
        """
        同步运行聊天代理

        Args:
            message: 用户输入消息
            stream: 是否开启流式返回

        Returns:
            非流式：ChatAgentResponse 对象；流式：生成器对象
        """
        if not stream:
            # 非流式：直接返回普通响应对象
            return ChatAgentResponse(f"收到你的同步非流式消息：{message}")
        else:
            # 流式：返回内部生成器函数的执行结果（生成器对象）
            def stream_generator():
                for chunk in ["同", "步", "流", "式"]:
                    sleep(0.5)  # 模拟同步IO等待
                    yield StreamingChatAgentResponse(chunk)
            return stream_generator()


# 异步 ChatAgent 实现
class AsyncChatAgent:
    async def run(
        self,
        message: str,
        stream: bool = False,
    ) -> ChatAgentResponse | AsyncGenerator[StreamingChatAgentResponse, None]:
        """
        异步运行聊天代理
        
        Args:
            message: 用户输入消息
            stream: 是否开启流式返回

        Returns:
            非流式：ChatAgentResponse 对象；流式：异步生成器对象
        """
        if not stream:
            # 非流式：直接返回普通响应对象
            return ChatAgentResponse(f"收到你的异步非流式消息：{message}")
        else:
            # 流式：返回内部异步生成器函数的执行结果（异步生成器对象）
            async def stream_generator():
                for chunk in ["异", "步", "流", "式"]:
                    await asyncio.sleep(0.5)  # 模拟异步IO等待
                    yield StreamingChatAgentResponse(chunk)
            return stream_generator()


# 同步测试函数
def test_non_streaming_agent():
    """测试同步非流式响应"""
    agent = ChatAgent()
    normal_res = agent.run("测试同步非流式", stream=False)
    print("=== 同步非流式测试结果 ===")
    if isinstance(normal_res, ChatAgentResponse):
        print(normal_res.content)
    print("\n")


def test_streaming_agent():
    """测试同步流式响应"""
    agent = ChatAgent()
    stream_gen = agent.run("测试同步流式", stream=True)
    print("=== 同步流式测试结果 ===")
    if not isinstance(stream_gen, ChatAgentResponse):
        for res in stream_gen:
            print(res.chunk, end="", flush=True)
    print("\n\n")


# 异步测试函数
async def test_async_non_streaming_agent():
    """测试异步非流式响应"""
    agent = AsyncChatAgent()
    normal_res = await agent.run("测试异步非流式", stream=False)
    print("=== 异步非流式测试结果 ===")
    if isinstance(normal_res, ChatAgentResponse):
        print(normal_res.content)
    print("\n")


async def test_async_streaming_agent():
    """测试异步流式响应"""
    agent = AsyncChatAgent()
    stream_gen = await agent.run("测试异步流式", stream=True)  # 异步生成器无需 await
    print("=== 异步流式测试结果 ===")
    if not isinstance(stream_gen, ChatAgentResponse):
        async for res in stream_gen:
            print(res.chunk, end="", flush=True)
    print("\n")


# 主函数：按需执行测试
if __name__ == "__main__":
    # 运行同步测试
    # test_non_streaming_agent()
    # test_streaming_agent()

    # 运行异步测试
    asyncio.run(test_async_non_streaming_agent())
    asyncio.run(test_async_streaming_agent())
```