## dead-code-cleanup

> 发现无意义代码时直接删除


# 无意义代码清理

在修改或审查代码过程中，若发现以下情况，立即删除并记录：

- 未被引用/调用的函数、类、变量。
- 重复实现或已废弃的逻辑（参考 code-design-plan.md 中的 guards/、events/ 设计）。
- 空分支、永不执行的条件、注释掉的死代码。

**BAD 示例**：保留未使用的 import 或 TODO 注释的废弃函数。

**GOOD 示例**：
- 分析 core/agents/tools/executor.py 时发现旧版 action 未在 specs.py 注册 → 删除对应方法。
- 删除 tests/ 中已失效的 mock（验证后）。

删除后更新相关测试与文档。优先保持代码库精简。

---
> Source: [GodyuFF/SuperVideoGenerator](https://github.com/GodyuFF/SuperVideoGenerator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
