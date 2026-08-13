## rules-maintenance

> rules 按 glob 匹配当前文件带入；新约定可建议写入 .cursor/rules


# Rules 维护

- 各 rule 按 **globs** 与当前打开/编辑路径匹配后带入；无 glob 或 alwaysApply: true 则始终生效。
- 对话中新确立项目级约定（布局、命名、API 风格、安全等）时，可建议加入 `.cursor/rules` 并通知用户，或拟好内容供确认。

---
> Source: [supplynexus/fulfillment-service](https://github.com/supplynexus/fulfillment-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
