# 知识库设计（Knowledge Base Design）V2

> Version：V2.0 
> 上游依据：《基于WTO渔业补贴协定的国际谈判智能体设计思路V6》《AI国际谈判教学平台总体设计V2》 
> 下游文档：《AI知识抽取工作流设计（Extraction Pipeline）》 
> 更新时间：2026-07-22

> **文档范围说明**：本文档**仅覆盖知识域（Knowledge Domain）**——历史的、只读的、有出处的内容。 运行时域（Project / Session / Amendment / TextVersion 等课堂产出对象）见《总体设计V2》第七章，本文档不重复。

---

# 一、设计目标与范围

## 1.1 定位

知识库的目标是**让 AI 的回答有据可查，并为教学模拟提供人工校准的初始素材**。

本期定位为**检索增强（RAG）**，而非条款级结构化知识库重建。这一定位决定了后续所有取舍。

||做|不做|
|---|---|---|
|文件处理|去重、类型识别、段落切分、向量化|—|
|元信息|文件编号、会议、日期、国家、语言、条款关联|—|
|证据|**段落级**定位与引用|句级、字符级精确对齐|
|议题|抽取候选 + 人工策展的教学议题|议题的自动生命周期追踪|
|提案|作为策展素材保留|全量结构化建模|
|立场|人工校准的立场卡 + 演变时间线|全语料自动立场抽取并直接发布|
|结果|教学对照用的历史结果|条款级结果与提案的自动映射|
|校验|—|修正案自动合规校验（二期评估）|

## 1.2 三条硬性约束

这三条贯穿全部对象设计，任何字段设计不得违反：

1. **可溯源（L1）**：任何涉及具体事实的内容，必须能定位到原始文件的具体段落。
2. **人工校准（L2）**：国家立场、教学议题、国家画像必须经人工审核后方可发布；未发布内容不得进入学生可见层。
3. **可见性分级**：历史谈判**结果**默认对学生不可见，仅在复盘阶段解锁。

---

# 二、存储架构

## 2.1 选型：关系库 + 向量库

```
┌──────────────────────────────────────────────────────┐
│                    应用层 / AI Agent                  │
└────────┬─────────────────────────────┬───────────────┘
         │ 结构化查询                   │ 语义检索
┌────────▼──────────────┐   ┌──────────▼───────────────┐
│      关系库            │   │        向量库             │
│ ─────────────────────  │   │ ────────────────────────  │
│ Organization           │   │ chunk_vector              │
│ Instrument / Meeting   │◄──┤ + metadata（用于过滤）     │
│ Document               │   │   document_id             │
│ Chunk（坐标与文本）     │   │   org / meeting / year    │
│ Evidence               │   │   doc_type / language     │
│ Issue / TeachingIssue  │   │   issue_tags              │
│ IssueOption            │   │   chunk_id ──────────────┐│
│ Proposal               │   └───────────────────────── ││
│ CountryPosition        │                              ││
│ PositionEvolution      │◄─────────────────────────────┘│
│ CountryProfile         │      chunk_id 回查坐标         │
│ ProfileMetric          │                               │
│ Outcome                │                               │
│ ReviewLog              │                               │
└────────────────────────┘                               │
         ▲                                               │
         │ 原始文件（对象存储：PDF / 音频 / 附件）          │
         └───────────────────────────────────────────────┘
```

## 2.2 不引入图数据库的理由

V1 草稿提及知识图谱。在本期 RAG 定位下，**建议不引入**，理由：

- 本期最深的关系查询是「教学议题 → 参与国 → 立场卡 → 证据 → 原始文件」，**深度 ≤ 4 跳且路径固定**，关系库外键 + 索引完全够用；
- 图数据库的收益在于**路径不固定的多跳探索**（如联盟网络传导、立场影响链），这属于二期「动态联盟分析」的需求；
- 引入第三种存储会显著增加运维与数据同步成本，与「一期最小可用」的范围设定冲突。

**二期评估触发条件**：若要做动态联盟网络分析或跨议题立场影响传导分析，届时再评估引入图存储或在关系库上做图计算。

## 2.3 检索策略：混合检索

纯向量检索在本场景会失效，因为国际组织文件大量依赖**精确标识符**：文件编号（TN/RL/W/274）、条款号（Article 3）、决议号（Resolution 13/06）、会议届次（MC12）。这些是精确匹配需求，语义相似度会给出错误结果。

因此采用三路混合：

|通道|用途|权重场景|
|---|---|---|
|**元数据过滤**|先按组织、会议、年份、文档类型、议题标签缩小范围|始终启用，前置过滤|
|**关键词/BM25**|文件编号、条款号、决议号、国家名的精确命中|查询含标识符时权重高|
|**向量检索**|概念性、解释性问题（「为什么补贴导致过度捕捞」）|查询为自然语言描述时权重高|

结果融合后回查关系库获取段落坐标，生成 Evidence。

---

# 三、对象总览

## 3.1 关系图

