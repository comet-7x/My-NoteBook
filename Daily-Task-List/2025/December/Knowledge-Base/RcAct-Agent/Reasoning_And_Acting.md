## 1. 什么是 ReAct？

**ReAct** 是 **Re**asoning（推理）与 **Act**ing（行动）的缩写。

- **核心理念：** 只有推理（如 Chain-of-Thought）容易产生幻觉，只有行动（如简单的 API 调用）缺乏灵活性。ReAct 将两者结合，让大模型在执行任务时，交替进行 **“思考 -> 行动 -> 观察”** 的循环。
    
- **工作流程：**
    
    1. **Thought (思考)：** 模型分析当前情况，决定下一步做什么。
        
    2. **Action (行动)：** 模型决定调用什么工具（如搜索、计算器、数据库），并生成参数。
        
    3. **Observation (观察)：** 外部代码执行工具，将结果返回给模型。
        
    4. **循环：** 模型根据观察结果再次思考，直到任务完成。





```json
{
	"text": "文本",
	"title": "标题", 
	"equation": "行间公式",
	"image": "图片",
	"image_caption": "图片描述",
	"image_footnote": "图片脚注",
	"table": "表格",
	"table_caption": "表格描述",
	"table_footnote": "表格脚注",
	"phonetic": "拼音",
	"code": "代码块",
	"code_caption": "代码描述",
	"ref_text": "参考文献",
	"algorithm": "算法块",
	"list": "列表",
	"header": "页眉",
	"footer": "页脚",
	"page_number": "页码",
	"aside_text": "装订线旁注", 
	"page_footnote": "页面脚注"
}
```

```json
{
    "id": "数据唯一性标识",
    "common_vector": "文本切片的密集向量",
    "bm25_vector": "文本切片的BM25向量",
    "multimodal_vector": "图像的多模态向量",
    "file_id": "文件唯一性标识",
    "file_name": "文件名",
    "file_path": "文件路径",
    "chunk": "文本切片内容",
    "payload": "时间戳",
    "$meta": "其他元数据"
}
```

```json
{
    "text": "文本",
    "title": "标题",
    "equation": "行间公式",
    "image": "图片",
    "image_caption": "图片描述",
    "image_footnote": "图片脚注",
    "table": "表格",
    "table_caption": "表格描述",
    "table_footnote": "表格脚注",
    "phonetic": "拼音",
    "code": "代码块",
    "code_caption": "代码描述",
    "ref_text": "参考文献",
    "algorithm": "算法块",
    "list": "列表",
    "header": "页眉",
    "footer": "页脚",
    "page_number": "页码",
    "aside_text": "装订线旁注",
    "page_footnote": "页面脚注"
}
```

