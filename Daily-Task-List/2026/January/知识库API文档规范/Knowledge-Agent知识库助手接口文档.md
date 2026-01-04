# Knowledge Agent 接口文档

本文档详细介绍了 Knowledge-Agent 服务提供的所有 API 接口。

## 服务信息

- **Base URL**: `http://localhost:48585/steins/alg/knowledge-agent`
- **版本**: 0.1.0

---

## 1. 健康检查 (Health Check)

用于验证服务是否正常运行。

- **接口路径**: `/health`
- **请求方法**: `GET`
- **请求参数**: 无

### 响应说明

响应体为 JSON 格式。

| 字段 | 类型 | 说明 | 示例 |
| :--- | :--- | :--- | :--- |
| `name` | string | 服务名称 | `"knowledge-agent"` |
| `status` | string | 服务状态 | `"healthy"` |
| `version` | string | 服务版本 | `"0.1.0"` |
| `datetime` | string | 当前服务端时间 | `"2026-01-04 02:31:58 UTC"` |

### 示例

**Request:**

```bash
curl --request GET \
  --url http://localhost:48585/steins/alg/knowledge-agent/health
```

**Response:**

```json
{
  "name": "knowledge-agent",
  "status": "healthy",
  "version": "0.1.0",
  "datetime": "2026-01-04 02:31:58 UTC"
}
```

---

## 2. 会话标题总结 (Session Summary)

根据用户的问题和模型的回答，生成简短的会话标题。

- **接口路径**: `/summary`
- **请求方法**: `POST`
- **Content-Type**: `application/json`

### 请求参数

请求体为 JSON 对象。

| 字段 | 类型 | 必填 | 说明 |
| :--- | :--- | :--- | :--- |
| `question` | string | 是 | 用户的问题 |
| `answer` | string | 是 | 模型的回答，默认为空字符串 |

### 响应说明

| 字段 | 类型 | 说明 |
| :--- | :--- | :--- |
| `status` | string | 请求状态，`"success"` 或 `"error"` |
| `data` | string | 生成的标题内容或错误信息 |

### 示例

**Request:**

```bash
curl --request POST \
  --url http://localhost:48585/steins/alg/knowledge-agent/summary \
  --header 'content-type: application/json' \
  --data '{
  "question": "鱿鱼的品种有哪些？",
  "answer": "鱿鱼，又叫柔鱼、枪乌贼..."
}'
```

**Response:**

```json
{
  "status": "success",
  "data": "常见鱿鱼品种及其特点总结"
}
```

---

## 3. 知识库问答 (Knowledge Base Chat)

核心问答接口，支持流式返回 (Server-Sent Events, SSE)。支持“思考模式”和“快速模式”。

- **接口路径**: `/chat`
- **请求方法**: `POST`
- **Content-Type**: `application/json`
- **响应类型**: `text/event-stream`

### 请求参数

| 字段 | 类型 | 必填 | 默认值 | 说明 |
| :--- | :--- | :--- | :--- | :--- |
| `session_id` | string | 是 | - | 会话 ID，由前端创建返回（每个用户的每个会话的独立id） |
| `message_id` | string | 是 | - | 消息 ID，由前端创建返回（每个用户的每个会话内的每条消息的独立id） |
| `message` | string | 是 | - | 用户的问题 |
| `mode` | string | 否 | `"thinking"` | 回答模式。可选值：`"thinking"` (思考模式), `"fast"` (快速模式) |
| `model` | string | 否 | `"Qwen3-30B-A3B"` | 模型名称 |
| `tags` | list[str] | 否 | `[]` | 知识库分区内的字段标签 |
| `partitions` | list[str] | 否 | `null` | 知识库分区名称列表 |
| `files` | list[str] | 否 | `[]` | 知识库文件 ID 列表 |
| `top_k` | int | 否 | `10` | 向量数据库查询返回的个数 |

### SSE 响应事件结构

接口返回一系列 SSE 事件。每个事件包含 `event` (事件类型) 和 `data` (JSON 数据)。

#### 公共数据字段 (BaseData)

所有事件的 `data` 部分都包含以下字段：

| 字段 | 类型 | 说明 |
| :--- | :--- | :--- |
| `message_id` | string | 数据节点的唯一 ID |
| `time_cost` | int | 消息处理耗时 (毫秒)，目前只有工具调用和结束节点提供该字段的值，其他默认为0 |

#### 事件类型列表

**1. `SEND_CHAT_START`**
   - **说明**: 对话开始。
   - **Data**:
     - `data`: string (固定文本 "the message of {message_id} has started")

**2. `SEND_THOUGHT_CONTENT`**
   - **说明**: 思考过程内容块 (流式)。
   - **Data**:
     - `node_type`: `"thought"`
     - `node_id`: string (节点 ID)
     - `data`: string (文本内容块)

**3. `SEND_COMMON_CONTENT`**
   - **说明**: 普通回答内容块 (流式)。
   - **Data**:
     - `node_type`: `"content"`
     - `node_id`: string (节点 ID)
     - `data`: string (文本内容块)

**4. `SEND_TOOL_CALL`**

   - **说明**: 工具调用信息。
   - **Data**:
     - `node_type`: `"tool_use"`
     - `node_id`: string (节点 ID)
     - `data`: Object (工具调用详情)
       - `tool_type`: string (工具名称)
       - `title`: string (展示标题)
       - `desc`: string (描述)
       - `tool_result`: Any (工具执行结果)

**5. `SEND_CHAT_END`**
   - **说明**: 对话结束。
   - **Data**:
     - `data`: string (固定文本 "the message of {message_id} has ended")

**6. `SEND_CHAT_ERROR`**
   - **说明**: 发生错误。
   - **Data**:
     - `data`: string (错误信息)

### 示例

**Request:**

```bash
curl --request POST \
  --url http://localhost:48585/steins/alg/knowledge-agent/chat \
  --header 'content-type: application/json' \
  --data '{
  "session_id": "sess_12345",
  "message_id": "msg_67890",
  "message": "搜索鱿鱼胴体蛋白制备及其应用特性研究",
  "mode": "thinking"
}'
```

**Response Stream (示例片段):**

```text
event: SEND_CHAT_START
data: {"message_id": "msg_67890", "time_cost": 0, "data": "the message of msg_67890 has started"}

event: SEND_THOUGHT_CONTENT
data: {"message_id": "msg_67890", "time_cost": 0, "node_type": "thought", "node_id": "node_1", "data": "我需要"}

event: SEND_TOOL_CALL
data: {"message_id":"msg_67890","time_cost":2327,"node_type":"tool_use","node_id":"7413416823510704128","data":{"tool_type":"rag_search","title":"RAG搜索结果","desc":"RAG搜索工具的响应数据","tool_result":[{"title":"秘鲁鱿鱼的除酸和冷冻鱿鱼滑品质保障的研究_焦甜甜.pdf","id":"621092937517764608","content":"秘鲁鱿鱼是美洲大赤鱿（Dosidicus gigas）的俗称..."},...]}}

event: SEND_THOUGHT_CONTENT
data: {"message_id": "msg_67890", "time_cost": 0, "node_type": "thought", "node_id": "node_1", "data": "查询知识库..."}

event: SEND_COMMON_CONTENT
data: {"message_id": "msg_67890", "time_cost": 0, "node_type": "content", "node_id": "node_2", "data": "Knowledge-Agent 是..."}

event: SEND_CHAT_END
data: {"message_id": "msg_67890", "time_cost": 19176, "data": "the message of msg_67890 has ended"}
```
