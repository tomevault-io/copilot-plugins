## wait-agent

> 每次会话启动，必须先读取 `.agents/index.yaml`，然后按其中的 `read_order` 加载上下文。

# WaitAgent

每次会话启动，必须先读取 `.agents/index.yaml`，然后按其中的 `read_order` 加载上下文。

所有执行规则、当前任务、约束、原语、工作流都在 `.agents/` 中定义。这是唯一的机器控制平面。

不维护人类文档（`docs/`），除非用户明确要求。

---
> Source: [kikakkz/wait-agent](https://github.com/kikakkz/wait-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
