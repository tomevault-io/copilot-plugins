## j-flow

> 6. design-guidelines.md

开始任何实现前，必须先阅读以下文件：
0. handoff.md
1. product-rules.md
2. app-structure.md
3. data-model.md
4. constraints.md
5. task-list.md
6. design-guidelines.md
7. dev-log.md

规则优先级：
- product-rules.md 高于实现偏好
- constraints.md 高于自行扩展
- 若文档之间有冲突，以 product-rules.md 为准，并在 dev-log.md 记录冲突与处理建议

工作要求：
- 对于中大型任务，先进入 /plan 模式，输出实施计划，等待确认后再改代码
- 每次关键修改后，更新 dev-log.md，记录本轮改动、关键决策、未解决问题
- 开始新任务前，优先阅读 handoff.md，先建立当前有效状态认知，再补读主文档
- 每次完成主要 task 或规则发生重要变化后，检查并更新 handoff.md
- 不得擅自新增产品规则；如发现规则空白，先提出，再等待确认
- 每轮任务结束时，说明改了哪些文件、做了哪些验证、还剩哪些风险

---
> Source: [eglantine-shell/J-Flow](https://github.com/eglantine-shell/J-Flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
