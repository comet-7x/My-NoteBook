# Milvus Schema与Function详解

## 1. `add_field()`

函数  [`add_field()`](https://milvus.io/api-reference/pymilvus/v2.6.x/ORM/CollectionSchema/add_field.md#Examples)用于创建数据字段：

- `field_name: str` ：用于指定数据字段名字

- <details>        
      <summary> <code> datatype: DataType</code> ：用于指定数据字段的类型：</summary>     
      <ul>         
          <li> 主键字段（Primary key）：             
              <ul>                 
                  <li> <code> DataType.INT64 </code> </li>                 
                  <li> <code> DataType.VARCHAR </code> </li>             
              </ul>         
          </li>         
          <li> 标量字段（Scalar fields）：             
              <ul>                 
                  <li> <code> DataType.BOOL </code> </li>                 
                  <li> <code> DataType.INT8 </code> </li>                 
                  <li> <code> DataType.INT16 </code> </li>                 
                  <li> <code> DataType.INT32 </code> </li>                 
                  <li> <code> DataType.INT64 </code> </li>                 
                  <li> <code> DataType.FLOAT </code> </li>                 
                  <li> <code> DataType.DOUBLE </code> </li>                 
                  <li> <code> DataType.BINARY_VECTOR </code> </li>                 
                  <li> <code> DataType.FLOAT_VECTOR </code> </li>                 
                  <li> <code> DataType.FLOAT16_VECTOR </code> </li>                 
                  <li> <code> DataType.BFLOAT16_VECTOR </code> </li>                 
                  <li> <code> DataType.VARCHAR </code> </li>                 
                  <li> <code> DataType.JSON </code> </li>                 
                  <li> <code> DataType.ARRAY </code> </li>             
              </ul>         
          </li>         
          <li> 向量字段（Vector fields）：             
              <ul>                 
                  <li> <code> DataType.BINARY_VECTOR </code> </li>                 
                  <li> <code> DataType.FLOAT_VECTOR </code> </li>                 
                  <li> <code> DataType.FLOAT16_VECTOR </code> </li>                 
                  <li> <code> DataType.BFLOAT16_VECTOR </code> </li>                 
                  <li> <code> DataType.SPARSE_FLOAT_VECTOR </code> </li>             
              </ul>         
          </li>     
      </ul> 
  </details>

- `dim: int` ：用于指定向量数据字段的向量维度

- `max_length: int` ：用于指定允许插入的字符串的最大字节长度，Milvus 我猜测是 **UTF-8** 格式（没有在文档里面看见）：

  - **ASCII 编码**：仅支持英文字符、数字、符号，单个字符占 **1 字节**（如 `"a"` 的字节长度为 1）。
  - **GBK/GB2312 编码**：英文字符占 1 字节，汉字占 **2 字节**（如 `"你好a"` 的 GBK 字节长度为 `2+2+1=5`。
  - **UTF-8 编码**：英文字符占 1 字节，汉字占 **3 字节**（部分生僻字占 4 字节），如 `"你好a"` 的 UTF-8 字节长度为 `3+3+1=7`。
  - **UTF-16 编码**：大部分字符占 2 字节，生僻字占 4 字节（如 `"你好a"` 的 UTF-16 字节长度为 `2+2+2=6`）。

- `nullable: bool`：用于指定数据字段能否接收空值。

- `enable_analyzer: bool`：用于指定 `VARCHAR` 类型字段是否做文本分析（分词+过滤），如果设置为`True`，该字段会启用  [**analyzer**（分析器）](https://milvus.io/docs/zh/analyzer-overview.md)，把原始文本切成 token，并按配置做小写化、停用词过滤等预处理。这个 **analyzer** 的行为由 `analyzer_params` 决定，例如：

  ```json
  analyzer_params = {
      "type": "standard" # Uses the standard built-in analyzer
  }
  ```

  - Milvus 内置的预配置分析器类型，可以通过指定其名称直接使用。可选的值：
    - `standard`：适用于通用文本处理，应用标准标记化和小写过滤。
    - `english`：针对英语文本进行了优化，支持英语停止词。
    - `chinese`：专门用于处理中文文本，包括针对中文语言结构的标记化。
  - 简单来说：`enable_analyzer`负责“如何把文本拆成 token，并进行规范化”，是**预处理阶段**。

- `enable_match: bool`用于指定 `VARCHAR` 类型字段是否启用关键词匹配，如果设置为`True`，会对该字段创建倒排索引，然后我们就可以对这个字段使用 ([`TEXT_MATCH(...)`](https://milvus.io/docs/zh/keyword-match.md#Enable-text-match))、[`PHRASE_MATCH(...)`](https://milvus.io/docs/zh/phrase-match.md#Enable-phrase-match) 这类基于 **term / phrase** 的匹配表达式来做过滤或与向量检索组合。

  - 简单来说：`enable_match=True`“负责是否基于这些 token 建倒排索引并提供关键词/短语匹配能力”，是**索引与检索阶段**。

<img src="https://milvus.io/docs/v2.6.x/assets/keyword-match.png" alt="image-20250613130203914" style="zoom:90%;" />

## 2. `Function()`

`Function()`：用于为某个原始字段自动生成向量/稀疏向量，或做重排的内置函数配置：
- `name: str`：定义函数的名字
- `input_field_names: str`：需要处理的输入字段（只接受一个字段）
- `output_field_names: str`：输出处理结果的字段（只接受一个字段）
- `function_type: FunctionType`：定义数据的内置处理操作，可以使用的嵌入函数类型有：
  - `FunctionType.BM25` : 基于 `VARCHAR` 字段的 [BM25](https://mloasisblog.com/blog/ML/RAG-Retrieval-Strategy#:~:text=%E4%BB%BB%E4%BD%95%E4%B8%80%E7%A7%8D%E6%96%B9%E6%B3%95%E3%80%82-,BM25,-BM25%EF%BC%88Best%20Matching) 排名算法生成稀疏向量。
  - `FunctionType.TEXTEMBEDDING` : 基于 `VARCHAR` 字段生成捕捉语义意义的密集向量。
  - `FunctionType.RERANK` : 对搜索结果应用重新排序策略。
-   `params: dict`：嵌入/排序函数的配置字典。支持的键因 `function_type` 而异：

## 3. `add_function()`

(内容待补充)
