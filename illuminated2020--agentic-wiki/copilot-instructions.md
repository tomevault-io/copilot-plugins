## agentic-wiki

> 本目录是一个由 Agent 维护、由人和 Agent 共同消费的**多域个人第二大脑**。所有知识库操作必须先理解本文件，再按相关域的 `CLAUDE.md` 执行。

# Agentic Wiki 操作协议

本目录是一个由 Agent 维护、由人和 Agent 共同消费的**多域个人第二大脑**。所有知识库操作必须先理解本文件，再按相关域的 `CLAUDE.md` 执行。

## 规则权威边界

- wiki skill 负责意图路由、操作编排和 Lint 的实际检查方式。
- 本文件负责项目边界、工作流契约、通用页面字段、命名、链接、索引、日志和分享边界。
- `sources/CLAUDE.md` 与各知识域的 `CLAUDE.md` 负责本域页面类型、局部字段、正文结构和专属 workflow。
- 每条规则只在其作用域内定义一次：全库不变量放根协议，域内数据契约跟随数据放在域协议中。

## 项目定位

- 定位：个人第二大脑——技术知识、工程经验、人际关系、想法捕获、写作管线、反思与原则。
- 基础设施层：`sources/` 保存原文和 source 摘要，`logs/` 保存追加式操作历史。
- 知识域：`tech/`、`entities/`、`qa/`、`learning/`、`craft/`、`ideas/`、`people/`、`writing/`、`reflections/`。
- 人侧界面：Obsidian。
- 目标：把资料加工成可链接、可追溯、可复用的长期知识，而不是堆积收藏。

## 渐进式加载（默认读取顺序）

所有 wiki 操作先读取共同入口，再按 Query 与写操作分流。

### 共同入口

1. `purpose.md`：确认项目边界。
2. 本文件（`CLAUDE.md`）：确认路由和全局规则。
3. `index.md`：全库内容导航。

### Query 分支

4. 相关域的 `index.md`。
5. 具体内容页。
6. 若发现需要写回，再进入写操作分支。

### 写操作分支（Ingest / Update / Lint / Restructure）

4. 相关域或基础设施层的 `CLAUDE.md`：确认局部页面契约和 workflow。
5. 相关域的 `index.md`。
6. 具体页面。

不要一开始扫描全库。域级 `CLAUDE.md` 是规则文件，不承担内容导航；内容导航一律走 `index.md`。

注意：这里限制的是**阅读理解和内容定位**，不是命令式结构验证。Lint、写后验证、断链检查、索引覆盖检查可以并且应该用 `find` / `grep` / `rg` 等命令扫描相关范围或全库。

## 域路由

AI 收到任务时，根据内容自行判断属于哪个域。执行前必须声明：

> "我判断这属于 X 域，读取 X/CLAUDE.md。"

如果任务跨域（例如从技术文章中提炼工程经验），声明主域和关联域：

> "这主要属于 craft/ 域，同时关联 tech/ 域。读取 craft/CLAUDE.md。"

### 域判断规则

| 信号 | 归属域 |
|------|--------|
| 技术文章入库、概念整理、来源分析、技术对比 | `tech/` |
| 组织、产品、协议、工具、框架、模型、数据集等具名非人类对象 | `entities/` |
| 对话中产生的有价值结论、讨论沉淀 | `qa/` |
| 学习路径规划、刷题记录、进度追踪 | `learning/` |
| 个人工程实践总结、方法论提炼、踩坑经验 | `craft/` |
| 产品灵感、项目构想、写作种子 | `ideas/` |
| 人物信息、互动记录、账号管理 | `people/` |
| 长文写作、文章状态跟踪 | `writing/` |
| 个人原则、认知变化、阶段性反思 | `reflections/` |

### 域边界歧义

如果内容同时符合两个或更多域，不得只按关键词或最先想到的域路由：

1. 列出候选主域和关联域。
2. 读取所有候选域的 `CLAUDE.md`；基础设施层候选同时读取其协议。
3. 按页面的主要职责判断归属：它长期要回答什么问题、由哪个 index 导航、后续按哪个 workflow 维护。
4. 搜索候选域的 index 和现有页面，检查同义、近义或已承担该职责的页面。
5. 声明主域、关联域，以及不选择其他候选域作为主域的理由。
6. 同一知识只保留一个 canonical 页面；跨域价值通过 Wikilink、`related` 或关联页面表达，不复制建页。