```
Organization ──┬── Instrument ──── (条款/决议条文)
               │
               ├── Meeting ──────── Document ──── Chunk
               │                       │            │
               │                       └────────────┴──► Evidence
               │                                            ▲
               ├── Issue（抽取候选）─────────┐                │
               │         │ 聚合/策展          │                │
               │         ▼                   │                │
               ├── TeachingIssue ── IssueOption               │
               │         │                                    │
               │         ├──────── CountryPosition ───────────┤
               │         │              │                     │
               │         │              └─ PositionEvolution ─┤
               │         │                                    │
               │         └──────── Outcome ───────────────────┤
               │                                              │
               ├── Proposal ─────────────────────────────────┤
               │                                              │
               └── Country ──── CountryProfile ── ProfileMetric

           所有 L2 对象 ──── ReviewLog（审核留痕）
```

## 3.2 对象清单

|#|对象|中文|生产方式|可信度|一期数量级|
|---|---|---|---|---|---|
|1|`Organization`|组织|人工配置|—|2|
|2|`Instrument`|法律文书|人工 + 抽取|L1|10–30|
|3|`Meeting`|会议|抽取|L1|100–300|
|4|`Document`|原始文件|抽取|L1|待定（见附录）|
|5|`Chunk`|文件分块|自动|L1|10 万级|
|6|`Evidence`|证据引用|自动生成|L1|动态|
|7|`Issue`|议题候选|抽取|—|1000+|
|8|`TeachingIssue`|**教学议题**|**策展**|**L2**|**WTO 6（已确认）**|
|9|`IssueOption`|**方案区间**|**策展**|**L2**|**18（6×3）**|
|10|`Proposal`|历史提案|抽取|L1|300–800|
|11|`Country`|国家/关税区|人工配置|—|50+|
|12|`CountryPosition`|**国家立场卡**|**策展**|**L2**|**54（WTO：6×9）**|
|13|`PositionEvolution`|**立场演变节点**|**策展**|**L2**|**约 190（54×3~4）**|
|14|`CountryProfile`|**国家画像**|**策展**|**L2**|**9**|
|15|`ProfileMetric`|画像指标|数据导入|L1|约 180（9×20）|
|16|`Outcome`|谈判结果|策展|L2|5–8 / 场景|
|17|`ReviewLog`|审核留痕|自动|—|动态|

> **加粗行是教学效果的真正来源**，WTO 场景合计约 **270 条**人工校准记录，是项目的关键路径。

---

# 四、对象详细设计

> 通用字段（所有对象均含，下表不再重复）： `id`（主键）、`created_at`、`updated_at`、`created_by`、`kb_version`（所属知识库发布版本）

## 4.1 Organization 组织

|字段|类型|必填|说明|
|---|---|---|---|
|`code`|string|✅|`WTO` / `IOTC`|
|`name_zh` / `name_en`|string|✅|名称|
|`member_term`|string|✅|成员称谓：WTO=`Member`；IOTC=`CPC`|
|`member_types`|json|✅|IOTC：`["成员","合作非缔约方"]`；WTO：`["成员"]`|
|`decision_rule`|enum|✅|`consensus` / `consensus_then_two_thirds`|
|`objection_window_days`|int||IOTC=120；WTO 为空|
|`objection_threshold`|string||IOTC=`1/3`；WTO 为空|
|`instrument_types`|json|✅|WTO：协定条款；IOTC：`Resolution` / `Recommendation`|
|`deadline_mechanism`|text||日落条款等压力装置说明|

> 该对象是多组织复用的配置入口，新增组织时无需改代码。

## 4.2 Instrument 法律文书

|字段|类型|必填|说明|
|---|---|---|---|
|`org_id`|FK|✅|所属组织|
|`code`|string|✅|如 `AFS`（渔业补贴协定）、`RES-24-01`|
|`title_zh` / `title_en`|string|✅||
|`instrument_type`|enum|✅|`agreement` / `resolution` / `recommendation`|
|`status`|enum|✅|`in_force` / `negotiating` / `superseded`|
|`entry_into_force`|date||如 2025-09-15|
|`provisions`|json||条款结构树（条 / 款 / 项），供条款级引用定位|
|`sunset_date`|date||日落节点，如 2029-09|

## 4.3 Meeting 会议

|字段|类型|必填|说明|
|---|---|---|---|
|`org_id`|FK|✅||
|`code`|string|✅|`MC12` / `IOTC-S28`|
|`meeting_type`|enum|✅|`ministerial` / `negotiating_group` / `committee` / `commission_session` / `scientific_committee`|
|`title`|string|✅||
|`start_date` / `end_date`|date|✅||
|`location`|string|||
|`agenda`|json||议程项列表|

## 4.4 Document 原始文件

|字段|类型|必填|说明|
|---|---|---|---|
|`doc_symbol`|string|✅|**文件编号**，如 `TN/RL/W/274`。**去重主键**|
|`language`|enum|✅|`en` / `fr` / `es` / `zh`|
|`is_primary_language`|bool|✅|同一 `doc_symbol` 下仅一份为 true（默认 en），检索默认只用 primary|
|`org_id` / `meeting_id`|FK|✅ /||
|`doc_type`|enum|✅|见 8.1，决定抽取路由|
|`title`|string|✅||
|`date`|date|✅||
|`author_country`|FK||提案国 / 报告国|
|`related_provisions`|json||关联条款，如 `["Article 3","Article 5"]`|
|`source_url`|string|✅||
|`file_path`|string|✅|对象存储路径|
|`page_count`|int|||
|`has_text_layer`|bool|✅|false 则需 OCR|
|`extraction_status`|enum|✅|`pending` / `extracted` / `failed` / `skipped`|

