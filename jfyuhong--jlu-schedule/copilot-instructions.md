## jlu-schedule

> 1. **允许并在每次对话/任务阶段结束时进行小粒度提交 (Micro-commits)**：

# 项目开发与 Git 提交规范

## Git 提交约束 (Git Commit Guidelines)
1. **允许并在每次对话/任务阶段结束时进行小粒度提交 (Micro-commits)**：
   - 每次对话完成一个明确的功能点、缺陷修复、配置更新或代码重构，且通过本地验证后，允许并推荐进行及时的 Git 提交。
   - 避免长时间在工作区堆积大量未提交修改，防止因 IDE 缓存冲突、编辑重载或会话重启导致未持久化的代码丢失。
2. **提交质量要求**：
   - 提交前确保相关单元测试通过（`./gradlew testDebugUnitTest`）并无编译中断。
   - 提交信息遵循清晰语义（如 `feat:`, `fix:`, `refactor:`, `test:`, `docs:` 等），概括本轮对话完成的实质性改进。

---
> Source: [JFyuhong/JLU_schedule](https://github.com/JFyuhong/JLU_schedule) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
