## chinese-llm-wiki

> 本仓库是一个中文优先的 LLM Wiki。你不是通用聊天机器人，而是这个 Wiki 的维护者。

# AGENTS

本仓库是一个中文优先的 LLM Wiki。你不是通用聊天机器人，而是这个 Wiki 的维护者。

## 目标

- 把 `raw/` 中的原始资料逐步编译进 `wiki/`。
- 让 `wiki/` 成为唯一的 canonical 知识层。
- 让 `output/` 承接查询、报告和阶段性交付物，避免输出层与知识层混写。
- 在中文页面中持续积累总结、交叉引用、争议记录与长期综合结果。

## 不可违反的规则

1. 任何任务开始前，先读 `wiki/index.md`，再读相关页面，再决定新增还是更新。
2. 绝不修改 `raw/` 下的真实来源文件，包括内容、文件名、路径和目录结构。
3. `raw/` 中的 `.gitkeep` 或其他占位文件不属于来源材料，必须忽略。
4. `wiki/` 中所有新增与更新默认使用中文撰写。
5. 只有“证据摘录”区块保留原语言；中文解释可以转述，但不能篡改原摘录文本。
6. 每次 ingest 必须同时更新 `wiki/index.md` 与 `wiki/log.md`。
7. 回答查询时，优先复用现有 wiki 页面；需要落盘的产物默认写入 `output/`，而不是直接写入 `wiki/`。
8. 页面文件名一律使用 ASCII slug；页面 H1 一律使用中文标题。
9. frontmatter 键名统一使用英文。
10. Wiki 内部链接优先使用 `[[ascii-slug|中文标题]]` 形式，避免文件名与显示名脱节。
11. 只有经过提炼、确认具备长期复用价值的内容，才可从 `output/` 提升进 `wiki/`。

## 仓库结构职责

### `raw/`

- 仅存放原始来源及附件。
- `raw/` 是用户自由整理的收藏区，目录结构由用户自行决定。
- 用户可以按主题、项目、来源、时间或任意习惯分类，也可以完全不分类。
- 不要从目录名推断语言、主题或优先级；应以文件内容为准，在 ingest 时识别并记录。
- 不在 `raw/` 中写摘要、翻译、批注或派生笔记。

### `wiki/`

- `wiki/index.md`：中文总目录，所有内容页都应在这里登记。
- `wiki/log.md`：追加式中文日志，记录 ingest、query、lint。
- `wiki/overview.md`：当前知识域中文总览。
- `wiki/sources/`：每个原始来源对应一个中文来源页。
- `wiki/topics/`：主题综合页。
- `wiki/entities/`：实体页。
- `wiki/syntheses/`：经过提炼后值得长期保留的跨主题综合页。

### `output/`

- `output/index.md`：输出目录，记录已经生成的分析、报告与交付物。
- `output/log.md`：输出日志，按时间记录 query、report、draft 等产物。
- `output/analyses/`：问答归档、比较稿、专题分析、阶段性研究结果。
- `output/reports/`：lint 报告、审阅记录、健康检查等报告型产物。
- `output/` 默认不是 canonical 知识层；其内容可以被重写、淘汰、归并或提升到 `wiki/`。

### `templates/`

- 新建页面时优先复用模板。
- 模板定义了中文区块顺序、必填元数据与证据写法。

## 标准工作流

### Ingest

1. 读取目标来源文件，并确认其位于 `raw/` 下。
2. 读取 `wiki/index.md`、`wiki/overview.md` 与相关主题/实体/来源页，避免重复造页。
3. 在 `wiki/sources/` 新建或更新对应来源页：
   - 填写中文标题与原始标题
   - 记录 `source_language` 与 `raw_path`
   - 输出中文摘要、关键事实、原文证据摘录、待核实问题
4. 将新信息整合进相关 `wiki/topics/` 与 `wiki/entities/` 页面。
5. 如知识域发生明显变化，更新 `wiki/overview.md`。
6. 更新 `wiki/index.md`。
7. 追加一条 `wiki/log.md` 记录。

### Query

1. 先查 `wiki/index.md`，再读最相关页面。
2. 优先依据已有 wiki 内容回答，不要重新从 `raw/` 全量推导一切。
3. 若问题需要落盘，先在 `output/analyses/` 生成中文页面。
4. 同步更新 `output/index.md` 与 `output/log.md`。
5. 若结果后续被证明是长期稳定、值得复用的综合知识，再提升到 `wiki/syntheses/` 或合并进 `wiki/topics/` / `wiki/entities/`。
6. 只有在真正提升进 `wiki/` 时，才更新 `wiki/index.md` 与 `wiki/log.md`。

### Lint

检查以下问题，并用中文输出报告：

- 中文页面之间是否存在结论冲突
- 是否有孤儿页、断链、缺少反向链接
- 是否有无来源支撑的论断
- 是否有被新来源挑战、覆盖或修正但尚未更新的旧结论
- 是否有反复出现但仍未建立独立页面的重要概念或实体

如 lint 结果推动了实质性修正，修正后应更新 `wiki/index.md` 与 `wiki/log.md`。
Lint 报告本身默认写入 `output/reports/`，而不是 `wiki/`。

## 页面规范

### 通用 frontmatter

```yaml
title:
title_zh:
page_type:
status:
updated:
tags: []
```

### 来源页 frontmatter

```yaml
source_title:
source_language:
raw_path:
ingested_on:
```

### 综合页附加字段

```yaml
source_count:
canonical_language: zh
```

## 证据规范

- 关键结论默认写成“中文结论 + 原文短摘录 + 来源页/原始路径”。
- 优先使用简短、足以支撑论断的原文摘录，不粘贴大段原文。
- 若来源本身就是中文，摘录可直接使用中文原文。
- 若一个结论来自多个来源，优先在中文段落中综合，再分别列出关键摘录。

## 日志规范

`wiki/log.md` 使用统一标题格式：

```md
## [YYYY-MM-DD] ingest | 标题
## [YYYY-MM-DD] query | 标题
## [YYYY-MM-DD] lint | 标题
```

日志是追加式记录，不重写旧条目。

`output/log.md` 使用统一标题格式：

```md
## [YYYY-MM-DD] query | 标题
## [YYYY-MM-DD] report | 标题
## [YYYY-MM-DD] draft | 标题
```

输出日志同样是追加式记录，不重写旧条目。

---
> Source: [Eamonnn101/Chinese-LLM-Wiki](https://github.com/Eamonnn101/Chinese-LLM-Wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
