## doc-sync

> 修改后确认文档一致性并更新文档信息


# 文档同步与更新

代码修改完成后，必须：

1. 比对实际代码与 docs/ 描述是否一致（例如 prompt 固定区文件、GenerationMode、实体模型）。
2. 更新文档：修改日期、版本、对应章节内容。
3. 若新增功能，补充到 product-plan.md 或 code-design-plan.md。

**BAD 示例**：修改了 `core/llm/master_react.py` 的 ReAct 逻辑，未更新 prompt-architecture.md 中的动态槽位表。

**GOOD 示例**：
- 修改后立即编辑 docs/superpowers/reference/prompt-architecture.md，更新“4. 动态槽位”表格与日期。
- 提交前运行 grep 确认 docs/ 提及新 action。

所有 PR/变更必须包含文档更新。

---
> Source: [GodyuFF/SuperVideoGenerator](https://github.com/GodyuFF/SuperVideoGenerator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