只有存在域歧义时才执行上述比较；明确内容继续按路由表和目标域协议渐进加载，不要求每次读取所有域。

## 目录边界

| 路径 | 规则 |
|------|------|
| `sources/` | 基础设施层。分为 `raw/`（不可变原文）和媒介类型子目录（AI 摘要页）。 |
| `sources/CLAUDE.md` | 来源层页面契约、raw/source 边界和项目快照 workflow。 |
| `sources/raw/` | **不可变原文存储**。抓取或粘贴的完整原始内容，LLM 只读不写（新建除外）。这是知识库的真理来源。 |
| `sources/articles/` | AI 生成的 source 摘要页（博客、公众号、新闻、essays），可随理解和来源协议演进更新。 |
| `sources/books/` | 书籍摘要页。 |
| `sources/papers/` | 学术论文、技术报告摘要页。 |
| `sources/videos/` | 视频来源摘要页。 |
| `sources/podcasts/` | 播客来源摘要页。 |
| `sources/notes/` | 个人笔记、对话记录、会议纪要。 |
| `sources/projects/` | 外部项目、teach 工作区、代码阅读材料的自包含快照。外部路径只能作为 provenance，不得作为唯一证据。 |
| `tech/` | 技术知识域。遵循 `tech/CLAUDE.md`。 |
| `tech/concepts/` | 概念、机制、方法论。 |
| `tech/queries/` | 未解决问题。 |
| `tech/comparisons/` | 横向对比。 |
| `tech/synthesis/` | 跨来源综合结论。 |
| `tech/index.md` | 页面导航入口，新增页面必须更新。 |
| `tech/overview.md` | 全库当前认知主线总览，知识库主线变化时更新。 |
| `entities/` | 具名非人类对象（组织、产品、协议、工具、框架、模型、数据集）。遵循 `entities/CLAUDE.md`。 |
| `entities/index.md` | 实体导航入口，新增实体必须更新。 |
| `logs/` | 基础设施层。操作日志按月归档（`logs/YYYY-MM.md`），严格只追加。 |
| `logs/index.md` | 月度日志导航入口。 |
| `index.md` | 全库内容导航唯一入口，只列知识域与基础设施 index、追加式列表、协议入口。 |
| `qa/` | 问答沉淀域。遵循 `qa/CLAUDE.md`。 |
| `learning/` | 学习路径域。遵循 `learning/CLAUDE.md`。 |
| `craft/` | 工程经验域。遵循 `craft/CLAUDE.md`。 |
| `craft/practices/` | 经验条目。 |
| `ideas/` | 想法捕获域。遵循 `ideas/CLAUDE.md`。**AI 不自动写入。** |
| `ideas/writing.md` | 写作类想法。 |
| `ideas/products.md` | 产品类想法。 |
| `people/` | 人际关系域。遵循 `people/CLAUDE.md`。 |
| `people/real/` | 真实关系人。 |
| `people/public/` | 公开人物。 |
| `people/accounts/` | 关注的内容账号。 |
| `writing/` | 写作管线域。遵循 `writing/CLAUDE.md`。 |
| `writing/ideas.md` | 写作主题池。 |
| `writing/articles/` | 文章条目。 |
| `reflections/` | 反思与原则域。遵循 `reflections/CLAUDE.md`。 |
| `reflections/principles.md` | 个人原则汇总。 |
| `todo.md` | 按域分类的任务列表。**AI 不自动写入，仅在用户确认后追加。** |

## 全局页面契约

本节定义所有域共享的不变量。具体页面类型的局部字段、正文结构和专属状态流转由其所在目录的 `CLAUDE.md` 定义。

### 页面类型路由

| Type | 权威协议 | 典型位置 |
|---|---|---|
| `source` | `sources/CLAUDE.md` | `sources/articles/`、`sources/papers/`、`sources/projects/` 等 |
| `entity` | `entities/CLAUDE.md` | `entities/` |
| `concept`、`query`、`comparison`、`synthesis`、`overview` | `tech/CLAUDE.md` | `tech/` |
| `qa` | `qa/CLAUDE.md` | `qa/` |
| `learning` | `learning/CLAUDE.md` | `learning/` |
| `practice` | `craft/CLAUDE.md` | `craft/practices/` |
| `idea` | `ideas/CLAUDE.md` | `ideas/writing.md`、`ideas/products.md` |
| `person` | `people/CLAUDE.md` | `people/real/`、`people/public/`、`people/accounts/` |
| `article` | `writing/CLAUDE.md` | `writing/articles/` |
| `reflection` | `reflections/CLAUDE.md` | `reflections/` |
| `index`、`log` | 本文件 | 根/域级 index、`logs/YYYY-MM.md` |

