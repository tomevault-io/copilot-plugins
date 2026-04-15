## atomic-ai

> 当生成或修改代码注释时，请严格遵守以下黄金法则：


当生成或修改代码注释时，请严格遵守以下黄金法则：

1.  **【核心原则】注释必须解释“为什么”（Why），而不是“做什么”（What）**。
    *   代码本身应该通过清晰的命名和结构来解释“做什么”。
    *   注释应该提供代码无法直接表达的背景信息、设计决策和意图。
    *   **错误示例**：`// 循环遍历用户列表` (这是代码本身在做的事)
    *   **正确示例**：`// 某些用户的账户可能处于非活跃状态，需要在此处进行筛选，以避免向他们发送通知。` (这解释了为什么需要筛选)
2.  **【避免时态性描述】注释必须是“永恒的”，避免描述当前的修改行为**。
    *   绝对不要使用“本次更新”、“新增了”、“修改了”、“修复了”等词语。这些信息属于版本控制的提交信息（commit message），而不是代码注释。
    *   注释应该描述代码在当前状态下的行为和目的，无论何时阅读，其含义都应保持不变。
    *   **错误示例**：`// 新增了对新版 API 格式的支持`
    *   **正确示例**：`// 为了同时兼容 v1 和 v2 版本的 API 响应，此处进行了条件判断。`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HJSunDev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/HJSunDev)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
