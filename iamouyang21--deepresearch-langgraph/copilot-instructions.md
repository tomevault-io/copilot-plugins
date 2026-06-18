## project-structure

> - 运行入口（LangGraph 开发服务器）：`yarn start` 调用 `@langchain/langgraph-cli`，配置来源于 [langgraph.json](mdc:langgraph.json)。

# 项目结构指南

- 运行入口（LangGraph 开发服务器）：`yarn start` 调用 `@langchain/langgraph-cli`，配置来源于 [langgraph.json](mdc:langgraph.json)。
- Graph 注册点：`deep_research` 对应入口 `[src/workflow.ts](mdc:src/workflow.ts):graph`。
- 图构建：见 [src/graph/builder.ts](mdc:src/graph/builder.ts)。
- 节点实现：见 [src/graph/nodes.ts](mdc:src/graph/nodes.ts)。
- 状态定义：见 [src/graph/state.ts](mdc:src/graph/state.ts)。
- Prompt 模板：见 [src/prompts/*.md](mdc:src/prompts) 与填充逻辑 [src/prompts/template.ts](mdc:src/prompts/template.ts)。
- Agent/工具：见 [src/agents/agent.ts](mdc:src/agents/agent.ts)、[src/tools](mdc:src/tools)。

注意：TypeScript 源码位于 `src/`，构建产物输出到 `dist/`（已在 `.gitignore` 中忽略）。

---
> Source: [iamouyang21/DeepResearch-Langgraph](https://github.com/iamouyang21/DeepResearch-Langgraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
