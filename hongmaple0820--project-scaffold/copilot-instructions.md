## project-scaffold

> 本文件是 project-scaffold 的 Agent 通用入口。它面向所有由脚手架派生的项目，只写跨项目可复用的治理规则，不绑定具体业务框架。

# AGENTS.md

本文件是 project-scaffold 的 Agent 通用入口。它面向所有由脚手架派生的项目，只写跨项目可复用的治理规则，不绑定具体业务框架。

## 工作原则

- 先读现状，再改文件。不要把旧项目假设套到当前仓库。
- 可执行规则优先于口头建议：重要规则应该落到脚本、门禁、模板或检查命令。
- 小任务保持轻量；跨模块、安全、数据、生产配置和发布任务自动升级治理强度。
- 未运行验证不能声称通过；dry-run 只能证明可调度。
- 不确定事实标记 `[UNCERTAIN]`，并说明缺少什么证据。

## 推荐工作流

```text
探索 -> 规划 -> 执行 -> 验证 -> 沉淀
```

常用命令：

```bash
make new-task NAME=task-slug LEVEL=M
make explore FILES='AGENTS.md CLAUDE.md README.md' MSG='main contradiction'
make gate-workflow
make verify PROFILE=scaffold
make bootstrap-scale
make workflow-upgrade-check
make workflow-upgrade-plan
make workflow-aios-adopt
make verify-list
```

SCALE 最新工作流能力：

```bash
make scale-mode TASK='...' FILES='...'
make scale-radar TASK='...' PHASE=plan LEVEL=M FILES='...'
make scale-context
make scale-codegraph
make scale-eval
make scale-dashboard
```

SCALE 安装检查使用 `make bootstrap-scale`；工作流升级使用 `make workflow-upgrade-check` / `make workflow-upgrade-plan`，审阅计划后再执行 `make workflow-upgrade-apply`；需要刷新 AI OS runtime 证据时运行 `make workflow-aios-adopt`。

## 任务分级

| Level | 场景 | 要求 |
| --- | --- | --- |
| S | typo、注释、小范围文档 | 最小编辑加相关验证 |
| M | bug、小功能、脚本或规范优化 | 探索、计划、验证和总结 |
| L | 跨模块、架构、模板体系 | 完整计划，执行前确认 |
| CRITICAL | 数据、权限、安全、生产配置、破坏性操作 | 人工确认、回滚方案、安全评审、完整验证 |

## 工具选择

Agent 应主动选择合适工具，但必须保留证据：

- UI/UX：`frontend-design`、`ui-ux-pro-max`、awesome-design-md 思路。
- 浏览器/E2E：Playwright、Agent Browser、web-access、Chrome DevTools MCP。
- 桌面/端侧：CUA/computer-use，仅在边界明确且必要时使用。
- 外部 CLI：Codex、Claude Code、Gemini CLI、OpenCode 等必须记录版本、命令和输出摘要。
- 代码理解：优先 CodeGraph/Graphify；不可用时明确回退到 `rg` 和局部读取。

## Git 和协作

- 默认分支格式：`<type>/<human-author>/<agent-platform>-<scope>-<task>-<MMDD>`。
- 开始前看 `git status --short --branch`。
- 不覆盖人类未提交改动，不顺手清理无关文件。
- 不使用 `git add .`。
- `main/master` 的提交、合并、tag 和推送必须有用户明确指令。

## 文档和资源

- 长期事实写入 README、标准、ADR、架构文档或 CONTEXT。
- 任务过程写入 `.planning/tasks/<date>-<slug>/`。
- 截图、coverage、E2E report、临时脚本、运行日志默认不提交。
- 模块变更必须更新对应模块文档和 `docs/CONTEXT-MAP.md` 的关系说明。

## 红线

- 禁止编造测试结果、命令输出、API 行为或文件关系。
- 禁止硬编码密钥、token、证书和生产账号。
- 禁止静默吞错、乱打日志、输出敏感数据。
- 禁止未确认的数据库、权限、生产配置和破坏性操作。
- 禁止把工具缺失或跳过步骤包装成成功。

## 交付格式

最终回复必须包含：

- 完成内容。
- 实际运行的验证命令和结果。
- 未验证项或剩余风险。
- 如涉及长期规则变化，说明同步更新了哪些文档。

---
> Source: [hongmaple0820/project-scaffold](https://github.com/hongmaple0820/project-scaffold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
