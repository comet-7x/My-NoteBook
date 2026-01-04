该文档主要用于介绍知识库Agent所有的接口信息


## 一、健康检查接口：
本接口用于验证**Knowledge-Agent**知识库助手接口是否存在，如果这个接口无法访问，那么说明整个服务不在运行中或者服务挂掉了。

### 1.请求信息：
- 方法：`GET`
- 路径：`/health`
- 完整URL：`http://localhost:48585/steins/alg/knowledge-agent/health`


### 2.请求示例：
```bash
curl --request GET \
  --url http://localhost:48587/steins/alg/knowledge-agent/health
```
