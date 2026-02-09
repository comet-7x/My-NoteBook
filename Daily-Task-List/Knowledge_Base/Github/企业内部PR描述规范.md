# 企业内部 Pull Request 描述规范（Markdown 笔记）

> 适用场景：企业内部 GitHub / GitLab / Gitee 等代码仓库  
> 目标：**降低沟通成本、提升 Review 效率、减少上下文依赖**

---

## 一、PR Description 的核心原则

在企业内部协作中，Pull Request 的描述并不是“形式化文本”，而是：

> **Reviewer 在不找你、不翻 Issue、不读全部代码的前提下，理解这个 PR 的唯一入口。**

一个合格的 PR 描述，应当回答清楚以下三个问题：

1. **为什么要改（Why）**
    
2. **改了什么（What）**
    
3. **会影响什么（Impact）**
    

---
[#SHOU-353 算法-知识库Agent 路径前缀匹配](https://applink.feishu.cn/client/mini_program/open?appId=cli_9e9150b90ffb1108&mode=appCenter&relaunch=true&path=pages%2Fpc%2Findex%3Freturn_url%3D%2Fpjm%2Fitems%2F69816b105e0bd65e520fd77d#SHOU-353%20算法-知识库Agent%20路径前缀匹配 "飞书任务：#SHOU-353 路径前缀匹配")
## 二、推荐的 PR Description Markdown 模板（企业版）

> 👉 可直接作为 `.github/pull_request_template.md`

````markdown
## 1️⃣ 背景 / 需求说明（Why）

- 本次 PR 的业务背景 / 技术背景
- 问题是如何被发现的（线上问题 / 性能瓶颈 / 产品需求 / 技术债）
- 如果不做会有什么影响

---

## 2️⃣ 改动内容（What）

本次 PR 的主要改动如下：

- [x] 新增：XXX 功能 / 模块
- [x] 修复：XXX Bug（原因：XXX）
- [x] 优化：XXX 逻辑 / 性能 / 结构
- [ ] 预留：XXX（后续 PR 处理）

> 如有必要，可补充整体流程或架构变化说明。

---

## 3️⃣ 关键实现说明（How）

> 用于帮助 Reviewer 快速理解设计选择

- 核心实现思路：
  - 为什么采用该方案
  - 与其他方案的对比（如有）

- 关键代码点：
  - `xxx_service.py`: 处理核心业务逻辑
  - `xxx_handler.ts`: 请求入口与参数校验

---

## 4️⃣ 测试情况（Testing）

- [x] 单元测试
- [x] 集成测试
- [ ] 回归测试
- [ ] 未测试（请说明原因）

### 测试说明

```text
示例：
1. 启动本地服务
2. 调用 POST /api/v1/xxx
3. 验证返回结果与数据库状态
````

---

## 5️⃣ 影响范围 & 风险评估（Impact / Risk）

### 影响范围

-  仅影响当前模块
    
-  影响公共组件 / SDK
    
-  影响下游服务
    
-  影响线上数据 / 配置
    

### 风险点

- 潜在风险：
    
    - XXX 场景下可能存在边界问题
        
- 风险控制措施：
    
    - 增加校验 / 降级 / 回滚方案
        

---

## 6️⃣ 是否存在不兼容变更（Breaking Changes）

-  是（请详细说明）
    
-  否
    

如是，请说明：

- API / 行为变更点
    
- 迁移方式 / 兼容策略
    

---

## 7️⃣ 关联内容（References）

- 需求文档：
    
- 设计文档：
    
- Issue / 任务：
    
    - Closes #123
        
    - Related to #456
        

---

## 8️⃣ Reviewer 关注点（必填）

> 明确告诉 Reviewer **该重点看什么**

-  逻辑正确性
    
-  性能 / 并发
    
-  可维护性 / 可扩展性
    
-  命名 / 风格
    
-  其他：_____________
    

````

---

## 三、企业内部 PR 描述简化版（低成本）

> 当 PR 较小，允许使用以下最小集模板

```markdown
## 背景

简要说明为什么要改。

## 改动

- 做了什么改动

## 测试

- 如何验证改动有效

## 影响

- 是否影响其他模块 / 服务
````

---

## 四、常见问题（企业 Review 场景）

### ❌ 不推荐的写法

- “修复了一些问题”
    
- “优化代码结构”
    
- “如题”
    

> 这些描述 **对 Reviewer 没有任何帮助**。

### ✅ 推荐的写法

- “修复在并发场景下 token 重复刷新的问题，避免缓存击穿”
    
- “将同步 RPC 调用改为异步批量处理，QPS 提升约 30%”
    

---

## 五、落地建议（非常重要）

1. **PR 模板强制启用**（仓库级）
    
2. Code Review 时将「PR 描述质量」作为 Review 项
    
3. 新人 Onboarding 时明确 PR 规范
    
4. 不合格 PR：
    
    - 可以 Review
        
    - 但必须要求补充 Description
        

---

## 六、推荐目录结构

```text
.github/
 ├── pull_request_template.md
 └── ISSUE_TEMPLATE/
```

---

> 📌 结论：  
> **好的 PR 描述 = 半个设计文档 + 一次高质量沟通。**