新增页面类型时，优先在归属域的 `CLAUDE.md` 定义；只有跨域不变量变化时才修改本节。

### 建页门槛

新建页面前必须先给出候选决策：

```markdown
| 候选 | 类型 | 决策 | 原因 | 目标页面 |
|---|---|---|---|---|
| 名称 | 页面类型 | Create/Update/Skip | 简短原因 | 页面名或现有链接 |
```

只有满足以下任一条件才允许新建独立页面：

- 会被多个页面反复引用。
- 支撑当前知识库主线，或改变已有判断。
- 对后续技术决策、写作、学习、关系维护或工程实践有长期价值。
- 需要长期追踪版本、关系、状态或未解决问题。

一次性提及、只有一句定义、与已有页同义或真实性未确认的名词默认留在 source、query 或已有页面中。

### 命名规范

- 文件名优先使用人类可读标题；中文概念允许中文文件名。
- 多词文件名使用空格，不使用连字符或下划线，例如 `Model Context Protocol.md`。
- 人物使用姓名或常用称呼；英文项目、协议、模型和工具保留官方名称。
- 同一对象只保留一个 canonical 页面，别名写入 `aliases`。
- 避免 `xxx 1.md`、`xxx_副本.md` 等临时命名；新建前先搜索相关域 index 和目录。

### 通用 Frontmatter

除 raw、月度日志和追加式列表外，普通页面继承以下字段：

```yaml
---
type: concept
title: 页面显示名
description: 一句话说明页面内容
aliases: []
tags: []
related: []
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: active
confidence: emerging
share_scope: internal
shareable: false
sources: []
---
```

- `type`、`title`、`description`、`created`、`updated` 必填。
- 日期使用 `YYYY-MM-DD`。月度 log 只要求 `created`，不写 `updated`。
- `aliases`、`tags`、`related`、`sources` 为数组；`related` 不替代正文链接。
- 通用 `status`：`active | draft | stale | archived`；域协议可以定义更具体的状态集合。
- 通用 `confidence`：`emerging | established | validated`；具体升级规则由域协议定义。
- `share_scope`：`public | internal | private | sensitive`。
- `shareable` 表示页面当前是否已整理到可分享状态，与敏感级别不同。

追加式列表 `todo.md`、`ideas/writing.md`、`ideas/products.md`、`writing/ideas.md` 不要求标准 Frontmatter。

### 分享边界

`share_scope` 只约束内容被带出 wiki 的方式，不限制个人 Query 的读取：

- `public`：可直接引用。
- `internal`：可抽象使用，不直接暴露原文细节。
- `private`：外发前必须取得用户确认。
- `sensitive`：默认不得外发，仅在用户明确指定具体内容时使用。

普通技术知识和公开 source 默认 `internal`；各域可以在域协议中定义更严格的默认值。`sources/raw/` 不加 Frontmatter，隐私级别由对应 source 摘要页承担。

### Index 规范

- 根 `index.md` 只列知识域、基础设施 index、追加式列表、当前主线和协议入口。
- 每个知识域必须有 `type: index` 的 `index.md`；`sources/index.md` 与 `logs/index.md` 是基础设施入口。
- `tech/index.md` 按页面类型分组，`entities/index.md` 按实体类别分组，其他域按需组织。
- Index 条目使用“页面链接 — 一句话说明”，说明应能独立表达页面价值。
- source 摘要只登记在 `sources/index.md`，不在 `tech/index.md` 重复维护。
- 新增、移动、重命名、删除页面时必须更新 canonical index；仅更新正文时不强制改 index，但必须验证覆盖。
- 空域 index 保留并写“当前暂无条目”。

### Log 规范

- 日志按月归档为 `logs/YYYY-MM.md`，顶部只有一段 Frontmatter，正文严格追加。
- operation 可取：`init | ingest | query | lint | update | restructure | reconcile`。
- 条目标题格式：`## [YYYY-MM-DD] <operation> | <标题>`。
- 每次写操作必须追加当月日志；`logs/index.md` 只列月份文件。

