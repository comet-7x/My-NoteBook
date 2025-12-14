## 1. 什么是 $ReAct$？

**$ReAct$** 是 **Re**asoning（推理）与 **Act**ing（行动）的缩写。

- **核心理念：** 只有推理（如 Chain-of-Thought）容易产生幻觉，只有行动（如简单的 API 调用）缺乏灵活性。ReAct 将两者结合，让大模型在执行任务时，交替进行 **“思考 -> 行动 -> 观察”** 的循环。
    
- **工作流程：**
    
    1. **Thought (思考)：** 模型分析当前情况，决定下一步做什么。
        
    2. **Action (行动)：** 模型决定调用什么工具（如搜索、计算器、数据库），并生成参数。
        
    3. **Observation (观察)：** 外部代码执行工具，将结果返回给模型。
        
    4. **循环：** 模型根据观察结果再次思考，直到任务完成。

## 2. 如何实现 $ReAct$？

实现 $ReAct$ 主要有两种方式：**原生 Prompt 实现（手写循环）** 和 **使用 Agent 框架（如 `LangChain/LangGraph`）**。
### 方式一：原生 Prompt 实现 (理解原理)

这是 $ReAct$ 的本质。你需要构建一个 Prompt 模板，并写一个 Python `while` 循环来驱动它。

**Prompt 模板示例：**
```python
Answer the following questions as best you can. You have access to the following tools:

{tools}

Use the following format:

Question: the input question you must answer
Thought: you should always think about what to do
Action: the action to take, should be one of [{tool_names}]
Action Input: the input to the action
Observation: the result of the action
... (this Thought/Action/Action Input/Observation can repeat N times)
Thought: I now know the final answer
Final Answer: the final answer to the original input question

Begin!

Question: {input}
Thought:{agent_scratchpad}
```


**代码逻辑 (伪代码)：**
```python
history = prompt_template.format(user_input="特斯拉现在的股价乘以2是多少？")

while True:
    # 1. 让大模型生成回复
    response = llm.predict(history)
    
    # 2. 判断是否结束
    if "Final Answer:" in response:
        print(response)
        break
    
    # 3. 解析 Action 和 Action Input
    action, action_input = parse_response(response)
    
    # 4. 执行工具 (Observation)
    observation = run_tool(action, action_input)
    
    # 5. 将结果拼接回历史上下文，继续循环
    history += f"\n{response}\nObservation: {observation}\n"
```