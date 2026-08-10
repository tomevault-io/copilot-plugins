## seam-audit

> 排查「接缝」bug（前后端/事件 producer↔consumer/字段·路由·DB 列管线是否真接通）时读——两类接缝签名 + 防误报铁律 + 可复用并行探子配方。做接缝/漂移/gap 排查前读。


# 接缝排查配方（interface-seam audit）

排查「一侧定义了、另一侧没接」的接缝 bug。承 2026-07 白板内（WB-006/007）→ 白板外两批（FE-001…013）审计沉淀；逐轮结论正文见 git 历史退役的「项目审计」§十一之五（原文曾在维护者本地规划目录，不在公开树）。

## 两类接缝签名（判据）

1. **渲染画了却没接**（drawn-but-not-wired）＝一侧有 UI/控件/动词/handler，却没把输出真正喂到另一侧、或输出被静默丢弃。
   - 例：多选 scale 手柄画了不 hit-test（WB-006）；`board_ops` 的 `ref` 目标被 applier 丢（FE-008）。
2. **已铺管线·零产出**（plumbed-but-zero-producer/consumer）＝字段/事件/路由/DB 列/类型在一侧已定义并承载，另一侧却无 producer（无人写/发）或无 consumer（无人读/渲）。
   - 例：rotation 全链认却无 producer（WB-007）；`run_id` 后端发前端丢（FE-003）；辩论 `BatchMetrics` 收了不 emit（FE-009）。

## 铁律：判 gap 前先查文档（防误报）

判「有后端无前端 / 有前端无后端」为 gap **之前，先 grep `docs/`** 看是否记为**有意休眠 / 退役 / ⏳待后端**——大量「缺口」实为 by-design（质量档 UI 内测退役、admin LLM 窗待后端、mobile lite 减法…，见 FE-004 翻案）。

**弱证据不冒充 bug**，置信五档只对前两档动手：
`GAP`（真缺口·修）/ `DRIFT`（漂移·修）/ `RESERVED`（前瞻预留 → `远期规划`）/ `BY-DESIGN`（有意 → 记档闭案）/ `UNSURE`（留观察）。

## 可复用配方

1. 按面切分（画板/分享/admin/记忆/…）派**并行只读探子**，每子代理带「两签名 + 本铁律 + 五档置信 + 必附 `file:line` 证据」（子代理必须用 `cursor-grok-4.5-high` 模型）。与讨论并行时的主/子分相见 `multitask.mdc`「讨论 + 审计分相」。
2. 探子回报的 GAP **先 Read 源码复核**——别信行号（FE-004 正是复核翻的案）；复核以外禁止主侧重跑整审计。
3. 复核为实的 GAP·DRIFT 才修，改完 `vitest`/`pytest` + `biome`/`ruff` + `tsc` 自检。
4. RESERVED 进 `远期规划`、BY-DESIGN 记档闭案、UNSURE 留观察。

## 收口判据

每批稳定 ~2 真 bug、二批后边际递减（多为 by-design）即收口，别为凑数造 busywork。

---
> Source: [Lawofall/AgentCore](https://github.com/Lawofall/AgentCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