> **多语言去重规则**：以 `doc_symbol` 为唯一键，同编号的多语言版本合并为一组，`is_primary_language=true` 的进入检索库，其余仅保留文件路径供查阅。
> 
> **各场景适用情况**：
> 
> - **WTO**：已确认仅下载保存英文版，该机制在 WTO 场景下空转，841 份即唯一编号数；
> - **IOTC**：官方语言为**英语和法语**，法文版文件真实存在，该机制**必须保留并生效**，`is_primary_language` 默认取英文版。
> 
> 因此该机制不因 WTO 场景不需要而移除。

## 4.5 Chunk 文件分块

|字段|类型|必填|说明|
|---|---|---|---|
|`document_id`|FK|✅||
|`chunk_index`|int|✅|文件内顺序|
|`text`|text|✅|分块文本|
|`page_no`|int|✅|所在页|
|`para_label`|string||**原文段落编号**，如 `12`、`3.2`（若原文有编号）|
|`para_index`|int|✅|段落序号（无原文编号时用推断序号）|
|`para_confidence`|float|✅|段落识别置信度，见 5.1|
|`char_start` / `char_end`|int|✅|在文件纯文本中的字符区间|
|`section_path`|string||所属章节路径，如 `III / B / 2`|
|`issue_tags`|json||议题标签（抽取产出），用于元数据过滤|
|`vector_id`|string|✅|向量库中的对应 ID|

**分块策略**：以**原文段落为基本单元**，不做固定长度切分。超长段落（>1000 字）内部再切但保留同一 `para_label`；过短段落（<50 字，如标题行）与相邻段落合并但记录原始边界。

## 4.6 Evidence 证据引用

|字段|类型|必填|说明|
|---|---|---|---|
|`document_id`|FK|✅||
|`chunk_id`|FK|✅||
|`doc_symbol`|string|✅|冗余存储，便于直接展示|
|`page_no`|int|✅||
|`para_label`|string||展示用，如 "Paragraph 18"|
|`char_start` / `char_end`|int||段内精确区间（可选）|
|`quote_excerpt`|text||**摘录片段，长度上限 100 字**，仅用于界面高亮定位|
|`linked_object_type`|enum|✅|被支撑的对象类型：`CountryPosition` / `TeachingIssue` / `Outcome` / `Proposal` / `answer`|
|`linked_object_id`|string|✅||
|`relevance`|float||相关度|

**展示格式统一为**：`TN/RL/W/274, Paragraph 18`（对应 V1 草稿的举例）。

> **注意**：`quote_excerpt` 仅供界面定位高亮，**不得**在 AI 生成的回答正文中大段复现原文；回答应转述并附证据链接。

## 4.7 Issue 议题候选（抽取产出）

|字段|类型|必填|说明|
|---|---|---|---|
|`org_id`|FK|✅||
|`name`|string|✅||
|`description`|text|||
|`category`|string|||
|`related_provisions`|json|||
|`source_documents`|json|✅|来源文件列表|
|`merged_into`|FK||策展后归入的 `TeachingIssue`|
|`curation_status`|enum|✅|`raw` / `merged` / `discarded`|

> 该对象**粒度细、数量大、噪声高，不直接面向教师和学生**，仅作为策展工作台的输入。教师端选择议题时看到的是 `TeachingIssue`。

## 4.8 TeachingIssue 教学议题 ★

对应 V6「Fish 1 阶段谈判的核心议题清单」中的一行。

|字段|类型|必填|说明|
|---|---|---|---|
|`org_id` / `instrument_id`|FK|✅||
|`code`|string|✅|如 `WTO-FISH1-01`|
|`name_zh`|string|✅|如「IUU 捕捞认定与处罚规则」|
|`background`|text|✅|议题背景，`student_pre`|
|`core_dispute`|text|✅|争议焦点是什么|
|`related_provisions`|json|✅|关联条款|
|`base_text`|text|✅|**基线案文**，模拟时的 `BaseText v0` 来源|
|`difficulty`|enum||`low` / `medium` / `high`，供教师选课参考|
|`visibility`|enum|✅|默认 `student_pre`|
|`review_status`|enum|✅|见 5.4|
|`evidence_ids`|json|✅|支撑证据|

## 4.9 IssueOption 方案区间 ★

对应 V6 议题表中的「谈判可选方案区间 A / B / C」。**独立成表**，因为学生的修正案需要与之关联，复盘时也需按方案统计。

|字段|类型|必填|说明|
|---|---|---|---|
|`teaching_issue_id`|FK|✅||
|`option_code`|string|✅|`A` / `B` / `C`|
|`summary`|string|✅|如「仅 RFMO 认定有效，沿岸国无权判定」|
|`description`|text||展开说明|
|`typical_supporters`|json||倾向支持的成员类别|
|`typical_opponents`|json||倾向反对的成员类别|
|`is_historical_outcome`|bool|✅|是否为历史上实际落地的方案|
|`outcome_note`|text||落地细节，如「最终折中达成 2 年缓冲条款」|
|`visibility`|enum|✅|方案本身 `student_pre`；`is_historical_outcome` 与 `outcome_note` **强制 `student_post`**|