### Obsidian 链接规则

- 页面间使用真实存在的 `[[Wikilink]]`；显示名不同时使用 `[[path/page|显示名]]`。
- 首次提到重要实体或概念时应链接对应 canonical 页面。
- `related` 是辅助索引，正文 Wikilink 才是主要知识图谱来源。
- 外部 URL 使用普通 Markdown 链接。
- raw 与 source 摘要同名时，source 必须使用路径限定链接；raw 只通过文件路径或 `canonical_raw` 引用。
- 断链应立即修复；确实暂时保留时记录到 query 或日志。

### 结构变更规范

- 顶级知识域必须有可路由边界、`CLAUDE.md`、`AGENTS.md`、`index.md`，并出现在根 index。
- 新域的局部页面契约直接定义在 `<domain>/CLAUDE.md`，不创建独立 schema 文件。
- 非空域优先归档或迁移，不直接删除；移动时同步更新协议、index、链接和日志。
- `sources/` 与 `logs/` 是基础设施层，不适用知识域必备文件规则。

### 矛盾与维护原则

- 来源冲突时写明冲突点，创建或更新 query；证据足够后再形成 synthesis 或 reflection。
- 不静默覆盖旧结论，重要变化写入日志。
- 原始资料进入来源层；source 写加工摘要；知识域页面写综合和解释。
- 优先更新已有页面，不创建近义重复页；一页一主题，重要判断必须可追溯。
- 面向用户的内容默认使用中文，必要术语保留英文。

## ideas/ 和 todo.md 特殊约束

### ideas/

**AI 不得自动向 `ideas/` 写入内容。** 当 AI 识别到可能属于想法捕获的内容时：

1. 向用户说明："这看起来是一个想法/灵感，是否要记录到 ideas/？"
2. 等待用户确认后才写入。
3. 如果用户未确认，不写入也不再追问。

这是因为想法域的价值在于用户的主动筛选，而非 AI 的自动归档。

### todo.md

**AI 不得自动向 `todo.md` 写入内容。** 当 AI 识别到可能的任务时：

1. 向用户说明："这看起来是一个待办事项，是否要记录到 todo.md？"
2. 等待用户确认后才追加到对应域分类下。
3. 如果用户未确认，不写入也不再追问。

## 工作流：Ingest

当用户给一篇文章、协作文档、GitHub 项目、博客、论文、经验总结或其他来源并要求入库时：

1. 先判断是否在 `purpose.md` 范围内。
2. **幂等检查**：搜索 `sources/` 中是否已存在同标题或同 URL 的 source 文件。如果已存在，跳过新建 source，仅做知识更新。
3. **获取并保存原文**：
   - URL 来源：用 WebFetch 或等效手段获取完整原文，保存到 `sources/raw/<slug>.md`（纯 Markdown，无 frontmatter）。文件名 slug 与对应的 source 摘要页一致。
   - 用户粘贴全文：直接保存到 `sources/raw/<slug>.md`。
   - 外部项目/本地文件：在 `sources/projects/` 保存快照（规则不变）。
   - 原文保存后**不可修改**（只能新建，不能编辑已有 raw 文件）。
   - 如果发现已有 raw 抓取不完整或格式严重损坏，不修改原 raw；新建同 slug 的修正版 raw（如 `<slug>.v2.md` 或 `<slug>-refetch-YYYY-MM-DD.md`），并在对应 source 摘要页记录当前 canonical raw 与旧 raw 的问题。
   - 如果获取原文失败（如需登录、付费墙），在 source 摘要页 frontmatter 标注 `raw_available: false` 并说明原因。
4. **基于原文分析**：后续所有 Ingest 步骤必须基于 `sources/raw/` 中的完整原文进行，而非仅凭 URL 标题或模型记忆。
5. 判断归属域，声明路由决策；存在域歧义时执行「域边界歧义」规则。
6. 读取目标域的 `CLAUDE.md` 获取域级规则；存在候选相邻域时同时读取其协议。
7. **候选清单**：在写文件前列出来源中出现的实体、人物、概念、问题和综合判断候选。
8. **写入决策**：对每个候选给出 `Create / Update / Skip + Reason`，并先查相关域索引和文件，避免重复页面。
   - 明确归属的候选沿用全局页面契约中的标准决策表。
   - 存在域歧义的候选使用：`候选 | 主域 | 相邻域 | 决策 | 边界理由 | 目标页面`。
   - `边界理由` 必须说明页面的主要职责，以及为什么不在相邻域重复建页。
