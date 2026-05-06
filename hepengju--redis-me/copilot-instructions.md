## frontend-simplicity-and-reuse

> 适用于所有新增功能代码，请默认遵守以下规则：


# 前端开发约束（功能新增）

适用于所有新增功能代码，请默认遵守以下规则：

1. **代码简洁优先**：实现以清晰、可读、低复杂度为目标，避免过度设计和不必要抽象。
2. **HTML 结构尽可能简单**：模板层级保持精简，避免无意义嵌套。
3. **样式尽可能简单**：优先复用现有样式能力，避免重复和复杂选择器。
4. **优先复用组件**：
   - 有可用的 ElementPlus 组件时，优先使用
   - 项目内已有可复用组件时，优先复用，不重复造轮子。

如确实无法复用，需要在实现说明中简要写明原因。

---
> Source: [hepengju/redis-me](https://github.com/hepengju/redis-me) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
