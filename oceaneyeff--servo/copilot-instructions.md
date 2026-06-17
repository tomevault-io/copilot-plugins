## servo

> > 这是当前仓库的 agent-facing 最小工作规则入口。若与 `docs/project-maintenance/` 或 `docs/harness/` 冲突，以对应承接层文档为准。

# AGENTS.md

> 这是当前仓库的 agent-facing 最小工作规则入口。若与 `docs/project-maintenance/` 或 `docs/harness/` 冲突，以对应承接层文档为准。

## Core

- 本项目的核心目标，是构建一个 `Codex-first` 的 AI coding harness 平台，并将其作为 repo-side contract layer 分发到多个项目中使用。
- 当前仓库以 AI coding 的 repo-side contract layer 形态承接这个目标。
- `docs/` 负责 truth boundary，`product/` 负责 canonical skills 与 adapters，`toolchain/` 负责部署、评测与治理脚本。
- `Harness` 现在是一级认知与文档域；已批准输入、执行边界、路由和写回规则由 `AGENTS.md`、`docs/harness/artifact/` 与 `docs/project-maintenance/governance/` 承接，不再保留独立 adjacent-system 文档域。
- `.agents/`、`.claude/` 只是 deploy target，不是源码层。
- 如果一个新目录说不清 owner 和层级，不要直接加到根目录。

## Default Boot

默认启动只读最小路由面：

1. `AGENTS.md`
2. `INDEX.md`
3. 根据 `INDEX.md` 选择当前任务对应的一个局部入口

仅当任务命中对应边界时才扩读承接层文档：

- 根目录分层、一级目录、hidden/state/mount 层或 `.nav/` 规则变化：扩读 `docs/project-maintenance/foundations/README.md`、`docs/project-maintenance/foundations/root-directory-layering.md` 和相关治理检查说明。
- `AGENTS.md`、review/verify 流程或退出标准变化：扩读 `docs/project-maintenance/governance/review-verify-handbook.md`。
- deployment / adapter 行为变化：扩读 `docs/project-maintenance/deploy/` 的相关入口与 `toolchain/README.md`。
- Harness doctrine、workflow family、artifact 合同或 canonical skill 入口路径变化：扩读 `docs/harness/README.md`、局部 artifact/skill 入口和对应 governance 检查。
- Worktrack 执行、调度、证据或关卡变化：扩读对应 `docs/harness/artifact/worktrack/` 局部合同，不默认预读全量 worktrack artifact。

## Route Contract

- `read_next`：
  - `INDEX.md`
  - 按任务进入 `product/`、`docs/` 或 `toolchain/` 的局部入口页
  - 只在命中治理、分层、Harness artifact、dispatch / gate / closeout 策略时扩读对应 foundations / governance / artifact 文档
- `do_not_read_yet`：
  - `.servo/`（运行 Harness 控制回路时可读取当前 control state 与必要 runtime artifact）
  - `.agents/`
  - `.claude/`
  - `.autoworkflow/`
  - `.spec-workflow/`
  - `.nav/`
- `stop_reading_when`：
  - 已确认当前任务落在哪一块正式内容区
  - 已拿到当前任务所需的最小模块入口
  - 继续扩读只会重复背景，而不会增加决策价值

## Default Flow

1. `plan`：先固定目标、范围、非目标、验收、风险和验证要求。
2. `implement`：只做当前任务，不顺手扩边界。
3. `verify`：先跑与改动面匹配的检查和测试。
4. `review`：把 diff、计划和验收标准对齐，确认没有遗漏同步项。
5. `writeback`：把已验证事实写回 `docs/project-maintenance/` 或 `docs/harness/`，并清理失效上下文。

## Required Sync

- 根目录分层、一级子目录、hidden/state/mount 层或 `.nav/` 规则变化时，必须同步更新 foundations 文档和对应治理检查。
- `AGENTS.md`、review/verify 流程或退出标准变化时，必须同步更新 `docs/project-maintenance/governance/review-verify-handbook.md`。
- deployment / adapter 行为变化时，必须同步更新相关 `docs/project-maintenance/deploy/` 文档和 verify 命令说明。
- Harness doctrine、workflow family、artifact 合同或 canonical skill 入口路径变化时，必须同步更新 `docs/harness/`、`product/*/skills/` 入口与对应治理检查。
- 只有已验证结果才可以回写为长期真相；未验证的结论不要写进知识层。

## Root Rules