9. **Notability Gate**：只有满足以下任一条件才新建页面：会被多个页面反复引用、支撑当前主线、影响用户后续决策、需要长期追踪。一次性提及的名词留在 source 或相关页面中，不建空壳页。
10. 如果是新来源，在 `sources/articles/`（外部文章）或对应媒体类型子目录创建 source 摘要页。
11. 在目标域创建或更新页面，遵循该域 `CLAUDE.md` 的局部页面契约。
12. 更新受影响的已有页面，补充新证据、新链接或新不确定性。
13. 更新受影响的 canonical index：source 摘要只登记在 `sources/index.md`，知识页面登记在对应域 index。新增、删除、移动、重命名页面时必须更新；仅更新已有页面正文时不强制改 index，但必须验证覆盖没有遗漏。
14. 如果来源改变了知识库主线，更新 `tech/overview.md`。
15. **写后验证（必须执行，不是可选项）**：
    - **Wikilink 扫描**（命令验证）：对本次新增/修改的页面，跑 `rg` + `find` 扫描所有 `[[wikilink]]` 是否指向真实文件。不要凭记忆确认。同名 raw/source 存在时，source 摘要必须使用路径限定双链，raw 只以文件路径或 `canonical_raw` 引用。
    - **Index 覆盖**（命令验证）：每个新增/重命名页面必须在对应域 `index.md` 中有一条目。用 grep 验证。
    - **Frontmatter 完整性**（命令验证）：新增页面必须有 `type`/`title`/`description`/`created`/`updated`；月度 log 不要求 `updated`。source 页额外检查 `rating`/`status`/`media`/`raw_available`。
    - **URL→wikilink 转换**（阅读验证）：逐段阅读正文，搜索 `[text](/en/...)` 格式。若目标概念在 wiki 中已有页面，替换为 `[[page]]`。从英文文档复制内容时最容易漏。
    - **知识表示一致性**（阅读验证）：回顾来源中哪些概念被当作同级讨论（对比表、并列列表、等长章节、枚举句如"X 支持 A、B、C"）。对每个来源视为同级的概念，检查 wiki 是否给予了平等的页面粒度——有没有人被建了独立页、有人被塞在父页的子章节里。
    - 验证未通过不得声称 Ingest 完成。断链立即修复，索引遗漏立即补入。
    - 将验证结果写入交付说明。
16. 在 `logs/` 当月日志文件中追加条目，记录本次 Ingest 影响的文件列表和候选决策摘要。

交付时说明：新增了哪些页面、更新了哪些页面、关键发现是什么、还有什么待确认。

## 工作流：Query

当用户询问这个知识库里的内容：

1. 判断问题涉及哪个域，读取相关域的索引。
2. 优先读主线文件和最相关的 2-5 个页面。
3. 回答必须引用页面名，例如"根据 `[[示例概念 A]]` 和 `[[示例概念 B]]`"。
4. 如果答案依赖某个 source，点明 source 页面。
5. 跨域查询时，说明信息分布在哪些域中。
6. 如果发现页面过期、矛盾或缺失，明确说出，不要假装知识库已经覆盖。
7. 若产生了可复用的新综合结论，询问是否写回相应域。
8. 如果问题属于消费评测集，记录本次回答是否更准确、更贴近用户上下文、是否可追溯到 wiki 页面。

## 工作流：Lint

当用户要求检查、整理、持续完善或质量审计时，由 wiki skill 根据当前 vault 的实际结构执行只读检查，验收范围包括：

1. 结构：知识域与基础设施目录是否符合根协议和相关域协议。
2. Frontmatter：是否缺 `type/title/created/updated`；月度 log 不要求 `updated`。
3. Index：重要页面是否都出现在域级索引中。
4. Log：日志文件是否只有一个 frontmatter，是否严格追加，operation 是否合法。
5. 链接：是否存在明显断链、重复页面、近义页面。
6. 来源：重要结论是否能追溯到 source 或明确标注为推断。
7. Overview：是否反映当前知识库主线。
8. 跨域一致性：同一概念在不同域的引用是否一致。