> **关键规则**：`is_historical_outcome` 与 `outcome_note` 属于字段级可见性控制。若学生在准备阶段即可看到哪个方案是历史答案，博弈空间当场归零。见 5.2。

## 4.10 Proposal 历史提案

|字段|类型|必填|说明|
|---|---|---|---|
|`document_id`|FK|✅|提案所在文件|
|`sponsor_country`|FK|✅|提案国|
|`co_sponsors`|json||联署国|
|`teaching_issue_id`|FK||关联教学议题（策展时挂接）|
|`operation`|enum||`add` / `delete` / `modify`|
|`target_provision`|string||目标条款|
|`content_summary`|text|✅|提案内容概述（转述，非原文照抄）|
|`rationale`|text||提案理由|
|`evidence_ids`|json|✅||

> 本期 `Proposal` 主要作为**策展素材与检索对象**，不做全量精细结构化。

## 4.11 Country 国家 / 关税区

|字段|类型|必填|说明|
|---|---|---|---|
|`code`|string|✅|ISO 或自定义（如 `EU`、`TPKM`）|
|`name_zh` / `name_en`|string|✅||
|`member_category`|enum|✅|V6 四类：`major_fishing` / `developed_regulatory` / `developing` / `ldc`|
|`org_memberships`|json|✅|在各组织中的身份（WTO 成员 / IOTC 成员 / 合作非缔约方）|
|`is_playable`|bool|✅|是否可作为模拟中的代表团|

### 4.11.1 一期参与国清单（已确认）

|国家 / 关税区|V6 成员分类|WTO 成员|IOTC 缔约方|备注|
|---|---|---|---|---|
|中国|渔业规模较大成员|✅|✅||
|日本|渔业规模较大成员|✅|✅||
|韩国|渔业规模较大成员|✅|✅||
|欧盟|渔业规模较大成员|✅|✅||
|俄罗斯|（V6 未分类，建议归入渔业规模较大成员）|✅|❌|**仅 WTO 场景可用**；策展前需做证据可得性检查|
|美国|发达成员（环保 / 监管导向）|✅|❌|**仅 WTO 场景可用**|
|澳大利亚|发达成员（环保 / 监管导向）|✅|✅||
|印度|发展中成员|✅|✅||
|印度尼西亚|发展中成员|✅|✅||

**场景绑定规则**：参与国清单**按场景绑定**，不是全局配置。IOTC 场景需另配清单（美、俄非 IOTC 缔约方，需以其他 IOTC 成员替代）。该规则由 `Country.org_memberships` 与 `CountryPosition.scenario_code` 支撑，新增国家仅需入库，无需改动 schema。

> **教学覆盖度提示**：当前 9 国覆盖 V6 四类成员中的前三类，**第四类（LDC）为空**，且缺少小岛屿 / 小型渔业国。这会使部分议题的方案缺少天然提案人——尤其议题⑤「透明度通报」的方案 B（低产量小国每 4 年上报、简化材料）在 9 国中无任何国家有动机提出。
> 
> 建议增补 **孟加拉国**（LDC，同为 WTO 与 IOTC 成员）与 **马尔代夫或斯里兰卡**（小岛屿 / 小型渔业国，同为 WTO 与 IOTC 成员），两者在两个场景下均可复用。增补成本为每国 6 张立场卡。是否增补由教师决定，见附录 A #11。

## 4.12 CountryProfile 国家画像 ★

|字段|类型|必填|说明|
|---|---|---|---|
|`country_id`|FK|✅||
|`scenario_code`|string|✅|所属场景（WTO-FISH1 / IOTC-xxx）|
|`narrative_basic`|text|✅|基本情况叙述|
|`narrative_fishery`|text|✅|渔业情况|
|`narrative_subsidy`|text|✅|补贴情况|
|`domestic_pressure`|json|✅|国内政治压力：渔民协会 / 环保组织 / 财政部门 / 地方政府，各含诉求与强度|
|`core_interest`|text|✅|核心国家利益概括|
|`visibility`|enum|✅|`student_pre`|
|`review_status`|enum|✅||
|`evidence_ids`|json|✅||

## 4.13 ProfileMetric 画像指标

**所有数值型事实独立成表，强制带来源与年份**，落实 V6「不能由模型自由编造」。

|字段|类型|必填|说明|
|---|---|---|---|
|`profile_id`|FK|✅||
|`metric_code`|string|✅|`population` / `gdp` / `gdp_per_capita` / `coastline_km` / `fishery_employment` / `catch_volume` / `seafood_export` / `distant_water_fleet` / `subsidy_total` / `subsidy_fuel` …|
|`value`|decimal|✅||
|`unit`|string|✅||
|`year`|int|✅|数据年份|
|`source`|enum|✅|`FAO_SOFIA` / `OECD_FSE` / `WTO_NOTIFICATION` / `WORLD_BANK` / `IMF` / `OTHER`|
|`source_note`|string||口径说明|
|`imported_at`|datetime|✅||

