## cursor

> 阶段模板快捷入口（生成与接力）


- 生成用户故事：使用 `prompts/stages/user_story.md`，产出到 `docs/USER_STORIES.md`，并按 `handover_schema.md` 输出交接 JSON（next_role=PM）
- 生成 PRD：使用 `prompts/stages/prd.md`，产出到 `docs/PRD.md`，并输出交接 JSON（next_role=PjM）
- 任务分解：使用 `prompts/stages/task_breakdown.md`，产出到 `docs/TASKS.md`，并输出交接 JSON（next_role=Arch）
- 技术设计：使用 `prompts/stages/tech_design.md`，产出到 `docs/TECH_DESIGN.md`，并输出交接 JSON（next_role=Dev）
- 最小实现：使用 `prompts/stages/implementation.md`，产出代码与测试，输出交接 JSON（next_role=QA）
- QA 测试：使用 `prompts/stages/qa_test.md`，产出测试报告与覆盖率，交接 JSON（next_role=Ops）
- 迭代发布：使用 `prompts/stages/iteration.md`，产出 CHANGELOG/发布说明/回归清单（next_role=PM/PO）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Poghappy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
