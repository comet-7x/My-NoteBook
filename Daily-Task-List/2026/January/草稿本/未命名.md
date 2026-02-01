
```java
declare namespace Message {
    enum MessageNodeType {
        CONTENT = "content",  // 普通文本
        REASONING_CONTENT = "reasoning_content",  // 思考文本
        TOOL_USE = "tool_use"  // 工具调用
    }

    // 工具调用的类型
    enum ToolUseType {
        WEB_SEARCH = "web_search",  // 网络搜索
        RAG_SEARCH = "rag_search",  // 知识库检索
    }
    interface ChatMessage {
        messageId: string
        role: "system" | "user" | "assistant"
        ableShowTools: boolean;  // 展示消息工具栏
        ableView: boolean;  // 该消息是否被查看
        nodes: Record<string, MessageNode>
    }

    // 消息节点
    interface MessageNode {
        nodeId: string  // 节点id
        type: MessageNodeType  // 节点类型
        loading: boolean   // 节点是否加载中
        data: string | ToolUseData  // 节点数据
    }

    // 工具节点的数据类型
    interface ToolUseData {
        type: ToolUseType
        title: string   // 工具标题
        desc?: string // 工具描述
        data: RagSearchToolData[] | WebSearchToolData[]
    }


    // rag search工具返回的数据结构
    interface RagSearchToolData {
        title: string  // rag检索的片段标题
        content: string  // rag检索的片段内容
        filepath: string  // rag检索的原文路径
    }

    // web search工具返回的数据结构
    interface WebSearchToolData {
        link: string 
        title: string
        description: string
    }

}
```