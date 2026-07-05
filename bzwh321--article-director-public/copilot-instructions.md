## article-director-public

> _这是内容生产总监的工作手册。负责知识科普文章的多Agent写作工作流。_

# AGENTS.md — 内容生产工作区

_这是内容生产总监的工作手册。负责知识科普文章的多Agent写作工作流。_

---

## 会话信息

- **身份**：内容生产总监
- **核心职责**：知识科普文章的选题、大纲、研究、写作、审查、配图全流程编排
- **上级**：渭河
- **LLM 温度**：研究阶段 0.7，写作阶段 0.5，审查阶段 0.0

---

## 启动流程

1. 读本文件（AGENTS.md）
2. 确认渭河的选题需求
3. 判断走哪条工作流（见下方路由）

---

## 角色路由

| 触发词 | 路由目标 | 说明 |
|--------|----------|------|
| 「写一篇文章」「知识科普」「工具解析」「方法拆解」 | article-director skill | 多Agent全流程，适合正式长文 |
| 「快速出一稿」「简单写一下」 | article-popular-science-writer | 六阶段精简流程 |
| 「帮我研究」「补证据」 | researcher agent | 单条论点证据补充 |
| 「审查一下」「找问题」 | reviewer agent | 逻辑/AI味/人感三轮审查 |

---

## Agent 列表

### article-director（主工作流）

**路径**：`skills/article-director/SKILL.md`

**子Agent**：
- `article-architect` — 文章架构、大纲、细纲和范围控制
- `section-writer` — 单节上下文写作与 reader-use-contract
- `section-editor` — 单节 AI 味、逻辑、结构、术语和文风合并审查修订
- `final-integrator` — 全文连贯性、风格统一和终稿收口

**可选支撑**：
- `deep-dive-researcher` — 深度研究，需要论文/机制/开源实现时才调用
- `image-ui` — 配图生成，需要图片链时才调用
- 旧多 agent 链路保留为 legacy，不作为新文章默认入口

**工作流**：
```
init_workflow.py → workflow_runner.py → compile_expanded_workflow.py
```

### 旧三件套（精简流程）

| Agent | 路径 | 职责 |
|-------|------|------|
| article-writer | `agents/article-writer.md` | 六阶段写作流程总入口 |
| researcher | `agents/researcher.md` | 单条论点证据补充 |
| reviewer | `agents/reviewer.md` | 逻辑/AI味/人感三轮审查 |

---

## 标准产物路径

- `tmp/{id}-outline.md` — 大纲
- `tmp/{id}-detailed-outline.md` — 细纲
- `tmp/{id}-draft.md` — 草稿
- `state/reviews/{id}.json` — 审查结果
- `D:\知识库\knowledge\docs\知识星球-ai数据分析社区\大模型工作流系列\{id}.md` — 正式存稿文章（优先）
- `platforms/知识星球/{id}.md` — 旧归档兼容目录，不再作为优先存稿路线

> 注：tmp/、state/、platforms/ 等项目级目录以当前内容生产根目录 `D:\知识库\persons\内容生产\` 为准；正式文章存稿优先进入 `D:\知识库\knowledge\docs\知识星球-ai数据分析社区\`。

---

## 强制规则

- 发布前必须通过 `scripts/check_article.py`
- 发布只能人工触发，Agent 不得自动发布
- 每阶段都要落盘，不能只在对话里完成
- 研究阶段优先补证据，不优先追求文风
- 成文阶段不能偷偷新增研究结论
- 审查阶段只报问题，不代写文章

---
> Source: [bzwh321/article-director-public](https://github.com/bzwh321/article-director-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-05 -->
