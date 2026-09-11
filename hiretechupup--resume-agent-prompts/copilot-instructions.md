## progress

> 本规范与 @changlogs.mdc 配套，要求 Cursor 在交付任何影响项目进度的改动时同步更新以下内容：

本规范与 @changlogs.mdc 配套，要求 Cursor 在交付任何影响项目进度的改动时同步更新以下内容：

1. `README.md` 中的「路线图」章节必须真实反映项目各阶段的完成度，并列出当前仍待完成的重点事项（测试、扩展功能、稳定性等）。若交付会调整阶段内容，需在同一提交中更新该表。
2. 项目主计划文档（通常位于 `docs/` 目录下）是进度单一事实来源。实现新能力或完成阶段任务时，要勾选对应清单、更新时间戳，并将新的 backlog / 风险写入「当前待办」或风险段落。
3. 额外的大功能文档用于记录具体交付细节；当其 TODO 有状态变化时，要同步回主计划文档与 README，避免多处信息不一致。
4. 如新增任务或未完成事项，请在上述文档中创建/更新待办条目，并在提交中记录到 `CHANGELOG.MD`（遵循 @changlogs.mdc）。

---
> Source: [HireTechUpUp/resume-agent-prompts](https://github.com/HireTechUpUp/resume-agent-prompts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-11 -->