> **禁止规则**：`ProfileMetric` 的 `value` **不得**由 LLM 生成，只能由数据导入任务写入。抽取管线若从 PDF 中读到数值，须走人工确认后方可入表。

## 4.14 Outcome 谈判结果

|字段|类型|必填|说明|
|---|---|---|---|
|`teaching_issue_id`|FK|✅||
|`meeting_id`|FK||达成结果的会议|
|`adopted_option`|FK||最终采纳的 `IssueOption`|
|`final_text`|text||最终案文（转述或引用，注意版权与篇幅）|
|`consensus_note`|text||达成方式说明；IOTC 场景记录反对方|
|`objecting_members`|json||IOTC：提出反对的 CPC 列表|
|`visibility`|enum|✅|**强制 `student_post`**|
|`review_status`|enum|✅||
|`evidence_ids`|json|✅||

## 4.15 ReviewLog 审核留痕

|字段|类型|必填|说明|
|---|---|---|---|
|`object_type` / `object_id`|string|✅|被审核对象|
|`from_status` / `to_status`|enum|✅|状态流转|
|`reviewer`|string|✅|审核人|
|`comment`|text||审核意见|
|`diff`|json||本次修改内容|
|`reviewed_at`|datetime|✅||

---

# 五、关键机制设计

## 5.1 段落级证据定位

### 为什么是段落级

- WTO 与 IOTC 文件本身按**编号段落**组织（Paragraph 12、Paragraph 18），引用惯例即是段落级，与真实实务对齐；
- **页级**引用无法定位到具体主张，一页可能包含多国发言，学生无法验证；
- **句级**在 PDF 转文本后边界不稳定（连字符换行、脚注插入、表格串行），成本高且不可靠。

### 定位方法与降级

|情形|策略|`para_confidence`|
|---|---|---|
|原文有显式段落编号|直接取用，写入 `para_label`|1.0|
|有编号但 OCR 识别不稳|编号 + 位置双重校验|0.7–0.9|
|无编号（如会议纪要连续叙述）|按空行 / 缩进 / 首行标识推断，`para_label` 留空，仅用 `para_index`|0.5–0.7|
|表格型文件（如补贴通报）|定位到「页 + 表 + 行」，`para_label` 记为 `Table N Row M`|0.6|
|扫描件 OCR|先 OCR 再走上述流程，置信度整体下调|≤0.6|

**规则**：`para_confidence < 0.6` 的 Chunk 仍可参与检索，但其生成的 Evidence 在界面上标注「定位为近似位置」。

## 5.2 可见性分级实现

### 默认可见性

|对象|默认 visibility|说明|
|---|---|---|
|`Document` / `Chunk`|`student_pre`|原始文件本身公开|
|`TeachingIssue`|`student_pre`|议题背景需学生预习|
|`IssueOption`（方案本身）|`student_pre`|学生需知道有哪些可选方案|
|`IssueOption.is_historical_outcome`|**`student_post`**|**字段级控制**|
|`IssueOption.outcome_note`|**`student_post`**|**字段级控制**|
|`CountryPosition`（主卡）|`student_pre`|各国公开立场|
|`PositionEvolution`|逐节点设定|默认 `student_pre`；涉及内部妥协底线的节点设 `teacher_only`|
|`CountryProfile` / `ProfileMetric`|`student_pre`||
|`Outcome`|**`student_post`**|强制|
|`Proposal`|`student_pre`|历史提案本身公开|

### 实现要点

- 可见性以**对象级为主、字段级为辅**。字段级控制仅用于 `IssueOption` 与 `PositionEvolution`，通过在应用层做字段白名单实现，避免 schema 复杂化；
- **快照时固化**：课程快照按 `student_pre` 过滤生成学生资料包，`student_post` 内容单独打包但加锁；
- **解锁动作**：教师在复盘阶段手动解锁，解锁事件记入 `ReviewLog`；
- **检索层同样受控**：学生提问时，RAG 的召回结果必须按 `visibility` 过滤，否则会从原始文件中泄露结果。这一点容易遗漏——**原始文件里也写着历史结果**。见 9.2 风险项。

## 5.3 可信度分级落库

不新增字段，通过**对象类型 + 状态**推导：

|内容|可信度|判定规则|
|---|---|---|
|RAG 回答|L1|必须附带 ≥1 条 Evidence，否则拒绝输出事实性断言|
|`TeachingIssue` / `CountryPosition` / `CountryProfile` / `Outcome`|L2|`review_status = published`|
|联盟建议、策略建议、谈判简报草案|L3|运行时生成，不入知识库，界面标注「AI 建议，仅供参考」|

## 5.4 审核状态机

```
     ┌──────────┐   提交审核   ┌──────────────┐   通过   ┌─────────────────┐   发布   ┌───────────┐
     │ ai_draft │ ──────────► │ under_review │ ───────► │ human_verified  │ ───────► │ published │
     └──────────┘             └──────┬───────┘          └─────────────────┘          └─────┬─────┘
           ▲                         │ 退回                                                │ 下架
           └─────────────────────────┘                                                     ▼
                                                                                    ┌────────────┐
                                                                                    │ deprecated │
                                                                                    └────────────┘
```

- 每次流转写 `ReviewLog`，记录审核人、时间、修改差异；
- **仅 `published` 状态的对象可进入课程快照**；
- 已被快照引用的对象若后续变更，不影响进行中的课程（见 5.5）。