- `product/` 是业务代码唯一源码根。
- 当前 `docs/` 是文档层，内部为 `book.md / project-maintenance / harness`。
- `docs/harness/` 是 Harness-first 文档主线。
- `toolchain/` 只放脚本、评测、测试、打包、部署工具。
- `.autoworkflow/`、`.spec-workflow/` 只属于 repo-local state layer。
- `.nav/` 只是 compatibility navigation layer，不能当真实结构定义。

## Docs Governance Baseline

- `docs/README.md`、`docs/project-maintenance/README.md`、`docs/harness/README.md` 和 `docs/*/README.md` 只做入口导航，不承载独占规则正文。
- `docs/` 下除 `README.md` 外的正文文档必须有 frontmatter：`title / status / updated / owner / last_verified`。
- `status` 只允许：
  - `docs/project-maintenance/` 与 `docs/harness/`：`active | draft | superseded`
- 不在 `docs/` 长期使用 `status: suspended`。共享保留内容转 `superseded`，非共享草稿移出 `docs/`。
- 研究结论准入后必须升格到承接层：
  - 项目维护规则、governance、deploy、usage-help 写 `docs/project-maintenance/`
  - Harness doctrine、workflow family 与 artifact 合同写 `docs/harness/`
  - 实现合同落 `product/` 或 `toolchain/`
- 新增或接管文档作用域时，必须同步更新最近入口页并清理旧入口，避免双份主线。

## Review / Verify

- 常规复核入口见 `docs/project-maintenance/governance/review-verify-handbook.md`。
- 在本仓库运行 Python 验证、部署或辅助命令时，默认使用 `PYTHONDONTWRITEBYTECODE=1 python3 ...`，避免在 `product/`、`docs/`、`toolchain/` 或 `tools/` 下生成 `.pytest_cache`、`__pycache__`、`.pyc` 或 `.pyo` 运行缓存。
- 修复类任务不得只压住当前症状；必须检查相邻状态、恢复路径和 operator-facing 语义，避免引入新的问题源，并尽量把修复做完整。
- 新功能若改变 Harness / skill / adapter / CLI / runbook 等用户实际操作路径，mock、fixture、单元测试只能作为回归层；收口前默认需要按 `docs/project-maintenance/testing/claude-post-deploy-behavior-tests.md` 补真实 Claude Code dogfood 证据，除非在 closeout 中明确说明不适用理由。
- 涉及根目录、路径、分层或治理规则时，优先跑：
  - `PYTHONDONTWRITEBYTECODE=1 python3 toolchain/scripts/test/folder_logic_check.py`
  - `PYTHONDONTWRITEBYTECODE=1 python3 toolchain/scripts/test/path_governance_check.py`
  - `PYTHONDONTWRITEBYTECODE=1 python3 toolchain/scripts/test/governance_semantic_check.py`
- 涉及 closeout 或 gate 变更时，再跑：
  - `PYTHONDONTWRITEBYTECODE=1 python3 toolchain/scripts/test/closeout_acceptance_gate.py --json`
- 涉及 adapter / deploy 变更时，再补对应 `adapter_deploy.py verify`。

## Writeback

- 项目维护与治理正文写到 `docs/project-maintenance/`。
- Harness doctrine、workflow family 与 artifact 合同写到 `docs/harness/`。
- template / checklist 只在它们能稳定承接执行时才保留。
- 不要把项目真相写进 `.agents/`、`.claude/`、`.nav/`。

## Current Warnings

- `.nav/` 现在只保留 `@docs` 与 `@skills` 两个有效兼容入口。
- `tools/` 只是 compatibility shim，真逻辑仍应落在 `toolchain/scripts/test/`。
- `docs/harness/adjacent-systems/`、`product/memory-side/` 与 `product/task-interface/` 已退役；相关执行边界由 `docs/harness/artifact/`、`AGENTS.md` 和 review/writeback 治理承接。

## Canonical References

- `docs/README.md`
- `docs/project-maintenance/README.md`
- `docs/harness/README.md`
- `docs/project-maintenance/foundations/README.md`
- `docs/project-maintenance/foundations/root-directory-layering.md`
- `docs/harness/foundations/Harness指导思想.md`
- `docs/harness/foundations/Harness运行协议.md`
- `docs/harness/artifact/worktrack/contract.md`
- `docs/harness/artifact/worktrack/plan-task-queue.md`
- `docs/harness/artifact/worktrack/gate-evidence.md`

---
> Source: [OceanEyeFF/servo](https://github.com/OceanEyeFF/servo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
