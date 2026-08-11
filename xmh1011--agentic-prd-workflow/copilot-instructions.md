## agentic-prd-workflow

> 本文件是给 Codex、Claude Code 或其他编码 Agent 的仓库级执行指令。人类使用说明看 `README.md` 和 `docs/`。

# AGENTS.md

本文件是给 Codex、Claude Code 或其他编码 Agent 的仓库级执行指令。人类使用说明看 `README.md` 和 `docs/`。

## 默认上下文

执行任何 PRD 变更任务前，先读取：

1. `AGENTS.md`
2. `.agent/project_steering.md`，如果存在
3. `.agent/prd_delta.md`，如果存在
4. `.agent/task_plan.md`，如果存在
5. `.agent/handoff_manifest.md`，如果存在
6. 与任务相关的 skill：
   - Codex 使用 `.agents/skills/**/SKILL.md`
   - Claude Code 使用 `.claude/skills/**/SKILL.md`

不要把 `docs/DESIGN.md` 和 `docs/SETUP.md` 全量塞进上下文，除非用户明确要求解释设计或配置。

## 标准命令

跨平台主入口：

```bash
python3 scripts/prd_workflow.py analyze
python3 scripts/prd_workflow.py steering
python3 scripts/prd_workflow.py manifest
python3 scripts/prd_workflow.py next
python3 scripts/prd_workflow.py progress
python3 scripts/prd_workflow.py complexity
python3 scripts/prd_workflow.py lanes
python3 scripts/prd_workflow.py design
python3 scripts/prd_workflow.py packages
python3 scripts/prd_workflow.py delegation
python3 scripts/prd_workflow.py clarify
python3 scripts/prd_workflow.py issues
python3 scripts/prd_workflow.py checklist
python3 scripts/prd_workflow.py consistency
python3 scripts/prd_workflow.py retrospect
python3 scripts/prd_workflow.py validate
python3 scripts/prd_workflow.py full
python3 scripts/prd_workflow.py status
```

Windows PowerShell：

```powershell
powershell -ExecutionPolicy Bypass -File scripts/prd_workflow.ps1 validate
```

如果本机没有 `python3`，但 `python` 是 Python 3，可使用：

```bash
python scripts/prd_workflow.py validate
```

## PRD 变更任务流程

当用户要求处理 PRD 变更时：

1. 如果 PRD 发生变化，运行：

```bash
python3 scripts/prd_workflow.py analyze
```

2. 读取 `.agent/project_steering.md`、`.agent/prd_delta.md`、`.agent/task_plan.md` 和 `.agent/handoff_manifest.md`。
3. 可运行 `python3 scripts/prd_workflow.py next` 查看依赖已满足的下一批 required 任务。
4. 可运行 `python3 scripts/prd_workflow.py progress` 检查 `.agent/task_progress.json` 是否符合任务依赖和状态约束。
5. 可运行 `python3 scripts/prd_workflow.py complexity` 查看 required 任务复杂度和拆分建议。
6. 可运行 `python3 scripts/prd_workflow.py lanes` 查看 required 任务的并行 lane/worktree 协作建议。
7. 可运行 `python3 scripts/prd_workflow.py design` 生成 requirements-to-design-to-tasks 的只读实现设计简报。
8. 可运行 `python3 scripts/prd_workflow.py packages` 生成本地只读 work package 交接文件。
9. 可运行 `python3 scripts/prd_workflow.py delegation` 生成主会话、专用 Agent 和 skill 的只读委派计划。
10. 只实现 `task_plan` 中 `required` 的任务；`review` 任务只评估，不主动扩展。
11. 使用 `.agent/task_plan.md` 的 `Requirement Traceability` 检查每个需求编号是否有对应任务和验证入口。
12. 涉及认证、权限、安全、数据迁移、外部 API 或核心接口时，必须补充真实项目验证钩子，或说明当前仓库只是框架 harness。
13. 完成后运行：

```bash
python3 scripts/prd_workflow.py validate
```

14. 验证通过后，确认 `agent/reports/validation-evidence.md`、`agent/reports/clarification-questions.md`、`agent/reports/task-progress.md`、`agent/reports/task-complexity.md`、`agent/reports/lane-plan.md`、`agent/reports/implementation-design.md`、`agent/reports/work-package-plan.md`、`agent/reports/agent-delegation-plan.md`、`agent/reports/issue-plan.md`、`agent/reports/review-checklist.md`、`agent/reports/consistency-analysis.md`、`agent/reports/retrospective.md` 和 `agent/reports/pr-body.md` 已生成。

## 实现约束

- 不要自行扩大需求范围。
- 不要编造业务口径、指标定义、权限边界或数据来源。
- 不确定事项写入 `agent/reports/open-questions.md`。
- 不要删除或降低验证门禁。
- 不要修改生产密钥、访问令牌、认证绕过逻辑。
- 不要直接 push 到 `main` 或 `master`。
- 不要在 PR body 中隐藏失败项。

## 安全边界

涉及安全或权限时必须考虑：

- 服务端校验
- API 契约
- 数据访问边界
- 敏感字段脱敏
- 文件下载和分享边界
- 审计或操作日志

涉及 AI 或自动化代码生成时必须考虑：

- 不自动执行危险命令
- 不访问密钥文件
- 不生成破坏性数据库操作
- 不绕过 review 和验证门禁
- 对不确定结论保持待确认

## 验证要求

必须运行：

```bash
python3 scripts/prd_workflow.py validate
```

接入真实项目后，将真实命令接入：

- `PROJECT_AUTHZ_VALIDATE_COMMAND`
- `PROJECT_SECURITY_VALIDATE_COMMAND`
- `package.json` 中的 `format`、`lint`、`typecheck`、`build`
- `migrations/`
- `contracts/`

## 提交与交付

只有验证通过才允许建议提交。

提交信息格式：

```text
docs(prd): update <需求编号或摘要>
feat(prd): implement <需求编号或摘要>
fix(prd): fix <问题摘要>
test(prd): add coverage for <需求编号或摘要>
```

最终交付时报告：

- 改了哪些文件
- 运行了什么验证
- 是否生成 `agent/reports/pr-body.md`
- 是否生成 `agent/reports/validation-evidence.md`
- 是否生成 `agent/reports/prd-quality.md`
- 是否生成 `agent/reports/clarification-questions.md`
- 是否生成 `agent/reports/test-strategy.md`
- 是否生成 `agent/reports/task-progress.md`
- 是否生成 `agent/reports/task-complexity.md`
- 是否生成 `agent/reports/lane-plan.md`
- 是否生成 `agent/reports/implementation-design.md`
- 是否生成 `agent/reports/work-package-plan.md`
- 是否生成 `agent/reports/agent-delegation-plan.md`
- 是否生成 `agent/reports/issue-plan.md`
- 是否生成 `agent/reports/review-checklist.md`
- 是否生成 `agent/reports/consistency-analysis.md`
- 是否生成 `agent/reports/release-gate.md`
- 是否生成 `agent/reports/retrospective.md`
- 是否按需生成 `agent/reports/next-action.md`
- 是否存在未解决问题

---
> Source: [xmh1011/agentic-prd-workflow](https://github.com/xmh1011/agentic-prd-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
