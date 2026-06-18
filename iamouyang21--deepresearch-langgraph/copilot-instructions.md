## agents

> Agent 构建与 Prompt 注入

# Agents 约定

- 统一在 [src/agents/agent.ts](mdc:src/agents/agent.ts) 通过 `createReactAgent` 构建。
- 使用 `getLLm()` 提供的 LLM 与工具组合。
- Agent 的 prompt 通过 [src/prompts/template.ts](mdc:src/prompts/template.ts) 注入变量（`{{ VAR }}` → `{VAR}`）。
- 典型签名：

```ts
export async function createAgent(name: string, tools: StructuredTool[], promptTemplate: string)
```

- 在图节点中调用时，确保将 `GraphAnnotation.State` 传入以便模板插值。

---
> Source: [iamouyang21/DeepResearch-Langgraph](https://github.com/iamouyang21/DeepResearch-Langgraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
