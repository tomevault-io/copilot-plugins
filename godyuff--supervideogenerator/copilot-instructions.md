## doc-driven-dev

> 文档驱动的代码修改流程，强制修改前按文档分析规划


# 文档驱动开发

每次进行代码修改前，必须执行以下流程：

1. 使用 Read 工具阅读 docs/ 及 core/prompt/ 下相关文档（prompt-architecture.md、code-design-plan.md、product-plan.md）。
2. 分析变更是否符合文档设计（如 core/prompt 固定/动态分层、ReAct 主编排、A2UI 协议）。
3. 制定修改方案，列出受影响文件与文档更新点。

**BAD 示例**：直接编辑 core/agents/xxx.py，未阅读 docs/。

**GOOD 示例**：
- 先 Read docs/superpowers/reference/prompt-architecture.md 确认 fixed/actions.md 结构。
- 规划：新增 action 需同步更新 agents/*/fixed/actions.md 及 registry.py。

若修改涉及 prompt 层或编排逻辑，必须引用文档中的“两条 LLM 调用链”。

---
> Source: [GodyuFF/SuperVideoGenerator](https://github.com/GodyuFF/SuperVideoGenerator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
