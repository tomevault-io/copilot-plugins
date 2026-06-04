## aiforge

> 执行 `generate-project-context` workflow 生成 `project-context.md` 后，必须确认文件包含 **Rule Document Registry** 章节。如果生成的文件缺少该章节，立即补充以下内容到文件顶部（frontmatter 之后、Technology Stack 之前）：

# ai-forge Project — AI Agent Instructions

## Rule Document Registry 同步规则

执行 `generate-project-context` workflow 生成 `project-context.md` 后，必须确认文件包含 **Rule Document Registry** 章节。如果生成的文件缺少该章节，立即补充以下内容到文件顶部（frontmatter 之后、Technology Stack 之前）：

```markdown
## Rule Document Registry

**凡是确认/修改/新增任何规则、约定或豁免，必须同步更新以下所有文档：**

| 文档 | 职责 |
|------|------|
| `_bmad-output/project-context.md` | AI agent 主规则文件，优化为 LLM 消费 |
| `_bmad-output/planning-artifacts/architecture/04-implementation-patterns.md` | 实现模式，面向人类可读 |
| `_bmad-output/planning-artifacts/architecture/03-core-decisions.md` | 技术决策事项记录，面向人类可读 |

> 两份文档内容互为镜像，任何一处规则变更必须同时更新另一处。
```

## 规则变更同步约束

任何 story 执行、code review、或临时决策中，凡确认/修改/新增规则边界（包括豁免、约定变更），必须在同一次操作中同步更新 Rule Document Registry 中列出的所有文档，不得遗漏。

## 文档归属规则

新增或移动文档时，按以下规则判断目标目录：

### docs/

| 目录 | 归属条件 |
|------|----------|
| `docs/` 根目录 | README「文档」章节直接链接的核心用户文档（双语 `.md` + `.zh.md`） |
| `docs/references/` | 早期草稿、调研报告、审核结论等已归档的过程性文档 |

### _bmad-output/

| 目录 | 归属条件 |
|------|----------|
| `analysis/` | 1-analysis 阶段产物：头脑风暴、市场调研、混沌工程等前置分析 |
| `planning-artifacts/` | 2-plan + 3-solutioning 阶段产物：PRD、架构设计、Epic/Story 规划 |
| `implementation-artifacts/` | 4-implementation 阶段产物（按下表细分子目录） |
| `archive/` | 已归档的早期快照 |
| 根目录 `project-context.md` | AI Agent 主规则文件（跨阶段持续更新） |

### _bmad-output/implementation-artifacts/

| 子目录 | 归属条件 |
|--------|----------|
| `stories/` | Story 规格文件 |
| `code-reviews/` | 跨 LLM Code Review 各轮次 summary/evaluation |
| `retrospectives/` | Epic 回顾、Story 审查总结、发布门禁 |
| `cr-rules/` | CR 工作流的规则提炼、TODO 追踪等元文档 |
| 根目录 | 跨子目录的全局跟踪文件（`sprint-status.yaml`、`index.md`） |

---
> Source: [flanliulf/aiforge](https://github.com/flanliulf/aiforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
