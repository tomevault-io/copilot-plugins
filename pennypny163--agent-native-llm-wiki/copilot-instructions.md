## agent-native-llm-wiki

> 把上游资料编译为结构清晰、来源可追溯、可持续更新的知识库。不要把

# 知识库 Agent 操作说明

## 目标

把上游资料编译为结构清晰、来源可追溯、可持续更新的知识库。不要把
`sources/imported/` 当作最终答案；它只是证据和上下文。

## 阅读顺序

1. 从 `maps/` 或 `learning-paths/` 定位主题。
2. 优先阅读 `status: canonical` 的页面。
3. 需要核查时，根据页面的 `sources` 查阅 `sources/imported/`。
4. 涉及易变事实时检查 `last_verified` 和 `freshness`。

## 写入规则

- 不直接编辑根目录的 DOCX 或 `sources/imported/`。
- 新知识优先补充现有 canonical 页面，避免创建同义页面。
- 新页面必须包含 YAML 元数据。
- 先根据 [内容模型](meta/content-model.md) 选择页面类型。
- 页面只承担一种主要用途：
  - `concept`：定义和边界；
  - `explanation`：原理及原因；
  - `how-to`：可执行步骤；
  - `reference`：参数、清单、对照；
  - `learning-path`：有顺序的学习路线；
  - `map`：主题导航。
  - `evergreen`：一个值得长期演化、可被新证据修正的核心判断。
- 引用格式使用 `source-id#章节编号`，例如
  `agent-from-zero#4.1`、`juliet-llm#4.3`。
- 不把来源中的说法自动升级为事实。来源冲突时记录在 `evidence/conflicts.md`。
- 代码示例必须注明是否经过运行验证。

## 页面元数据

```yaml
---
title: 页面标题
aliases: []
type: concept
domain: agent
status: canonical
sources:
  - agent-from-zero#1.2
last_verified: 2026-06-25
freshness: low
confidence: medium
prerequisites: []
related: []
---
```

`freshness`：

- `low`：基础原理，通常不随时间变化；
- `medium`：工程方法，可能被新实践修正；
- `high`：模型、框架、API、价格、排行榜等易变信息。

## 更新流程

1. 执行 `make apply_updates` 更新来源层并生成影响报告。
2. 查看 `meta/update-report.md`。
3. 把新增或修订内容应用到 canonical / evergreen 页面。
4. 必要时更新主题地图、别名和冲突记录。
5. 执行 `make verify_wiki`。
6. 在 `meta/changelog.md` 记录有知识意义的变更。

---
> Source: [pennypny163/agent-native-llm-wiki](https://github.com/pennypny163/agent-native-llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
