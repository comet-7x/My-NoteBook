# AI国际谈判教学平台总体设计（V1）

> Version：V1.0  
> 适用范围：WTO《渔业补贴协定》、RFMO（IOTC）及后续国际组织谈判模拟  
> 更新时间：2026-07

---

# 一、项目目标

本平台旨在利用人工智能、大语言模型（LLM）、知识图谱与RAG（Retrieval-Augmented Generation）技术，构建一个面向国际谈判教学的智能模拟平台。

平台以**国际谈判议题（Issue）**为核心，以国际组织公开文件为数据来源，通过AI自动抽取议题、提案、国家立场及谈判结果，帮助教师快速创建谈判课程，帮助学生深入理解国际谈判全过程。

平台遵循真实国际组织（WTO、RFMO等）的谈判规则，模拟真实外交谈判流程，而不是简单的角色扮演。

---

# 二、设计原则

整个系统遵循以下设计思想：

> **以国际谈判议题（Issue）为核心，以国际组织原始文件（Evidence）为证据，以AI知识抽取构建结构化知识库，以教师创建谈判项目为组织方式，以学生围绕议题开展协商一致谈判为教学活动，以AI全过程辅助和复盘分析为支撑。**

因此：

- PDF不是知识库
- 文件不是课程
- 条款不是模拟对象
- **Issue（议题）才是整个系统的核心对象（Core Object）**

---

# 三、数据来源

## 3.1 RFMO（IOTC）

需要录入知识库的数据类型如下：

|所属分组|英文名称|数量|
|--------|--------|------|
|Meeting Reports|会议报告|356|
|Meeting Documents|执行摘要|301|
|Meeting Documents|会议文件|6049|
|Meeting Documents|会议信息|188|
|National Reports|国家报告|352|
|Information Papers|信息文件|1006|
|NGO Statements|非政府组织声明|161|
|FAO Documents|粮农组织文件|2|
|Project Report|项目报告|2|
|Scientific Data|数据集|89|
|Guidelines|指南|11|

> 根据课程需求，仅录入上述绿色部分文件。

---

## 3.2 WTO《渔业补贴协定》

目前已获取文档如下：

|机构|实际下载|
|------|------|
|G/FS 委员会|27|
|TN 谈判馆藏|310|
|WT/MIN 部长会|197|
|WT/L 法律文本|11|
|WT/LET 议定书|93|
|G/SCM 补贴通报|157|
|WT/GC 总理事会|45|
|JOB/RL|1|

共计：

- 文件总数：1117
- 已下载：841
- 预计录入知识库：848份

---

# 四、知识抽取流程

所有PDF统一经过AI抽取，形成结构化知识。

整个流程共分为五层。

---

## 第一层：Document Metadata（文件元信息）

自动抽取文件基础信息。

包括：

- Document ID
- Title
- Organization
- Meeting
- Agenda（所属议程）
- Date
- Country
- Article
- Language
- Source URL

输出示例：

```json
{
  "document_id":"TN/RL/W/274",
  "meeting":"MC12",
  "country":"China",
  "article":"Article 5",
  "date":"2021-11-25"
}
```

---

## 第二层：Issue Mining（议题抽取）

AI识别本文讨论了哪些谈判议题。

每个Issue包括：

- Issue Name
- Background
- Issue Description
- Issue Category
- Related Articles
- Priority

例如：

```
Issue

IUU认定主体

Article 3
```

一篇文件可以对应多个Issue。

---

## 第三层：Proposal Mining（提案抽取）

针对每个Issue继续抽取提案。

每个Proposal包括：

- Proposal ID
- Proposal Name
- Proposal Type
- 修改方式
  - 新增（Add）
  - 删除（Delete）
  - 修改（Modify）
- 修改内容
- 修改理由

例如：

```
Proposal A

所有RFMO均可认定

Proposal B

仅沿海国认定

Proposal C

建立联合委员会
```

---

## 第四层：Country Position Mining（国家立场抽取）

AI识别不同国家对于Proposal的态度。

包括：

- 国家
- 支持
- 反对
- 中立
- 保留意见
- Evidence
- Confidence

例如：

```
中国

支持 Proposal B

Evidence

Paragraph 12
```

---

## 第五层：Negotiation Outcome（谈判结果抽取）

抽取最终谈判结果。

包括：

- 是否采纳
- 是否形成Chair Text
- 是否进入最终协定
- 最终案文
- Consensus情况

例如：

```
Issue

Transparency

↓

Proposal B

↓

MC12 Adopted
```

---

# 五、知识库总体结构

知识库不是PDF集合，而是Issue知识库。

```
Knowledge Base

│

├── Issue Database

├── Proposal Database

├── Country Position Database

├── Country Profile Database

├── Evidence Database

└── Reference Database
```

其中：

## Issue Database

保存：

- Issue
- 背景
- 条款
- 生命周期

---

## Proposal Database

保存：

- Proposal
- 修改内容
- 修改理由

---

## Country Position Database

保存：

- 国家历史立场
- 支持方案
- 红线
- 历史变化

---

## Country Profile Database