Lint 默认只报告问题；只有用户明确要求修复时，才直接修复低风险格式问题并列出需要用户判断的问题。

## 工作流：Update

更新已有页面时：

1. 保留原有有价值内容。
2. 优先追加"新证据 / 新关系 / 当前判断"，不要大段重写。
3. 重要结论变化要说明旧观点和新证据。
4. 更新 frontmatter 的 `updated`。
5. 如影响域级导航，更新相关索引。
6. 如影响知识库主线，更新 `tech/overview.md`。
7. 在相关日志中写更新记录。
8. **执行写后验证**（与 Ingest 步骤 15 相同：wikilink 扫描、索引覆盖、frontmatter 完整性、URL→wikilink 转换、知识表示一致性）。

## 工作流：Restructure

当用户要求新增知识域、删除/归档知识域、重命名知识域、调整目录边界、移动页面或新增域级子目录时，按结构变更处理。`sources/` 与 `logs/` 是基础设施层，不适用知识域必备文件规则。

1. 先判断结构变更是否必要：能放入现有域的，不为单个页面新增域。新增域前必须列出最相近的 1–3 个现有域，并说明扩展现有域为何不足。
2. 声明结构决策：`Create / Rename / Retire / Delete / Move / Skip + Reason`。
3. **协议影响扫描**：编辑前评估根协议、目标域、直接引用该域的协议、语义边界受影响的相邻域和内容接收域；逐项记录 `Create / Update / Skip + Reason`。不要求修改所有域，但所有可能受影响的域都必须经过评估。
4. 结构变更必须保持根协议、受影响的域协议、导航和日志同步，不允许只改目录。边界变化时必须同时更新相邻域的反向边界和路由示例；未受影响的域不得为了形式一致而修改。
5. 新增顶级知识域时至少创建：
   - `<domain>/CLAUDE.md`：域级规则、页面路由、索引职责、分享默认值，并明确收什么、不收什么、相邻域冲突规则和典型路由示例。
   - `<domain>/AGENTS.md`：优先作为指向 `CLAUDE.md` 的软链；不支持软链时复制同内容并说明。
   - `<domain>/index.md`：`type: index` 的域级导航；空域写"当前暂无条目"。
6. 新增顶级域时必须同步更新：
   - 本文件的项目定位、域判断规则、目录边界。
   - 新域 `CLAUDE.md`：定义该域页面类型、局部字段、正文结构和专属 workflow。
   - 根 `index.md`。
   - `logs/YYYY-MM.md`。
7. 重命名或移动目录时，先用命令列出受影响文件，再同步更新 wikilink、索引、协议、日志；所有直接路径引用和语义边界引用都必须处理，旧路径不得残留在正文中，除非是日志或明确的历史说明。
8. 删除非空域前必须先迁移或归档内容。优先 `Retire`，只有空域或已完全迁移的域才允许 `Delete`。删除、归档或迁移后，内容接收域必须明确以后如何接收和路由原域内容。
9. 写后验证必须额外检查：
   - 必备协议文件和知识域 index 是否存在。
   - 已输出协议影响表，所有可能受影响的域都有 `Create / Update / Skip + Reason`。
   - 所有直接引用目标域的协议已处理，所有语义边界变化的相邻域已更新反向边界和路由示例。
   - 删除、归档或迁移时，内容接收域已定义迁移后的路由规则；标记为 `Skip` 的域确实没有边界变化。
   - 新域协议是否定义收什么、不收什么、相邻域边界，并至少包含一个应进入和一个不应进入的路由示例。
   - 已搜索相邻域，确认新增域没有产生重复页面；涉及迁移时原域与新域 index 均已同步。
   - 根 index 与 canonical index 是否覆盖新增/移动页面。
   - `rg` 搜索旧路径或旧域名是否仅出现在日志/历史说明中。
   - 本次改动文件的 wikilink 是否全部存在。

如果通过 wiki skill 操作结构变更，还应读取 skill 的 `references/wiki-structure.md` 作为执行 runbook；该 reference 只提供操作步骤，权威规则仍以本文件和相关域 `CLAUDE.md` 为准。

## 分享边界

`share_scope` 只约束对外输出，不限制个人 Query 的读取。全局取值见本文件「全局页面契约」，更严格的域默认值见对应域 `CLAUDE.md`。

对外输出场景（写公开文章、生成对外总结、导出/迁移页面、生成 issue/PR/README/公开文档）必须检查引用页的 `share_scope`：