## 5.5 版本与快照

### 知识库发布版本

- 知识库整体有 `kb_version`（如 `2026.07.1`），每次批量发布递增；
- 所有对象记录其所属 `kb_version`。

### 课程快照的两种实现

|内容|快照方式|理由|
|---|---|---|
|教学议题、方案区间、立场卡、演变节点、国家画像、指标、基线案文|**物化复制**到 Project|量小（约 400 条），复制成本低；且必须保证课程期间绝对稳定|
|Document / Chunk / Evidence|**版本引用**（记录 `kb_version`）|量大（10 万级），复制不现实；原始文件本身不会变更|

快照记录：`snapshot_at`、`kb_version`、`snapshot_manifest`（所含对象 ID 清单），供复盘时追溯「学生当时看到的是哪一版」。

---

# 六、国家立场卡与演变时间线（重点）

V6 要求「展示 2017 年至今的关键观点」，且明确**必须人工校准，不可由 AI 自由生成**。本节采用「主卡 + 演变子表」结构。

## 6.1 CountryPosition 立场卡（主表）

一张卡 = **一个国家在一个教学议题上的当前立场**。

|字段|类型|必填|说明|
|---|---|---|---|
|`country_id`|FK|✅||
|`teaching_issue_id`|FK|✅||
|`scenario_code`|string|✅||
|`stance`|enum|✅|`support` / `oppose` / `neutral` / `conditional` / `silent`|
|`preferred_option`|FK||倾向的 `IssueOption`（A/B/C）|
|`acceptable_options`|json||可接受的方案集合|
|`rationale_economic`|text|✅|**经济**层面的原因|
|`rationale_political`|text|✅|**政治**层面的原因|
|`rationale_social`|text||**社会**层面的原因|
|`red_line_tendency`|text||历史上表现出的不可让步点|
|`tradeable`|text||历史上表现出的可交换项|
|`stance_stability`|enum|✅|`firm`（长期坚持）/ `shifting`（有变化）/ `unclear`|
|`visibility`|enum|✅|默认 `student_pre`|
|`review_status`|enum|✅||
|`evidence_ids`|json|✅|**至少 1 条**，否则不得发布|

> **`rationale_economic / political / social` 三个字段是本项目的教学核心。** V6 反复强调「重点不是别人说过什么，而是为什么这样说」，教学目标第 2 条亦是「能分析立场形成的经济、政治和社会原因」。因此三层原因**拆成独立字段而非合并为一段叙述**，以强制策展时逐层填写、并支持学生端分层展示。

## 6.2 PositionEvolution 演变节点（子表）

一条记录 = **一个时间点上的立场表态**。

|字段|类型|必填|说明|
|---|---|---|---|
|`position_id`|FK|✅|所属立场卡|
|`event_date`|date|✅||
|`meeting_id`|FK||发生场合|
|`event_label`|string|✅|如「MC12 部长会」「2019 年谈判组会议」|
|`stance_at_time`|enum|✅|当时的立场|
|`change_type`|enum|✅|见下表|
|`statement_summary`|text|✅|当时观点概述（**转述，不照抄原文**）|
|`driver`|text||变化动因（国内选举、资源评估恶化、外部压力…）|
|`visibility`|enum|✅|默认 `student_pre`；涉及底线试探的节点可设 `teacher_only`|
|`evidence_ids`|json|✅||

### `change_type` 枚举

|值|含义|教学价值|
|---|---|---|
|`initial`|初次表态|建立基准|
|`reinforce`|强化原立场|说明红线所在|
|`soften`|软化|**识别可交换空间**|
|`shift`|转向|**最具教学价值**：为什么改变|
|`silent`|转为沉默/回避|战术性回避的信号|
|`withdraw`|撤回主张|妥协达成的标志|

## 6.3 教学用途映射

|学生端场景|使用字段|
|---|---|
|阶段三「认识其他成员」——理解为什么这样说|`rationale_*` 三字段 + `stance_stability`|
|阶段三——识别可争取对象|`stance_stability = shifting` + `change_type = soften` 的节点|
|阶段四「制定策略」——找可交换项|`tradeable` + `acceptable_options`|
|阶段六「复盘」——与历史立场比较|完整演变时间线 + 学生实际表现对照|

> **复盘对照示例（V6 原例）**：历史上印度始终坚持 S&DT（`stance_stability = firm`，多个 `reinforce` 节点），若本次学生代表印度最终放弃，系统在复盘中提示「本次谈判偏离历史立场」。

---

# 七、教学议题卡的策展要求

## 7.1 一张完整议题卡的构成

以 V6 议题①为例，说明各对象如何组合：

```
TeachingIssue：IUU 捕捞认定与处罚规则
├─ background        为什么需要认定规则、认定权归属为何有争议
├─ core_dispute      认定主体是谁——RFMO / 沿岸国 / 联合机制
├─ related_provisions Article 3
├─ base_text         该议题对应的基线案文（模拟起点）
├─ IssueOption A     简化认定流程、快速停发补贴
│                    typical_supporters: 太平洋岛国、环保导向成员
├─ IssueOption B     增设多层信息交换、听证缓冲期
│                    typical_supporters: 远洋渔业大国、发展中成员
├─ IssueOption C     仅 RFMO 认定有效，沿岸国无权判定
│                    typical_supporters: 部分发达成员
└─ Outcome           [student_post] 历史落地方案与细节
```

