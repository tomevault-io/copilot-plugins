## recallnest

> - **运行时：Bun**。禁止使用 `npm`、`yarn`、`pnpm`，所有命令走 `bun`

# RecallNest 项目规则

## 1. 技术栈约束（强制）

- **运行时：Bun**。禁止使用 `npm`、`yarn`、`pnpm`，所有命令走 `bun`
- **语言：TypeScript strict**。禁止 `any` 类型，复杂接口提取到独立类型文件
- **向量存储：LanceDB**。不引入其他向量数据库
- **嵌入模型：Jina v5**。不替换嵌入方案
- **MCP 协议：`@modelcontextprotocol/sdk`**。MCP tools 统一走 `registerTool()`

## 2. 编码规范

- 新增 MCP tool 必须同时写测试（`src/__tests__/`）
- 禁止生成带 `// TODO` 或 `// placeholder` 的半成品代码——缺信息就停下来问
- 修改已有 tool 的 schema 前，先确认调用方兼容性
- 脚本放 `scripts/`，不在 `src/` 之外创建业务 `.ts` 文件

## 3. Agent 行为准则

- **写代码后必须自己跑测试**：执行 `bun test`，读报错，自主修复，直到全绿才交付
- 实现新功能前，先输出分步计划并等待确认，再进入执行阶段
- 任务颗粒度过大时，主动拆分子任务逐步推进，不要一口气撸完然后崩溃
- 禁止用未验证的 `git status` 结果写入 checkpoint

## 4. Git Push 规则（重要！）

- **所有 push 只推 origin**（`AliceLJY/recallnest`）
- **绝对不要 push 到 upstream**（`CortexReach/memory-lancedb-pro`）—— 那是上游公开仓库，推了等于暴露私有改造
- `trihippo/recallnest` 已停止维护，不再推送
- 默认 `git push` 即可（默认推 origin）
- 需要给上游提 PR 时，走 fork + PR 流程，不直接 push

## 5. Feature Flag

- `RECALLNEST_MULTI_VECTOR=true` — 多向量 L0/L1/L2 检索
- `RECALLNEST_KG_MODE=true` — KG 三元组提取 + 图遍历
- `RECALLNEST_EMOTION_SCORING=true` — Emotion detection + salience-weighted Weibull decay + arousal boost + retrieval scoring
- `RECALLNEST_CONSTRUCTIVE_RETRIEVAL=true` — Multi-source candidate expansion + source-map grounded reconstruction (resume default, search opt-in)
- `RECALLNEST_NARRATIVE_MODE=true` — Autobiographical narrative metadata layer (life-period / general-event / specific-event)
- `RECALLNEST_PREDICTIVE_MEMORY=true` — Heuristic-predicted prospective reminders (zero LLM, behavioral signals)

## 6. 测试基线

- 改完代码必须跑 `bun test`，全量通过才能 commit
- 当前基线：1692 tests / 0 fail
- 新增功能必须配套测试，基线只能涨不能降

---
> Source: [AliceLJY/recallnest](https://github.com/AliceLJY/recallnest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
