## infinitum

> 本项目使用 Forge Loop 共享 Agent 工作流。不要在本项目复制完整核心规则；通用流程从共享核心读取，项目事实从本地 `.ai/` 文件读取。

# AGENTS.md

## Project Entry

本项目使用 Forge Loop 共享 Agent 工作流。不要在本项目复制完整核心规则；通用流程从共享核心读取，项目事实从本地 `.ai/` 文件读取。

## Read Order

任何 Agent 开始工作前必须按顺序阅读：

1. `.ai/workflow-config.md`
2. `.ai/project-context.md`
3. `.ai/overrides.md`
4. `.agent-workflow/forge-loop/core/AGENTS.core.md`
5. `.agent-workflow/forge-loop/core/.ai/adapters/agent-entry.md`
6. `.agent-workflow/forge-loop/core/.ai/adapters/resolver.md`
7. 相关 workflow、skill 和 template

## Minimal Use

用户可以直接说：

```text
/forge <需求、bug、探索问题或继续指令>
```

Agent 必须自动判断走 Quick Lane、Feature Iteration、单任务实现、Review 或复盘，不要求用户手动选择流程。

自然语义也应直接唤起对应阶段，例如：
- “设计一下 / 技术方案” -> H2 Design
- “拆任务 / 排计划” -> H3 Task Planning
- “补测试 / 回归一下” -> Testing
- “集成一下” -> Integration
- “准备提交 / 开 PR / 提交前检查” -> Code Review
- “复盘一下 / 收尾” -> Retrospective

## Always-On Discipline

只要本项目存在 `.ai/workflow-config.md`，Forge Loop 不只服务于 slash-style 命令。用户用普通文本要求实现、修 Bug、分析测试失败、Review、继续迭代或确认是否完成时，Agent 也必须先按 Forge Loop 路由或检查状态，再写代码、声明完成或跨门禁。

- Bug、测试失败和异常行为：先复现、读错误、查近期变更、定位根因，并把证据写入 quick task、requirement 或 task result。
- 完成声明：必须基于本轮新运行的验证命令和结果，不能用推测性语言代替证据。
- 准备提交 commit、开 PR、请求 merge 或提交前检查时：先走 `forge-loop review` / Code Review Skill，对当前 diff 做结构化 pre-commit 检查。
- 并行、Medium 及以上风险、共享契约、跨模块或超过 3 个文件的任务：任务完成后先做 Spec Compliance Review，再做 Code Quality Review。
- 进入 Quick Lane 或 Feature Iteration 后，读取 `forge-loop route` 返回的 `modelTier`、`recommendedModel`、`agentEnvironment`、`neededFiles` 和 `contextEstimate`；小改动优先降到低档位模型，复杂任务优先升到高档位模型，并按 `neededFiles` 读取上下文。
- 需求、设计、任务和测试命名优先使用 `CONTEXT.md`、`CONTEXT-MAP.md`、`.ai/project-context.md` 或 `docs/agents/domain.md` 中已有的领域语言。
- Issue URL、`#123` 或本地 issue 文件按可用的 `docs/agents/issue-tracker.md` 和 `docs/agents/triage-labels.md` 做 intake / triage；小范围明确改动仍可走 Quick Lane。
- 架构改善和 zoom-out 请求默认走 Quick Lane spike，只产出模块地图、架构候选和 follow-up，不直接重构。
- 修改 Forge Loop 核心、模板、命令触发语或适配层时，必须补充 smoke test 覆盖对应行为。

## CLI Controls

如果项目环境可以运行 `forge-loop` 或 `npx @shawnxie666/forge-loop`，Agent 必须优先用 CLI 固定流程控制：

| Step | Command |
| --- | --- |
| 路由判断 | `forge-loop route "<request>" --json` |
| 生成产物 | `forge-loop scaffold <quick|requirement|feature|design|contract|plan|parallel-plan|result|retrospective|experiment> --slug <slug> --request "<request>"` |
| 门禁检查 | `forge-loop gate <H1|H2|H3|H4> --slug <slug>` |
| 人类批准记录 | `forge-loop approve <H1|H2|H3|H4> --slug <slug>` |
| 下一步判断 | `forge-loop next [--slug <slug>]` |
| 状态检查 | `forge-loop status [--slug <slug>]` |
| 工作流评估 | `forge-loop eval [--summary|--propose] [--slug <slug>]` |
| 填写质量检查 | `forge-loop validate [--slug <slug>]` |
| 安装与工作流检查 | `forge-loop doctor --tier auto --workflow --slug <slug>` |

