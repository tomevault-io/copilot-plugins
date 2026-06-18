## llm

> LLM 提供与模型选择

# LLM 约定

- 统一从 [src/llms/llm.ts](mdc:src/llms/llm.ts) 暴露 `getLLm(temperature?: number)`。
- 默认使用 `ChatDeepSeek`，读取环境变量 `DEEP_SEEK_API_KEY`。
- 如需切换模型或供应商，在该文件中集中管理。

---
> Source: [iamouyang21/DeepResearch-Langgraph](https://github.com/iamouyang21/DeepResearch-Langgraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
