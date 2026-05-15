## knowledgebase

> 这是一个 LLM 驱动的个人知识库系统。核心理念是"编译器"隐喻：

# KnowledgeBase — Claude Code 项目约定

## 项目概述

这是一个 LLM 驱动的个人知识库系统。核心理念是"编译器"隐喻：
- `raw/` = 源代码（人工输入的原始素材）
- `wiki/` = 编译产物（LLM 生成和维护）
- `output/` = 派生输出（幻灯片、图表、报告）

**重要：wiki/ 下的所有内容由 LLM 写和维护，人类通常不直接编辑。**

## 语言约定

- 文件名：小写英文 + 连字符（如 `attention-mechanism.md`）
- 文件内容：中文
- 代码注释：英文
- YAML frontmatter 字段名：英文

## 目录结构

```
raw/           原始素材（人工输入）
  _registry.md   素材注册表
  articles/      网络文章
  papers/        学术论文
  books/         书籍
  repos/         代码仓库
  datasets/      数据集
  images/        图片素材
  videos/        视频笔记
  misc/          其他

wiki/          LLM 编译的知识 wiki
  _index.md      总索引（每次编译后更新）
  _graph.md      概念关系图谱
  _changelog.md  编译变更日志
  concepts/      原子概念条目
  topics/        主题综述
  guides/        操作指南
  comparisons/   对比分析
  timelines/     时间线
  glossary/      术语表

output/        派生输出
  slides/        Marp 幻灯片
  charts/        matplotlib 图表
  reports/       综合报告
  exports/       PDF/HTML 导出

templates/     模板
  prompts/       LLM prompt 模板
  content/       内容格式模板

tools/         Python CLI 工具
media/         Wiki 引用的媒体资源
```

## 命名规范

### raw/ 素材命名
- articles: `YYYY-MM-DD-slug.md`（如 `2026-04-01-attention-is-all-you-need.md`）
- papers: `author-YYYY-short-title.md`（如 `vaswani-2017-attention.md`）
- books: 每本书一个子目录 `book-title/`，内含 `_meta.md` 和 `chapter-NN.md`
- repos: `repo-name.md`
- datasets: `dataset-name.md`

### wiki/ 条目命名
- 概念名用小写连字符：`transformer.md`、`attention-mechanism.md`
- 对比用 `a-vs-b.md` 格式
- 每个子目录有 `_index.md` 作为分类索引

## Wiki 条目格式

所有 wiki 条目必须包含 YAML frontmatter：

```yaml
---
title: 条目标题
aliases: [别名1, 别名2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources:
  - raw/papers/xxx.md
  - raw/articles/xxx.md
tags: [tag1, tag2]
---
```

条目结构：
1. `## 概述` — 一段话摘要
2. `## 核心内容` — 详细解释（根据条目类型灵活调整小节）
3. `## 关联` — 使用 `[[backlinks]]` 链接相关条目
4. `## 参考来源` — 链接回 raw/ 素材

## 编译规则

1. 每次编译后必须更新 `wiki/_index.md` 和 `raw/_registry.md`
2. 所有 wiki 条目之间使用 `[[wikilinks]]` 互相链接
3. 新概念必须检查是否已有相关条目，避免重复
4. `_changelog.md` 记录每次编译的变化摘要，格式: `## [YYYY-MM-DD] operation | Subject`
5. 图片资源放在 `media/images/`，用相对路径引用

## Wiki 分类策略

Wiki 有 6 个子目录：concepts/、topics/、guides/、comparisons/、timelines/、glossary/。在知识库规模较小时（<50 个条目），遵循以下简化策略：

- **默认使用 `concepts/`**：除非内容明显属于其他类别，所有条目都放在 concepts/
- **只在明确场景使用其他类别**：
  - `comparisons/` — 仅当内容核心是两个概念的对比
  - `guides/` — 仅当内容是步骤式操作指南
  - `topics/` — 仅当一个主题已积累 3 个以上相关 concepts/ 条目时，才值得创建独立的 topic 综述
  - `timelines/` — 仅当时间维度是核心组织逻辑
  - `glossary/` — 由 /lint 自动维护
- **索引更新原则**：只更新有变化的分类索引（如只新增了 concepts/ 条目，不需要更新 topics/_index.md）

## 素材注册表 (raw/_registry.md)

追踪每个素材的编译状态。格式：

```markdown
| 文件 | 类型 | 导入时间 | 状态 | 关联条目 |
|------|------|----------|------|----------|
```

状态：`pending`（待编译）、`compiled`（已编译）、`updated`（素材已更新需重新编译）

## 工具

- `tools/search.py` — 全文搜索 wiki
- `tools/stats.py` — wiki 统计
- `tools/validate.py` — 链接完整性检查
- `tools/graph_export.py` — 知识图谱导出

---
> Source: [freeman5860/KnowledgeBase](https://github.com/freeman5860/KnowledgeBase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