CLI 不可用时，才按 `.forge-loop/commands/` 和共享 core 文档手动执行同等逻辑。读取 Workflow Core 时只读 `.ai/workflow-config.md` 的 `Core Files` 以及当前阶段需要的 workflow、skill、template，不要读取整个 core 目录。不得在 `gate` 返回 blocked 时继续跨越对应人工门禁；`gate` 通过后仍需人类确认，并用 `approve` 写入批准记录。

`scaffold` 和 `approve` 会维护 `.forge-loop/state/active.json`；未传 `--slug` 的 `status`、`next` 和 `validate` 应优先使用该 active state。CLI 也可以把 route/scaffold/gate/approve/validate 的轻量 eval 事件写入 `.forge-loop/state/evals/`，用于 `forge-loop eval --summary` 和 `forge-loop eval --propose`；事件里可包含上下文估算，后续用来发现过大的读取计划。`.forge-loop/state/` 是本地运行态，不提交到业务仓库；eval proposal 只能进入 `docs/workflow-experiments/` 实验，不自动改核心规则。

## Slash-Style Commands

如果用户输入以下命令，Agent 必须读取 `.forge-loop/commands/<command>.md` 并按其中流程执行：

| Command | Purpose |
| --- | --- |
| `/forge <任意请求>` | 默认智能路由，自动判断 Quick / Feature / Next / Implement / Review |
| `/forge-quick <需求>` | 低风险小改动，走 Quick Lane |
| `/forge-fix <问题>` | 小 bug 修复，走 Quick Lane |
| `/forge-spike <问题>` | 技术探索，只分析不实现 |
| `/forge-feature <需求>` | 新功能需求进入 H1 |
| `/forge-bugfix <问题>` | Bug 修复进入 H1 |
| `/forge-next [说明]` | 从当前门禁继续到下一门禁 |
| `/forge-implement <Task ID>` | 执行单个任务 |
| `/forge-review [范围]` | 生成 Review 报告并停在 H4 |

Codex 使用用户级 skill `~/.codex/skills/forge-loop/SKILL.md` 来识别这些命令；其他 Agent 如果没有项目级自定义 slash-command 文件机制，也必须把这些输入当作普通文本命令处理。`.forge-loop/commands/` 默认是本地适配缓存，如果缺失，应优先运行 `forge-loop init --profile codex` 或直接使用 CLI 控制命令。

默认优先使用 `/forge`。只有用户明确想指定路径时，才使用 `/forge-quick`、`/forge-feature` 等细分命令。

## Local Project Rules

- 项目产物写入本项目的 `specs/`、`tasks/`、`docs/`。
- Quick Lane 产物写入 `tasks/quick/`。
- `.agent-workflow/` 默认是本地缓存，不提交到业务仓库；缺失时运行 `forge-loop upgrade` 恢复。
- `.forge-loop/` 和 `.claude/skills/` 默认是本地工具适配缓存，不提交到业务仓库；缺失时重新运行对应 profile 的 `forge-loop init`。
- `.forge-loop/state/` 始终是本地运行态，不提交到业务仓库。
- `docs/agents/` 记录 issue tracker、triage label 和 domain docs 规则。
- `docs/adr/` 记录少量高价值架构决策。
- `.out-of-scope/` 记录明确拒绝或长期不做的事项。
- 不修改共享核心目录，除非用户明确要求升级工作流或本项目使用 `--vendor-core` 固定 core。
- 项目专属规则写入 `.ai/overrides.md`。
- 项目事实、命令、技术栈写入 `.ai/project-context.md`。
- 命令语义写在 `.forge-loop/commands/`。

## Human Gates

- H1：需求确认。
- H2：设计与契约确认。
- H3：并行计划确认。
- H4：最终 Review。
- H5：上线确认，可选。

---
> Source: [shawnxie94/infinitum](https://github.com/shawnxie94/infinitum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
