# WTO 文档库文件分类说明文档（完善稿 v3）

> **文档状态**：工作稿，待老师审阅  
> **修订日期**：2026-11-24  
> **编写目的**：让开发团队在抽取 WTO 文档库数据时，能稳定判断“一份文件处于什么阶段、是什么性质”，从而约束模型回答，避免把过程性/草案表述当成现行正式规则。  
> **阅读提示**：文中〔E1〕〔E5〕等标记对应附录 A 的证据索引；凡标“假设”的内容均为初步推断，尚未完成大样本验证，需要老师判断是否继续投入。

---

## 目录

* [第 0 章 我们要解决什么问题（给老师的一页说明）](#第-0-章-我们要解决什么问题给老师的一页说明)
* [第 1 章 一份文件在搜索结果页包含哪些基础信息（原稿保留）](#第-1-章-一份文件在搜索结果页包含哪些基础信息原稿保留)
* [第 2 章 分类框架校准：对原有四种说法的核查与三维方案](#第-2-章-分类框架校准对原有四种说法的核查与三维方案)
* [第 3 章 〔第一部分〕根据文号和标题信息初筛文件类别](#第-3-章-第一部分根据文号和标题信息初筛文件类别)
* [第 4 章 〔第二部分〕阅读真实文件页面信息，判断文件类别](#第-4-章-第二部分阅读真实文件页面信息判断文件类别)
* [第 5 章 分类结果如何约束模型回答](#第-5-章-分类结果如何约束模型回答)
* [第 6 章 待老师确认清单](#第-6-章-待老师确认清单)
* [附录 A 证据索引](#附录-a-证据索引)
* [附录 B 中英术语对照](#附录-b-中英术语对照)
* [附录 C WTO 正式文件首页版式骨架示意](#附录-c-wto-正式文件首页版式骨架示意)

---

## 第 0 章 我们要解决什么问题（给老师的一页说明）

### 0.1 问题
我们手上有一个 WTO 文档库。模型在回答“WTO 对渔业补贴是怎么规定的”这类问题时，很容易把**谈判过程中的文本**当成**已经生效的规则**——例如把主席散发的第二轮谈判草案（`TN/RL/W/285`）说成“WTO 现行规则要求成员不得……”。这类事实错误的根源不是模型读不懂英文，而是**我们没在数据结构中告诉模型这份文件在程序上处于什么位置**。

### 0.2 我们的思路
我们建议做**两级判定**，而不是一步到位：

| 级别 | 输入 | 输出 | 成本 | 本稿位置 |
| :--- | :--- | :--- | :--- | :--- |
| **第一级：初筛** | 文号 + 英文原标题 | 候选性质 + 候选阶段 + 置信度（高/中/低） | 低，可批量跑 | 第 3 章 |
| **第二级：精判** | 文件首页版式 + 正文自述句 + 外部状态库 | 确认或纠正一级结论；补齐“是否已通过/已生效” | 中，需取页面 | 第 4 章 |

设计前提是：**初筛一定会有边界误判**，所以初筛的输出必须带置信度，且只在“高置信”时直接落库，中低置信一律进第二级。

### 0.3 我们已验证的三条核心结论
1. **WTO 官方分类维度与“过程性/草稿/已通过/已生效”不是一一对应关系**，它至少拆分为四套独立维度：文号结构、文件系列性质、访问权限、版本后缀〔E1〕〔E4〕-〔E10〕。
2. **“已通过”不等于“已生效”，且“生效状态”是动态变化的**。WTO《渔业补贴协定》（`WT/MIN(22)/33`）于 2022 年 6 月 17 日由部长级会议通过，但需三分之二成员（111 个）交存接受书方可生效；它在长达 **3 年零 3 个月**的时间里处于“已通过但未生效”状态，直到 **2025 年 9 月 15 日**才正式生效（且仅对交存接受书的成员生效）〔E12〕〔E13〕。如果标签混为一谈，模型必然出错。
3. **访问权限（Restricted）不等于文件不正式**。会议纪要（`M` 系列）是正式文号的正式文件，但依 `WT/L/452` 政策在发布后 45 天内为限制访问〔E6〕〔E7〕。把 `Restricted` 当成“非正式文件”是第二类常见错误。

### 0.4 需要老师优先决定的四件事
* **Q1**：是否接受用“三维标签 + 两项元数据”取代原来的单一“文件类别”字段？（第 2 章）
* **Q2-a**：暂定的十类文件性质是否够用？（第 2 章 2.4）
* **Q3**：初筛“仅高置信直接落库、中低置信强制进第二级”的门槛是否合适？（第 3 章）
* **Q4-a**：是否将“生效状态”独立为可动态更新的外部字段？（第 4 章 4.5）

---

## 第 1 章 一份文件在搜索结果页包含哪些基础信息（原稿保留）

> 本章为原有内容，保留原结构与图片路径；章末附我们对 3 处表述的校准建议（1.5）。

### 1.1 文号信息拆解

以链接里面的第 1 页第 2 份文档为例：[WTO 文档库搜索页结果链接](https://docs.wto.org/dol2fe/Pages/FE_Search/FE_S_S006.aspx?MetaCollection=WTO&SymbolList=&Serial=&IssuingDateFrom=&IssuingDateTo=&CATTITLE=&ConcernedCountryList=&OtherCountryList=&SubjectList=&TypeList=&FullTextHash=371857150&ProductList=&BodyList=&OrganizationList=&ArticleList=&Contents=&CollectionList=%22TN%22&RestrictionTypeName=&PostingDateFrom=&PostingDateTo=&DerestrictionDateFrom=&DerestrictionDateTo=&ReferenceList=&Language=ENGLISH&SearchPage=FE_S_S001&ActiveTabIndex=1&HSClassificationList=&ServicesClassificationList=&EnvironmentClassificationList=&ICSClassificationList=&ICSClassificationDescList%3aEnvironmentClassificationDescList%3aServicesClassificationDescList%3aHSClassificationDescList=&languageUIChanged=true)

![image.png](图片和附件/image%202.png)

**TN/RL/40：**
- `TN/RL/`：规则谈判组报告大类（对应第一张表第一行 `TN/RL/` 报告类）
- `40`：该系列下的顺序编号，是规则组第 40 号报告

`TN/RL/40` 代表一份文件的文号，根据文件 [WTO DOCUMENT NOMENCLATURE（世贸组织文件命名规则）](https://docs.wto.org/gtd/Complete_List_WTO_Collections_Symbols/WTO_document_nomenclature.pdf)（第 32 页）以及 [文件命名规范](https://docs.wto.org/gtd/Default.aspx?langue=e&pagename=WTOSymbols)：

![image.png](图片和附件/image.png)

![image.png](图片和附件/image%205.png)

### 1.2 标题信息拆解

同上例子，标题为：**Negotiating Group on Rules - Fisheries subsidies - Report By the Chair, H.E. Mr. Leslie Ramsammy to the Negotiating Group on Rules - 8 July 2026**

1. **Negotiating Group on Rules - Fisheries subsidies**
   - 归属：规则谈判组；具体议题：**渔业补贴（渔业补贴规则谈判，IUU 相关）**
2. **Report By the Chair, H.E. Mr. Leslie Ramsammy to the Negotiating Group on Rules**
   - 文件性质：**主席报告（Chair’s Report）**
   - 发文主体：规则谈判组主席 Leslie Ramsammy
   - 报送对象：规则谈判组全体成员
   - 暗示：这不是成员提案，也不是会议纪要，是主席汇总磋商进展、各方分歧、谈判文本状态，写给谈判组的汇总报告；这类报告经常向上提交给 TNC（TN/C）
3. **8 July 2026**：报告日期

### 1.3 附加信息拆解

同上例子：

**① Access: Unrestricted**
- `Unrestricted` = Public，无限制公开，和第一张表里 `TN/RL/` 密级 Public 匹配，文件共分为以下类别，按照官方 [文件公开与访问状态文档](https://docs.wto.org/gtd/Default.aspx?langue=e&pagename=WTODerestriction)：

![image.png](图片和附件/image%204.png)

**② 8 July 2026**：报告日期

**③ 附加元数据：**
- `10/07/2026`：WTO 系统录入 / 分发日期
- `80 KB`：文件大小
- `Pages 4`：文件页数
- `Doc #: 26-4930`：WTO 内部归档流水号
- 语言版本：英文、法文、西班牙文（WTO 三大官方语言）

### 1.4 文件状态信息拆解

以链接里面的第 2 页第 1 份文档为例（同一搜索链接）：

![image.png](图片和附件/image%203.png)

按照官方文件状态标识文档〔E2〕：

![image.png](图片和附件/image%201.png)

### 1.5 对本章 3 处表述的校准建议

| 序号 | 原表述 | 校准建议 | 依据 |
| :--- | :--- | :--- | :--- |
| 1-1 | “这类报告经常向上提交给 TNC（TN/C）” | 建议改为：**报送对象以标题中的 `to X` 为准**。`TN/RL/40` 标题是 `to the Negotiating Group on Rules`，即面向规则谈判组本身；主席向 TNC 作的报告通常是另一份独立文件（如 `TN/C/` 或新闻稿） | 规则谈判组页面说明与 `TN/RL/40` 标题报送对象不同〔E4〕〔E15〕 |
| 1-2 | 附加信息里第 ② 项“8 July 2026：报告日期”与 1.2 重复 | 建议合并；并将 `10/07/2026` 暂记为“分发/录入日期（`distribution_date`）”，与文件自身日期（`doc_date`）区分存储 | 搜索结果页元数据口径需统一 |
| 1-3 | “Doc #: 26-4930：WTO 内部归档流水号” | 建议补充：该编号对应文件首页左上角括号编号（形如 `(26-4930)`），是**页面与搜索结果交叉校验的关键锚点** | WTO 正式文件首页统一版式〔附录 C〕 |

> **【待老师确认 Q1-a】** 1-1 的校准是否成立？  
> 老师意见：______________________

---

## 第 2 章 分类框架校准：对原有四种说法的核查与三维方案

### 2.1 核查结论
**方向是对的，问题出在“这四种说法不在同一个维度上”**。如果直接做互斥单选标签，模型必然出现系统性事实错误。

| 原有说法 | 核查结论 | 建议术语 | 关键依据 |
| :--- | :--- | :--- | :--- |
| **过程性文件** | 官方无此统称，但确实存在一组边界清晰的文件（报告、纪要、来文/提案、工作文件、会议通知），共同点是“记录或推动进程，本身不创设权利义务” | **程序与记录类文件**（界面/教学可保留“过程性文件”别称） | 各机构文件系列说明将 Reports / Minutes / Communications / Working documents 与 Decisions 分列〔E4〕-〔E10〕 |
| **草稿文件** | **成立**，官方有明确对应词 `Draft text` / `Draft Decision` / `Draft Declaration` | **草案类文件**（需与版本修订 `Rev.` 区分） | MC12 文件页单列 `Package of Draft Decisions and Declarations`〔E11〕；`TN/RL/W/285` 标题含 `Draft text`〔E4〕 |
| **已通过文件** | **成立**，官方用 `Adopted` | **已通过（Adopted）**，限定为“已获有权机构正式通过” | MC12 文件页将已通过包与草案包分栏〔E11〕 |
| **已生效文件** | **不能与“已通过”并列在同一静态层**——两者之间存在长期中间态 | **已生效（In force）**，需作为独立动态字段 | 《渔业补贴协定》2022-06-17 通过，直至 2025-09-15 才正式生效〔E12〕〔E13〕 |

### 2.2 为什么必须拆维度：三个真实反例

1. **反例一：受限访问（Restricted）≠ 非正式文件**。规则谈判组会议纪要 `TN/RL/M/` 官方标注为 `Restricted`（发布后 45 天内受限），但它是具有正式文号的正式会议纪要〔image.png〕〔E6〕。
2. **反例二：W 系列 ≠ 草案**。`W` 系列（Working documents）同时包含“成员提案”和“主席草案”。例如 `WT/GC/W/773` 是中国关于 WTO 改革的提案（`COMMUNICATION FROM CHINA`），属于成员立场；而 `TN/RL/W/285` 是渔业补贴第二轮谈判草案（`Draft text`）〔E4〕。文号类型码相同，阶段完全不同。
3. **反例三：非正式文号里装着草案内容**。TRIPS 理事会的 `JOB/IP/` 系列被官方定义为非正式文件，但明确包含 `draft decisions`、`working negotiating texts`〔E9〕。因此“文号正式性”与“文本阶段”必须解耦。

### 2.3 推荐的字段体系（三维标签 + 两项元数据）

| 维度 | 字段名 | 取值范围 | 判定依据 |
| :--- | :--- | :--- | :--- |
| **A. 文件性质** | `doc_nature` | 报告 / 会议纪要 / 来文・提案 / 工作文件 / 决定・宣言・协定 / 通报 / 信息文件 / 行政通知 / 会议室文件 / 争端裁决 / 其他 | 文号系列 + 标题类型词 + 正文自述句 |
| **B. 文本阶段** | `stage` | `C0` 未定 / `C1` 程序记录 / `C2` 草案 / `C3` 已通过 / `C4` 已生效 | 标题 `Draft` + 正文方括号 + 外部通过/生效记录 |
| **C. 文号正式性** | `symbol_status` | 正式文号 / 非正式文号（`JOB`、`RD`） | 文号前缀 |
| **D. 访问权限** | `access` | `Unrestricted` / `Derestricted` / `Restricted` / `Secretariat`〔E2〕 | 搜索页 Access 字段 + 首页右上角标记 |
| **E. 版本关系** | `version` | 原件 / `Add.n` / `Corr.n` / `Rev.n` / `Suppl.n` / `Vol.n`〔image 1〕 | 文号后缀 |

**示例（修正后）**：
> 《`TN/RL/W/285`》（2024-11-29 散发；正式文号的**工作文件**；**草案文本 C2**，未经通过；`Access: Unrestricted`）—— 主席在**草案**中建议……

### 2.4 暂定的十类“文件性质”清单

| 性质 | 典型文号 / 标题信号 | 是否创设权利义务 | 依据 |
| :--- | :--- | :--- | :--- |
| 报告（Reports） | `TN/RL/` 主系列；标题 `Report by the Chair` | 否 | 〔image.png〕〔E4〕 |
| 会议纪要（Minutes） | `.../M/`；标题 `Minutes of the meeting` | 否（仅记录） | 〔E5〕〔E6〕 |
| 来文・提案（Communications/Proposals） | `.../W/`、`.../GEN/`、`.../COM/`；`Communication from` | 否 | 〔E5〕〔E8〕 |
| 工作文件（Working documents） | `.../W/` | 否（内含草案时另判 C2） | 〔E6〕〔E9〕 |
| 决定・宣言・协定（Decisions/Declarations/Agreements） | `WT/MIN(xx)/`、`WT/L/`、`G/L/`、`S/L/` | **是**（须核实通过/生效） | 〔E11〕〔E5〕 |
| 通报（Notifications） | `.../N/` | 否（透明度义务） | 〔E7〕〔E9〕 |
| 信息文件（Information documents） | `.../INF/` | 否 | 〔E5〕 |
| 行政通知（Airgrams/Convening notices） | `WTO/AIR/`、`ICN/` | 否 | 〔E7〕〔E8〕 |
| 会议室文件（Room documents） | `RD/` | 否，无正式文号地位 | 〔E4〕〔E9〕 |
| 争端裁决（DS reports/awards） | `WT/DS.../` | 对当事方有约束力，非立法 | 〔E1〕 |

> **【待老师确认 Q2-a】** 十类是否够用？是否需将“加入文件（`WT/ACC/`）、贸易政策审议报告（`WT/TPR/`）”单列？  
> 老师意见：______________________

---

## 第 3 章 〔第一部分〕根据文号和标题信息初筛文件类别

### 3.1 初筛的能力边界
初筛只做三件事：**定性质候选、定阶段候选、给置信度**。明确不做：
* 不直接判定“已通过（C3）”或“已生效（C4）”（文号和标题无法证明通过与否，见 4.5）；
* 遇到未收录系列，直接给 `C0` + 低置信，禁止猜测。

### 3.2 文号六段结构与版本后缀陷阱
按官方命名规则，文号由 `Collection / Series / Type / Country / Sequential number / Status suffix` 构成〔E1〕。
* **版本后缀陷阱**：`Rev.` 表示**整体替换原文件**（replaces entirely），`Add.` 表示增加实质内容，`Corr.` 表示更正错误或取消前文，`Suppl.` 表示补充非实质信息〔image 1〕。开发上必须识别 `Rev.` 链条，确保模型回答时**以最新 `Rev.` 为准**，而将 `Add.` 与原件合并检索。

### 3.3 系列 → 性质映射表（按机构建表，不按后缀全局建表）
**核心发现**：同一后缀在不同机构含义不同（例如 `TN/RL/GEN/` 为 Communications，而 `WT/GC/COM/` 才是 Communications）〔image.png〕〔E5〕〔E8〕。因此映射表必须精确到 `(collection, series)` 粒度：

| 文号模式 | 官方描述 | `doc_nature` | `stage` 候选 | 置信度 | 依据 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `WT/GC/`、`G/C/`、`S/C/`、`IP/C/`、`TN/RL/`（主系列） | 报告、年度报告、主席报告 | 报告 | C1 | 高 | 〔image.png〕〔E4〕-〔E9〕 |
| `.../W/`（如 `TN/RL/W/`、`WT/GC/W/`） | 工作文件（混装成员提案与主席草案） | 工作文件 / 来文・提案 | C1 或 **C2**（看标题） | 中 | 〔E4〕-〔E10〕 |
| 各机构 `.../M/` | 正式会议纪要（45 天内通常受限） | 会议纪要 | C1 | 高 | 〔E5〕〔E6〕 |
| `.../GEN/` | 一般来文/提案（`TN/RL/GEN` 为公开提案） | 来文・提案 | C1 | 中 | 〔image.png〕〔E4〕〔E8〕 |
| `.../N/` | 成员通报 | 通报 | C1 | 高 | 〔E7〕〔E9〕 |
| `.../INF/` | 信息文件（注：无 `WT` 前缀的 `INF/ECOM/` 等多为 JSI 非正式磋商文件） | 信息文件 / 待定 | C1 / C0 | 中/低 | 〔E5〕 |
| `WT/L/`、`G/L/`、`S/L/` | 法律文件、决定、豁免、认证 | 决定・宣言・协定 | **C3 候选** | 中 | 〔E5〕〔E7〕 |
| `WT/MIN(xx)/`（无 `W`） | 部长级会议通过的决定/宣言/协定 | 决定・宣言・协定 | **C3 候选** | 中高 | 〔E11〕〔E12〕 |
| `WT/MIN(xx)/W/` | 部长级会议草案 | 决定・宣言・协定（草案） | **C2** | 高 | 〔E11〕 |
| `WT/DS.../` | 争端解决文件 | 争端裁决 | 依标题 | 中 | 〔E1〕 |
| `JOB/` | 非正式文件（可能内含草案） | 依内容判 | C0/C1/C2 | 高（仅正式性） | 〔E5〕〔E9〕〔E14〕 |
| `RD/` | 会议室文件（仅归档用非官方文号，长期受限） | 会议室文件 | C1 | 高 | 〔E4〕〔E6〕〔E9〕 |
| `WTO/AIR/`、`ICN/` | 会议召集通知 | 行政通知 | C1 | 高 | 〔E7〕〔E8〕 |

### 3.4 标题类型词词典（必须基于英文原标题 `title_en`）

| 标题信号词 | 指向性质 | 指向阶段 | 备注 / 排除条件 |
| :--- | :--- | :--- | :--- |
| `Draft text` / `Draft Decision` / `Draft Declaration` | 决定・协定 或 工作文件 | **C2** | **排除**：若前面出现 `Comments on` / `Communication from`，则属 C1 提案而非草案本体 |
| `Communication from [Member]` / `Proposal by` | 来文・提案 | C1 | 成员单方立场，**不等于草案** |
| `Report by the Chair` / `Chairperson's Summary` | 报告 | C1 | 汇总进程或主席个人归纳 |
| `Minutes of the meeting` | 会议纪要 | C1 | 仅会议记录 |
| `Notification` | 通报 | C1 | 履行透明度义务 |
| `Decision` / `Declaration` / `Agreement`（无 `Draft`） | 决定・宣言・协定 | **C3 候选** | 仍需外部核实通过与生效记录 |
| `Note by the Secretariat` / `Background note` | 信息 / 工作文件 | C1 | 秘书处说明，非成员共识 |

### 3.5 初筛判定卡（组合规则 + 置信度）

| 规则 | 触发条件 | 输出 | 置信度 | 是否直接落库 |
| :--- | :--- | :--- | :--- | :--- |
| **R1** | 文号含 `/W/` 且标题含 `Draft`（且**不含** `Comments on` / `Communication from`） | 草案 C2 | 高 | 是 |
| **R2** | 文号 `WT/MIN(xx)/W/` | 草案 C2 | 高 | 是 |
| **R3** | 文号 `JOB/` 或 `RD/` | 非正式文号；性质依标题 | 高（正式性）/ 低（性质） | 仅落 `symbol_status` |
| **R4** | 主系列 + 标题 `Report by the Chair` / `Minutes` | 报告 / 纪要 C1 | 高 | 是 |
| **R5** | `WT/MIN(xx)/`（无 `W`）+ 标题含 `Decision`/`Declaration`/`Agreement` 且无 `Draft` | C3 候选 | 中高 | **否**，进第二级 |
| **R6** | `WT/L/`、`G/L/`、`S/L/` + 标题含 `Decision`/`Agreement` | C3 候选 | 中 | **否**，进第二级 |
| **R7** | 标题含 `Draft` 但同时含 `Comments on` / `Communication from` | 来文・提案 C1 | 中 | **否**，进第二级 |
| **R8** | 系列未收录或标题无类型词 | C0 | 低 | 否 |

### 3.6 真实样本初筛验证

| 文号 | 标题关键片段 | 初筛输出 | 与官方事实是否一致 | 证据 |
| :--- | :--- | :--- | :--- | :--- |
| `TN/RL/40` | Report By the Chair … to the Negotiating Group on Rules | 报告 / C1 / 正式 / Unrestricted | 一致 | 〔image.png〕〔image 2〕 |
| `TN/RL/W/285` | Additional Provisions on Fisheries Subsidies – **Draft text** | 工作文件 / **C2** / 正式 / Unrestricted | 一致（主席 2024-11-29 散发） | 〔E4〕〔E16〕 |
| `TN/RL/W/285/Add.1` | … – Explanatory note from the Chair – **Addendum** | 工作文件 / C2 / `Add.1` / Unrestricted | 一致 | 〔image 3〕〔E4〕 |
| `WT/MIN(22)/W/20` | Agreement on Fisheries subsidies – **Draft text** | 草案 / C2 | 一致 | 〔E11〕 |
| `WT/MIN(22)/33` | Agreement on Fisheries Subsidies | 协定 / C3 候选 $\rightarrow$ 经第二级核实为 **C4 已生效**（2025-09-15） | 一致 | 〔E12〕〔E13〕 |
| `WT/GC/W/773` | CHINA'S PROPOSAL ON WTO REFORM / COMMUNICATION FROM CHINA | 来文・提案 / C1（**非草案**） | 一致 | 〔E5〕 |
| `RD/TN/RL/184` | Fisheries Subsidies: Article A.1 and Article B | 会议室文件 / 非正式文号 | 一致 | 〔E4〕 |

> **【待老师确认 Q3-a】** R1-R8 这套规则是否符合预期？特别是 R6（`WT/L/`、`G/L/`、`S/L/` 是否必须逐份核实通过记录，不能直接默认 C3）？  
> 老师意见：______________________

---

## 第 4 章 〔第二部分〕阅读真实文件页面信息，判断文件类别

### 4.1 第二级要解决的核心问题
1. **拆解混装系列**：确认 `W` 系列或 `JOB` 文件里到底装的是成员单方提案还是主席草案；
2. **确认文本成熟度**：检查正文是否存在未决方括号 `[…]` 或 `DRAFT TEXT` 副标题；
3. **补齐程序与生效状态**：结合外部官方记录，确认“是否通过”及“是否生效”。

### 4.2 页面版式骨架（假设 H1，待扩样验证）
WTO 正式文件首页具有高度统一的版式骨架（详见附录 C 示意图）：
* **左上角**：WTO 徽标 + 括号流水号 `(YY-NNNN)`（与搜索页 `Doc #` 一一对应，用于防抓取错位）；
* **右上角**：依次为访问权限（仅受限时标 `RESTRICTED` / `DERESTRICTED`）、文号、文件日期、`Page: 1/N`、`Original: English`；
* **中部居中**：大写议题标题 + **类型副标题**（如 `COMMUNICATION FROM CHINA`、`DRAFT TEXT`、`REPORT BY THE CHAIR`，是最可靠的性质锚点）；
* **首段**：固定模板化的**正文自述句**。

> **假设 H1**：首页“类型副标题 + 首段自述句”可覆盖 90% 以上的性质精判。  
> **假设 H2（待验证）**：页脚是否逐页带文号与页码，暂不作为判定依据。

### 4.3 正文自述句词典与“方括号”强信号

| 首段自述句模板 | 含义 | 性质/阶段判定 |
| :--- | :--- | :--- |
| `The following communication, dated X, is being circulated at the request of the delegation of [Member].` | 成员来文，按其要求散发 | 来文・提案 / **C1** |
| `The following draft agreement text, dated X, has been prepared by the Chair / Co-convenors ...` | 主席/共同召集人起草的草案 | 工作文件 / **C2** |
| `Members are invited to ...` / `Delegations are invited to submit ...` | 邀请行动，非正式决定 | **C1** |
| `The Ministerial Conference decides that ...` / `Members hereby agree ...` | 决定/协定条文 | **C3/C4 候选**（须外部核实） |

* **方括号信号（Bracketed text）**：WTO 官方明确解释，谈判文本中的方括号 `[…]` 表示**尚未达成一致、仍在讨论的文本**〔E14〕。若正文出现大量未决方括号，即使标题未写 `Draft`，也强制判为 **C2 草案**，且禁止模型将括号内条款作为定论输出。

### 4.4 第二级判定决策树（命中即停）
1. **首页有 `RESTRICTED` / `DERESTRICTED`？** $\rightarrow$ 仅写入 `access` 字段，**不参与性质判定**。
2. **文号为 `JOB/` 或 `RD/`？** $\rightarrow$ `symbol_status = 非正式文号`，继续第 3 步判内容。
3. **读取类型副标题 + 首段自述句** $\rightarrow$ 锁定 `doc_nature`。
4. **副标题含 `DRAFT`，或正文含未决方括号 `[…]`？** $\rightarrow$ `stage = C2`。
5. **属 `WT/MIN(xx)/`（无 W）或 `WT/L/`、`G/L/`、`S/L/` 且为决定/协定？** $\rightarrow$ 进入 4.5 外部核实；核实前一律标 `C3 候选`，**严禁标“已生效”**。

### 4.5 页面本身证明不了的两件事：通过与生效
一份文件 PDF 自身无法证明它“后来有没有被通过”或“现在有没有生效”。必须依赖四条外部核验路径：

| 路径 | 核验方式 | 能确认什么 | 证据 |
| :--- | :--- | :--- | :--- |
| **P1 部长级会议文件页** | 查看 MC 官方分栏（`Adopted` 与 `Draft` 分开列出） | 是否已被部长级会议通过 | 〔E11〕 |
| **P2 理事会/委员会记录** | 检索对应机构的通过纪要与决定 | 是否已被理事会/委员会通过 | 〔E5〕〔E6〕 |
| **P3 生效条款 + 接受书计数** | 查协定生效门槛 + WTO 官网接受书交存状态 | 是否达到生效条件（`C4`） | 〔E13〕 |
| **P4 草案与通过件映射** | 比对文本内容（不可仅凭相邻编号推断） | 版本沿革（如 `WT/MIN(22)/W/20` $\rightarrow$ `WT/MIN(22)/33`） | 〔E11〕〔E12〕 |

**核心动态案例（建议纳入评测集）**：
* 《渔业补贴协定》（`WT/MIN(22)/33`）于 **2022-06-17** 通过（进入 `C3`），需 111 个成员交存接受书；在长达 3 年多的时间里处于“已通过、未生效”状态，直至 **2025-09-15** 达到门槛正式生效（转为 `C4`，且仅对已接受成员生效，同时触发 4 年日落条款，若 2029 年 9 月前未达成第二波补充规则将失效）〔E13〕。

> **【待老师确认 Q4-a】** 是否认可将“生效状态（`in_force`）”设为独立动态字段，并仅对核心协定做定期同步？  
> 老师意见：______________________

### 4.6 抽样验证方案
* **分层抽样**：按 6 类各抽 5 份（共 30 份）——主系列报告、W 系列、M 系列纪要、WT/MIN 草案、WT/MIN 通过件、JOB/RD 非正式文件。
* **双盲标注与验收**：两人独立按 4.4 决策树标注；精判内部一致率 **$\ge 90\%$** 方可作为落库标准，同时统计初筛与精判的差异率，反哺修订第 3 章规则。

> **【待老师确认 Q4-b】** 30 份样本量与 90% 一致率阈值是否合适？  
> 老师意见：______________________

### 4.7 抽取字段规范与 JSON 示例（供开发直接实现）

```json
{
  "symbol": "TN/RL/W/285",
  "symbol_parts": {
    "collection": "TN",
    "series": "RL",
    "type_code": "W",
    "country": null,
    "seq": "285",
    "version_suffix": "Original"
  },
  "title_en": "Negotiating Group on Rules - Additional Provisions on Fisheries Subsidies - Draft text",
  "doc_date": "2024-11-29",
  "distribution_date": "2024-11-29",
  "doc_number": "24-8425",
  "pages": 25,
  "access": "Unrestricted",
  "symbol_status": "正式文号",
  "doc_nature": "工作文件",
  "stage": "C2",
  "in_force": "不适用",
  "has_brackets": true,
  "evidence_quote": "Additional Provisions on Fisheries Subsidies – Draft text",
  "confidence": "高",
  "review_stage": "精判确认"
}
````

### 4.8 工程获取约束

`docs.wto.org` 对自动爬虫有 `robots.txt` 限制。建议备选路径：人工/授权批量导出，或仅对“高价值议题 + 初筛中低置信”文档做页面精判。

  

> **【待老师确认 Q4-c】** 页面获取优先走人工下载还是限定范围精判？
> 
>   
> 
> 老师意见：______________________
> 
>   

## 第 5 章 分类结果如何约束模型回答

### 5.1 引用必须携带状态标签（System Prompt 强制约束）

模板：`《[文号]》（[日期]，[doc_nature]，[stage 中文]，Access: [access]）—— [正文主张]`

  

**正确示例**：

  

> 《`TN/RL/W/285`》（2024-11-29，正式文号的工作文件，**草案文本，未经通过**，`Access: Unrestricted`）中，规则谈判组主席建议对造成产能过剩和过度捕捞的补贴设置禁止性条款。这**不构成** WTO 现行义务。
> 
>   

### 5.2 五类禁止句式（对应五类事实错误）

|**#**|**错误句式**|**正确改写**|**触发标签**|
|---|---|---|---|
|1|“WTO 规定成员不得对过度捕捞提供补贴……”|“主席在第二轮谈判草案（`TN/RL/W/285`）中提议禁止……”|`stage=C2`|
|2|“成员已就渔业补贴补充规则达成共识”|“主席在报告（`TN/RL/40`）中总结了各方进展，分歧仍在”|`C1 + 报告`|
|3|“该文件为非正式文件”（理由：`Restricted`）|“该文件为正式文号的会议纪要，45 天内限制访问”|`access ≠ symbol_status`|
|4|“该协定一经部长级会议通过即对所有成员生效”|“该协定于 2022-06-17 通过，2025-09-15 达到 2/3 门槛后正式生效（且仅对交存接受书的成员生效）”|`C3 ≠ C4`|
|5|“根据 WTO 最新文本……”（引用已被 `Rev.` 替换的旧版）|明确写明“以最新修订版 `Rev.n` 为准”|`version_suffix`|

### 5.3 不确定时的降级话术

当 `confidence=低` 或 `stage=C0` 时，强制输出：“现有材料未能确认该文件的最终通过或生效状态，仅能确认其为某机构的[性质]。”严禁使用“应该是”“显然已生效”等猜测词。

  

> **【待老师确认 Q5-a】** 是否将上述 5 类禁止句式做成 25 条评测集作为模型上线验收门槛？
> 
>   
> 
> 老师意见：______________________
> 
>   

## 第 6 章 待老师确认清单

建议优先拍板 **Q1、Q2-a、Q3、Q4-a** 四项，其余可随开发推进逐步确认：

  

|**编号**|**问题**|**我们的建议**|**老师意见**|
|---|---|---|---|
|**Q1**|是否用“三维标签 + 两元数据”取代单一“文件类别”字段|建议采用（第 2.3）||
|**Q2-a**|十类文件性质是否够用（是否单列加入/TPR 文件）|建议先跑十类，按实际数据扩充||
|**Q1-a**|第 1.5 的 3 处原稿表述校准是否成立|建议按 1.5 修改||
|**Q3**|初筛“仅高置信直接落库”的门槛是否合适|建议维持（R1/R2/R4 高置信）||
|**Q3-a**|`WT/L/`、`G/L/`、`S/L/` 能否直接视为 C3|建议不能，须核实通过记录||
|**Q4-a**|是否将“生效状态”独立为动态字段|建议独立（已通过 ≠ 已生效）||
|**Q4-b**|抽样 30 份、精判一致率 $\ge 90\%$ 是否合适|建议按此执行||
|**Q4-c**|页面获取路径（人工/授权/限量精判）|建议限定“草案类 + 部长级文件”||
|**Q5-a**|是否建 25 条禁止句式评测集|建议建，作为上线门槛||

## 附录 A 证据索引

|**编号**|**来源**|**链接**|**支撑本稿哪部分**|
|---|---|---|---|
|E1|WTO 文件符号总说明|https://docs.wto.org/gtd/Default.aspx?langue=e&pagename=WTOSymbols|3.2 文号六段结构|
|E2|WTO 文件访问权说明|https://docs.wto.org/gtd/Default.aspx?langue=e&pagename=WTODerestriction|1.4、2.3-D|
|E3|WTO Document Nomenclature（官方命名规则 PDF）|https://docs.wto.org/gtd/Complete_List_WTO_Collections_Symbols/WTO_document_nomenclature.pdf|第 1 章|
|E4|规则谈判组文件系列说明 + 渔业补贴草案清单|https://www.wto.org/english/tratop_e/rulesneg_e/rulesneg_e.htm|3.3、3.6（`TN/RL/W/285` 为 Unrestricted）|
|E5|总理事会文件系列说明|https://www.wto.org/english/thewto_e/gcounc_e/gcounc_e.htm|2.4、3.3|
|E6|货物贸易理事会文件系列说明（纪要 45 天受限）|https://www.wto.org/english/tratop_e/gatt_e/gatt_e.htm|3.3、反例一|
|E7|服务贸易理事会文件系列说明|https://www.wto.org/english/tratop_e/serv_e/cts_e.htm|3.3|
|E8|TBT 委员会文件系列说明|https://www.wto.org/english/tratop_e/tbt_e/tbt_com_e.htm|3.3 后缀语义不统一|
|E9|TRIPS 理事会文件系列说明（`JOB/IP` 含草案）|https://www.wto.org/english/tratop_e/trips_e/intel6_e.htm|反例三|
|E10|贸易谈判委员会文件系列说明|https://www.wto.org/english/tratop_e/dda_e/tnc_e.htm|3.3|
|E11|MC12 官方文件页（已通过包与草案包分栏）|https://www.wto.org/english/thewto_e/minist_e/mc12_e/documents_e.htm|2.1、4.5-P1|
|E12|MC12 成果页（Geneva Package）|https://www.wto.org/english/thewto_e/minist_e/mc12_e/mc12_e.htm|0.3、3.6|
|E13|WTO 官方新闻：《渔业补贴协定》于 2025-09-15 正式生效|https://www.wto.org/english/news_e/news25_e/fish_15sep25_e.htm|0.3、4.5（动态生效核心案例）|
|E14|WTO 官方术语解释（`bracketed` 与 `Job document`）|https://www.wto.org/english/news_e/news11_e/agng_21jan11_e.htm|4.3 方括号、3.3 JOB|
|E15|规则谈判组主席向 TNC 的报告|https://www.wto.org/english/news_e/news24_e/tnc_12dec24_e.pdf|1.5-1-1|
|E16|第三方分析：`TN/RL/W/285` 谈判状态|https://www.ris.org.in/sites/default/files/Publication/Policy-Brief-No-135-Professor-Mukesh-Bhatnagar-Pankhuri-Gaur-and-Mr-Ayush-Tiwari.pdf|3.6|

## 附录 B 中英术语对照

|**中文（建议用）**|**English**|**备注**|
|---|---|---|
|文号|document symbol|唯一标识|
|状态后缀|status suffix|`Add`/`Corr`/`Rev`/`Suppl`/`Vol`〔image 1〕|
|修订（整体替换）|revision (`Rev.`)|**整体替换原件**|
|程序与记录类文件|process record|建议术语（界面可保留“过程性文件”）|
|草案|draft text / draft decision|`stage=C2`|
|已通过|adopted|`stage=C3`|
|已生效|in force / entry into force|`stage=C4`，需外部动态核实|
|非正式文件|informal document (`JOB`)|无正式文号地位|
|会议室文件|room document (`RD`)|仅归档用非官方文号|
|访问权限|access rights|与正式性无关|
|解除限制|derestriction|依 `WT/L/452` 政策〔E6〕|

## 附录 C WTO 正式文件首页版式骨架示意

> 注：为避免外链失效，此处以纯文本结构化展示 WTO 正式文件首页的固定位置锚点，供开发团队写 PDF/OCR 解析规则参考（抽样验证阶段可补入真实截图）。

Plaintext

```
+-------------------------------------------------------------------+
| [WTO 徽标]                                 RESTRICTED (仅受限时显示) |
| (24-8425) <-- 对应搜索页 Doc #              TN/RL/W/285             |
|                                            29 November 2024       |
|                                            Page: 1/25             |
|                                            Original: English      |
+-------------------------------------------------------------------+
| Negotiating Group on Rules   <-- 发文机构                          |
|                                                                   |
|        ADDITIONAL PROVISIONS ON FISHERIES SUBSIDIES  <-- 议题标题  |
|                        DRAFT TEXT                    <-- 性质副标题 |
|                                                                   |
| The following draft text, dated 29 November 2024, is being        |
| circulated by the Chair...   <-- 正文自述句（精判最强证据）          |
+-------------------------------------------------------------------+
```