保存：

- 国家画像
- 渔业情况
- GDP
- 渔业补贴
- 政治压力
- 国内利益

---

## Evidence Database

保存：

所有Issue对应的PDF证据。

例如：

```
Issue

↓

Evidence

TN/RL/W/274

Paragraph 18
```

---

# 六、系统数据模型

```
Organization

│

├── WTO

│   └── Fish Subsidies Agreement

│       ├── Meeting

│       │

│       ├── Agenda

│       │

│       ├── Issue

│       │

│       ├── Proposal

│       │

│       ├── Country Position

│       │

│       ├── Evidence

│       │

│       └── Outcome

│

└── RFMO

    └── IOTC
```

---

# 七、Issue生命周期

每个Issue不是静态存在，而是在多个会议中不断演化。

```
Issue

↓

首次提出

↓

国家提交Proposal

↓

Chair Draft

↓

多轮修订

↓

Consensus

↓

Final Agreement
```

系统记录Issue完整生命周期。

---

# 八、教师端业务流程

## Step 1

选择组织

```
WTO

RFMO
```

---

## Step 2

填写课程基础信息

包括：

- 课程名称
- 教师
- 时间
- 学时
- 模拟时长

---

## Step 3

选择协定

例如：

```
Fish Subsidies Agreement

IOTC Resolution
```

---

## Step 4

选择会议

例如：

```
MC12

MC13

Committee Meeting
```

---

## Step 5

选择Agenda

例如：

```
Agenda 1

Agenda 2

Agenda 3
```

---

## Step 6

选择Issue

例如：

```
Issue1

Issue2
```

支持：

- 单Issue
- 多Issue
- 完整会议

---

## Step 7

选择参与国家

例如：

- 中国
- 欧盟
- 美国
- 日本
- 印度
- 澳大利亚
- LDC

---

## Step 8

AI生成国家Brief

包括：

- 国家利益
- 谈判目标
- 红线
- 可交换利益
- 谈判建议
- Opening Statement

教师可修改。

---

## Step 9

生成谈判项目

平台自动生成：

- 学生资料
- 国家材料
- 条款
- 背景
- 提案
- 历史立场

---

# 九、学生端业务流程

整个模拟遵循真实国际谈判流程。

```
认识问题

↓

认识自己国家

↓

认识其他国家

↓

制定谈判策略

↓

正式谈判

↓

协商一致

↓

AI复盘

↓

教师点评
```

---

## 第一阶段

认识问题

学习：

- 为什么谈判
- 为什么制定协定
- 条款背景
- 历史争议

AI负责：

知识讲解。

---

## 第二阶段

认识自己的国家

学习：

- GDP
- 渔业
- 补贴
- 国内政治
- 国家利益

AI负责：

生成国家画像。

---

## 第三阶段

认识其他成员

学习：

- 国家立场
- 联盟关系
- 利益冲突

AI负责：

联盟分析。

---

## 第四阶段

制定谈判策略

生成：

- Core Objectives
- Red Lines
- Negotiable Items
- BATNA
- Talking Points

最终形成：

Negotiation Brief。

---

## 第五阶段

正式谈判

流程如下：

```
主席宣布会议开始

↓

秘书处介绍Agenda

↓

介绍Issue

↓

General Statement

↓

第一轮磋商

↓

提交Proposal

↓

秘书处整合

↓

第二轮磋商

↓

形成Chair Draft

↓

继续修改

↓

Consensus Check

↓

会议结束
```

AI担任秘书处。

---

## 第六阶段

AI复盘

生成：

- 联盟形成过程
- Proposal变化
- 国家妥协
- 最终Agreement
- 与真实历史比较
- 学习建议

教师进行点评。

---

# 十、AI角色设计

平台采用多智能体（Multi-Agent）架构。

包括：

## AI讲解员

负责：

知识讲解。

---

## AI研究助手

负责：

RAG检索。

---

## AI国家顾问

负责：

国家Brief生成。

---

## AI秘书处

负责：

会议记录。

维护实时谈判文本。

---

## AI主席助手

负责：

程序提醒。

判断是否符合WTO规则。

---

## AI复盘分析员

负责：

生成会议复盘。

---

# 十一、平台整体数据流

```
国际组织原始文件（PDF）

↓

OCR

↓

Metadata

↓

Issue Mining

↓

Proposal Mining

↓

Country Position Mining

↓

Outcome Mining

↓

Knowledge Base

↓

教师创建课程

↓

学生模拟谈判

↓

AI全过程辅助

↓

教师点评

↓

课程评价
```

---

# 十二、后续扩展方向

本架构具有良好的可扩展性，可逐步扩展至：

- WTO Fish 2 谈判
- WTO Fish 3 谈判
- IOTC 谈判模拟
- ICCAT 谈判模拟
- WCPFC 谈判模拟
- FAO 国际渔业治理
- 气候变化国际谈判（UNFCCC）
- 生物多样性谈判（CBD）
- 联合国海洋法谈判（UNCLOS）

未来所有国际组织均可复用同一套知识抽取与谈判模拟框架。