## 7.2 策展工作台的最小要求

策展是知识库的**关键路径**，工具需支持：

1. 左侧展示抽取出的 `Issue` 候选与相关 `Chunk`，右侧编辑 `TeachingIssue`；
2. 一键从 Chunk 生成 Evidence 并挂接到当前字段；
3. **未挂 Evidence 的字段不可提交审核**（强制约束）；
4. 立场卡支持按「议题 × 国家」矩阵批量填写与进度追踪（约 90 格）；
5. 审核视图：并排显示 AI 草稿与人工修改，提交后写 `ReviewLog`。

> 详细的策展流程与 AI 辅助起草设计见《AI知识抽取工作流设计》第三部分。

---

# 八、数据字典（枚举定义）

## 8.1 `doc_type` 文档类型（决定抽取路由）

|值|中文|代表|主要产出|
|---|---|---|---|
|`proposal`|提案型|TN/RL/W/xxx、IOTC Prop|提案内容、提案国、理由|
|`meeting_report`|会议纪要/报告型|委员会纪要、IOTC Meeting Report|**各国发言与立场（立场主来源）**|
|`decision`|决定型|WT/MIN、WT/L、Resolution|最终案文、通过情况|
|`notification`|通报型|G/SCM 补贴通报|补贴金额与类型（表格）|
|`national_report`|国家报告|IOTC National Report|履约情况、本国数据|
|`scientific`|科学型|执行摘要、Scientific Data|资源状态、评估结论|
|`ngo_statement`|NGO 声明|NGO Statements|外部主张（非成员立场）|
|`guideline`|指南|Guidelines|程序与操作规范|
|`legal_text`|法律文本|协定正文、议定书|条款结构|
|`other`|其他|||

## 8.2 其他枚举

|枚举|取值|
|---|---|
|`visibility`|`student_pre` / `student_post` / `teacher_only`|
|`review_status`|`ai_draft` / `under_review` / `human_verified` / `published` / `deprecated`|
|`stance`|`support` / `oppose` / `neutral` / `conditional` / `silent`|
|`stance_stability`|`firm` / `shifting` / `unclear`|
|`change_type`|`initial` / `reinforce` / `soften` / `shift` / `silent` / `withdraw`|
|`operation`|`add` / `delete` / `modify`|
|`member_category`|`major_fishing` / `developed_regulatory` / `developing` / `ldc`|
|`decision_rule`|`consensus` / `consensus_then_two_thirds`|
|`extraction_status`|`pending` / `extracted` / `failed` / `skipped`|

---

# 九、数据质量规则

## 9.1 强制校验（不满足则不可发布）

|#|规则|适用对象|
|---|---|---|
|1|必须挂接 ≥1 条 Evidence|`CountryPosition`、`PositionEvolution`、`TeachingIssue`、`Outcome`|
|2|`review_status` 必须为 `published` 方可进入快照|全部 L2 对象|
|3|`rationale_economic` 与 `rationale_political` 不得为空|`CountryPosition`|
|4|`value` 必须有 `source` 与 `year`|`ProfileMetric`|
|5|`Outcome.visibility` 必须为 `student_post`|`Outcome`|
|6|同一 `doc_symbol` 下 `is_primary_language=true` 有且仅有一条|`Document`|
|7|每个 `TeachingIssue` 至少有 2 个 `IssueOption`|`TeachingIssue`|
|8|参与国清单中每国对每个议题都有立场卡（矩阵完整性）|快照前校验|
|9|每个 `IssueOption` 至少有一个参与国的 `preferred_option` 或 `acceptable_options` 指向它|快照前校验|

## 9.2 风险项与对策

|风险|说明|对策|
|---|---|---|
|**原始文件泄露历史结果**|学生用 RAG 检索原始文件，可直接读到最终案文与通过情况|检索层按 `visibility` 过滤；对 `doc_type = decision` 的文件在准备阶段整体设为 `student_post`|
|**AI 编造国家立场**|立场类问题最易产生幻觉|立场类提问强制走 L2 资产而非自由生成；无对应立场卡时回答「暂无校准数据」|
|**数值幻觉**|GDP、补贴金额等被模型编造|`ProfileMetric` 禁止 LLM 写入；数值类回答只能引用该表|
|**多语言重复污染检索**|同一文件多个语言版本重复召回（IOTC 英/法双语；WTO 仅英文，不受影响）|`is_primary_language` 过滤|
|**段落定位错误**|OCR 或无编号文件定位偏差|`para_confidence` 标注；低置信度标「近似位置」|
|**策展工时超预算**|约 270 条 L2 记录是关键路径|矩阵进度追踪；优先完成参与国 × 议题的核心格子，非参与国延后|

---

# 十、初始化规模估算（一期）

