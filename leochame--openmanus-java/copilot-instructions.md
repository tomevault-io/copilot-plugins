## openmanus-java

> 我们构建了 `aiframework` 作为项目中的 Agent 开发框架。

我们构建了 `aiframework` 作为项目中的 Agent 开发框架。

本文档是仓库内协作与开发的总纲。

## 1. 协作原则

- `AGENTS.md` 是总纲，优先级高于普通说明文档。
- 所有角色都要遵守“小步推进、逐步完成”的节奏。

## 2. 代码要求

1. 代码要求精简，不要过度抽象。
2. 在使用设计模式的时候注意使用设计模式。
3. 注意代码层级关系。

进一步约束如下：

- 优先在现有结构中演进，不为了“未来可能扩展”提前引入复杂框架。
- Domain、Agent、Infra、aiframework 之间职责要清晰，避免双向渗透。
- 新增能力优先通过清晰接口接入，不直接把实验逻辑散落到 Controller 或配置层。
- 上下文管理、CodeAct、MCP 这类核心能力，必须先有最小可运行链路，再逐步增强。

## 3. 文档索引

本文件用于定义仓库级总纲、通用约束和文档关系。

当前阶段文档如下：

- `docs/DEVELOPMENT_GOALS.md`：记录开发目标。
- `docs/TECHNICAL_IMPLEMENTATION.md`：记录技术方案。
- `docs/DEVELOPMENT_PROGRESS.md`：记录开发进度。
- `docs/Review.md`：记录 review 意见。

除 `AGENTS.md` 与 `docs/DEVELOPMENT_GOALS.md` 外，其余文档默认在流程推进中逐步补充。

文档发生冲突时，按以下顺序判断：

1. `AGENTS.md`
2. `docs/DEVELOPMENT_GOALS.md`
3. `docs/TECHNICAL_IMPLEMENTATION.md`
4. `docs/Review.md`
5. `docs/DEVELOPMENT_PROGRESS.md`

文档可以进行更新，缩减过期信息。

---
> Source: [leochame/OpenManus-Java](https://github.com/leochame/OpenManus-Java) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
