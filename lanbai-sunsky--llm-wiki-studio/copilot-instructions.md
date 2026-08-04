## llm-wiki-studio

> 你在这个仓库里工作时，默认把它当成一个本地课程资料控制台。

# LLM Wiki Studio

你在这个仓库里工作时，默认把它当成一个本地课程资料控制台。

- `原始资料/` 是事实来源。
- `已创建的Wiki/` 是对话学习记忆层，不是需要单独打分验收的最终成品。
- `复习资料/` 是考试输出层。
- 每轮学习对话结束后，如有新的理解、例子、疑问、易错点或用户确认的结论，优先使用 `.claude/skills/learning-memory-save/SKILL.md` 保存到 Wiki。
- 生成复习资料时，优先使用 `.claude/skills/review-materials/SKILL.md`；输入优先级是用户上传的重点标注文件，其次是 `原始资料/`，Wiki 只是可选补充。
- 不要因为 Wiki 为空、只有骨架或缺少正文就拒绝生成复习资料。
- 每次只生成一个本次输出文件夹，里面固定包含 `复习资料.md` 和 `评测报告.md`。
- `复习资料.md` 的每一行只能使用 `知识点名称~必考` 或 `知识点名称~有可能考`，不要生成提纲、问答清单、表格或额外说明文件。
- 网页端不再提供独立的“生成复习资料”按钮；用户会在 Agent 对话界面中自然语言触发 `review-materials` skill，底层仍由 Claude Code CLI 执行。
- 不要围绕 Wiki 质量检查、语义评分或旧 Agent 工作流展开；重点调试和使用 Skills。
- 不要再依赖旧的 LangGraph Agent 工作流。
- 需要写文件时，优先在当前工作区内完成，并保留现有目录结构。

---
> Source: [lanbai-sunsky/llm-wiki-studio](https://github.com/lanbai-sunsky/llm-wiki-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