|对象|数量级|生产方式|工时敏感度|
|---|---|---|---|
|`Document`|WTO 约 841 + IOTC 筛选后待定|自动|低|
|`Chunk`|10 万级|自动|低|
|`Issue`（候选）|1000+|自动|低|
|`TeachingIssue`|**WTO 6（已确认）** + IOTC 待定|**策展**|中|
|`IssueOption`|WTO 18（6 议题 × 3 方案）+ IOTC 待定|**策展**|中|
|`CountryPosition`|**WTO：6 × 9 = 54 张**（已确认参与国）|**策展**|**高**|
|`PositionEvolution`|每卡 3–4 节点 ≈ **190**|**策展**|**高**|
|`CountryProfile`|**9**|**策展**|中|
|`ProfileMetric`|每国约 20 项 ≈ **180**|数据导入 + 校对|中|
|`Outcome`|8–10|**策展**|低|

> **关键路径为 `CountryPosition` + `PositionEvolution`，WTO 场景合计约 244 条。** 排期应以此为基准，而非以 PDF 抽取量为基准。
> 
> 若后续增补参与国，每增 1 国 = +6 张立场卡 +约 21 个演变节点 +1 份画像 +约 20 条指标，可增量排期。

## 10.1 语料覆盖度体检（策展前置步骤）

立场卡为 L2 资产，**每张必须挂接至少一条 Evidence**（质量规则 9.1 #1）。若某参与国在 841 份 WTO 文件中的可引用材料过少，其 6 张立场卡将无法达到发布标准。

因此在策展开工前，须对 9 个参与国各执行一次覆盖度统计：

|统计项|口径|用途|
|---|---|---|
|作为 `author_country` 的文件数|该国提交的提案、报告数量|反映主动表态强度|
|在 `doc_type = meeting_report` 文件中被记录发言的次数|立场证据的主要来源|反映可引用材料总量|
|按 6 个教学议题分布的覆盖情况|逐议题统计|定位到具体哪几张卡缺料|
|时间分布（2017 年至今）|按年统计|支撑演变时间线是否做得出来|

**处置规则：**

|覆盖度|处置|
|---|---|
|充足|正常策展|
|部分议题缺料|该议题立场卡标注证据强度较弱，或由教师人工补充权威来源|
|整体偏低|建议替换为同类别中覆盖度更好的国家（见附录 A #11、#12）|

> **已知风险点**：俄罗斯在 WTO 渔业补贴谈判中的公开发言记录相对稀少，是本次体检的重点核查对象。

> 该体检的实现方式（统计脚本与产出报表）见《AI知识抽取工作流设计》前置步骤。

---

# 十一、对外接口需求（供应用层）

本节仅列能力需求，不含 API 细节。

|类别|能力|使用方|
|---|---|---|
|**检索**|混合检索（元数据过滤 + 关键词 + 向量），返回 Chunk 与 Evidence|知识导师 Agent|
|检索|按 `visibility` 与当前阶段过滤召回结果|知识导师 Agent|
|**教学资产读取**|按场景获取教学议题 + 方案区间|教师端建课|
|教学资产读取|按国家 × 议题获取立场卡与演变时间线|学生端阶段三、国家智库 Agent|
|教学资产读取|按国家获取画像与指标|学生端阶段二|
|**快照**|按参数生成课程快照，返回 manifest|教师端 Step 11|
|快照|按快照 ID 读取课程期内的稳定视图|全流程|
|**解锁**|教师触发 `student_post` 内容解锁|复盘阶段|
|**策展**|议题候选浏览、Evidence 挂接、审核流转|策展工作台|
|**管理**|知识库版本发布、数据质量校验报告|管理后台|

---

# 附录 A：待确认事项

|#|事项|影响|状态|
|---|---|---|---|
|1|~~WTO 文件三语去重后的实际 `doc_symbol` 数量~~|—|✅ **已确认**：仅保存英文版，841 份即唯一编号数|
|2|~~教学议题最终数量~~|—|✅ **已确认**：WTO 为 6 项（含技术援助机制）|
|3|参与国清单|立场卡数量|✅ **WTO 已确认 9 国**（见 4.11.1）；⚠️ **IOTC 场景需另配**（美、俄非 IOTC 成员）|
|4|IOTC 第一节选用的具体议题|IOTC 语料筛选与议题卡产出|⚠️ 待教师确认|
|5|`ProfileMetric` 的基准年份口径（统一取最新年 / 统一取某年）|数据导入规则|待教师 + 数据同学|
|6|立场卡演变时间线的起始年份（V6 提「2017 年至今」是否沿用）|演变节点数量|待教师确认|
|7|是否需要中文译文入库（原文英文 + 中文摘要 / 全文翻译）|存储与检索设计、成本|待确认|
|8|~~组内角色分工与评分单位~~|—|✅ **已确认**：评分单位为**小组**，`Delegation` 无需成员子表，录音标注到代表团即可|
|9|是否增补 LDC / 小岛屿国家参与国（见 4.11.1 教学覆盖度提示）|议题⑤等方案缺少提案人|⚠️ 待教师确认|
|10|俄罗斯在渔业补贴谈判中的语料证据是否充足|该国 6 张立场卡能否达到发布标准|待策展前做证据可得性检查|
|11|IOTC 法文版文件是否已下载保存|IOTC 去重机制是否需实际启用|待数据同学核实|
|12|向量库与关系库的具体技术选型|部署与运维|待开发确认|