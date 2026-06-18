## langgraph

> LangGraph 图构建与节点约束

# LangGraph 约定

- 图在 [src/graph/builder.ts](mdc:src/graph/builder.ts) 中通过 `StateGraph` 构建。
- 为返回 `Command` 的节点使用 `addNode(name, fn, { ends: [...] })` 明确可能跳转的终点。
- 使用 `START`/`END`，或 `addConditionalEdges` 管理分支逻辑。
- 新增节点时：
  1. 在 [src/graph/nodes.ts](mdc:src/graph/nodes.ts) 中实现；
  2. 在 `builder.ts` 注册并配置 `ends`/边；
  3. 如返回 `Command`，确保 `goto` 值在 `ends` 中；
  4. 必要时更新 [src/graph/state.ts](mdc:src/graph/state.ts) 的 `GraphAnnotation`。

## Prompt 与消息
- Prompt 文件位于 [src/prompts/*.md](mdc:src/prompts)。
- 通过 [src/prompts/template.ts](mdc:src/prompts/template.ts) 使用 `ChatPromptTemplate` 注入变量（`{{ VAR }}` → `{VAR}`）。

---
> Source: [iamouyang21/DeepResearch-Langgraph](https://github.com/iamouyang21/DeepResearch-Langgraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