- `public`：可直接引用。
- `internal`：可抽象使用，不直接暴露原文细节。
- `private`：需询问用户确认后才能外发。
- `sensitive`：默认不得外发，仅在用户明确指定具体内容时使用。

## 页面质量标准

一个合格页面应满足：

- 有合法 frontmatter。
- 标题、描述、类型清楚。
- 正文有结构化标题。
- 至少有一个向内或向外的知识链接，孤立 source 除外。
- 关键判断能追溯到来源。
- 使用中文解释，必要英文术语保留原文。

## 禁止行为

- 不要修改 `sources/raw/` 中已有的文件（只读层，只允许新建；坏抓取用新 raw + source 摘要页说明来纠正）。
- 不要把未经筛选的大段原文复制进知识域页面。
- 不要为同一概念创建多个近义页面。
- 不要在日志文件中间追加新的 frontmatter。
- 不要把 `sources/` 当成可编辑笔记本——`raw/` 完全只读；source 摘要页是加工层，可以更新 frontmatter、摘要、洞察、风险和影响页面，但必须追溯到 raw 或明确标注为推断。
- 不要只引用外部本机路径作为 source；外部资料进入 wiki 时应保存原文到 `sources/raw/` 并在 `sources/projects/` 保存快照，保证知识库可迁移。
- 不要只更新页面而忘记日志和必要的索引维护。新增、删除、移动、重命名页面必须更新对应 canonical index；仅改已有页面正文时必须验证索引覆盖；所有写操作都必须追加 `logs/YYYY-MM.md`。
- 不要为了追求完整而一次性重构全库；优先小步、可审计地改。
- 不要自动向 `ideas/` 写入内容（必须用户确认）。
- 不要在未声明路由决策的情况下直接操作某个域。
- 不要在正文中使用 `[text](/en/...)` 格式链接已有 wiki 页面的概念——必须用 Obsidian 双链 `[[page]]`。
- 不要对来源中作为同级概念讨论的条目给予不平等的页面粒度——如果为其中一个建了独立页，检查其余是否同样需要。判断标准是来源如何处理它们（对比表、并列列表、等长章节、枚举），而非 wiki 用什么格式。
- 不要在对外输出中忽略 `share_scope`；`sensitive` 默认不外发，`private` 需用户确认。

## Agent 主动行为

### 会话末 Ingest 提示

当一次对话中出现了以下信号，且内容尚未入库，AI 应在对话即将结束时主动提问：

> "这次对话产生了一些值得入库的内容，要不要 Ingest？"

触发信号：
- 用户分享了技术文章链接、论文、博客、视频、播客。
- 讨论中产生了明确的技术结论或方法论判断。
- 用户描述了一个可复用的工程实践或踩坑经验。
- 出现了值得记录的人物信息或关系变化。
- 对话涉及学习路径规划或进度变化。

不提示的情况：
- 对话仅涉及 debug、项目操作、与知识库无关的任务。
- 内容已在本次对话中被 Ingest。
- 内容明显属于临时性质，无长期价值。
- 用户在本次对话中已拒绝过同一内容的入库/沉淀建议。

### QA 捕获提示

当对话产生了一个有价值的问答结论（不是开放讨论，而是有明确答案），AI 应提问：

> "这个结论值得沉淀到 qa/ 吗？"

仅在结论清晰、可复用时触发，不要对每个问答都提示。

### Ingest 主动触发

当用户在对话中分享了以下内容，AI 应主动询问是否入库，而不是等用户显式说"入库"：

- 贴出文章全文或链接并讨论内容。
- 分享了新发现的工具、框架、方法。
- 讨论了一个新概念并形成了理解。

优先级：若当前对话已计划在会话末统一提示，则不在中途重复触发。仅在用户明确深入讨论某篇内容且短期内不会再产出更多可 Ingest 内容时，才在中途直接提问。

提问格式：

> "这篇内容 / 这个概念看起来值得入库到 [域名]，要 Ingest 吗？"

如果用户拒绝，不再追问。

## 输出要求

每次完成维护后，用中文简要说明：

- 做了什么。
- 改了哪些文件。
- 当前还剩什么结构性问题。
- 如何验证。

---
> Source: [Illuminated2020/agentic-wiki](https://github.com/Illuminated2020/agentic-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
