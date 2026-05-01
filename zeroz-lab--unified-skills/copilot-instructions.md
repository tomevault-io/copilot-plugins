## unified-skills

> > 宪法 + 44 技能 + 9 命令 = 按阶段加载的多产物开发技能套件。支持 Claude Code 和 Codex CLI。

# Unified Skills

> 宪法 + 44 技能 + 9 命令 = 按阶段加载的多产物开发技能套件。支持 Claude Code 和 Codex CLI。

## 如果你是一个 AI Agent

停。先读完这一节再做任何事。

Unified 是一套行为塑造技能——每个 SKILL.md 里的流程、红旗表、常见说辞表都是经过设计的，不是随意写的散文。随意修改措辞会改变 agent 行为，产生无法预料的后果。

**修改技能之前：**
1. 先通读整个技能。理解每个章节为什么存在。
2. 读 [CANON.md](CANON.md)。技能继承宪法的 10 条规则——技能级别的步骤不能与宪法冲突。
3. 跑 `./validate`（见下方验证章节）。提交前修掉所有违规。
4. 新增技能时，用 `templates/feature/` 作为起点。
5. 把完整 diff 给你的人类 partner 看过，获得明确同意。

**创建 PR 之前：**
1. 确认无 stub 残留——每个步骤必须有可操作的内容，不能是占位符。
2. 确认命名规范：`<phase>-<role>-<skill>/SKILL.md`。
3. 确认技能包含：入口/出口条件、可操作流程、常见说辞表、红旗清单、验证清单；强纪律技能必须额外包含 Iron Law。

## 宪法

[CANON.md](CANON.md) — 10 条不可变规则。技能可以添加纪律，但绝不能放松宪法条款。

## 项目结构

```
unified/
├── CANON.md                 宪法（10 条，最高优先级）
├── AGENTS.md                入口配置（本文件）
│
├── skills/                  44 技能 / 6 阶段
│   ├── define/              定义（3）
│   ├── build/               构建（15）
│   ├── verify/              验证（11）
│   ├── ship/                发布（7）
│   ├── maintain/            维护（5）
│   └── reflect/             复盘（2）
│
├── commands/                9 命令入口（Claude Code 斜杠命令）
├── .agents/skills/           9 命令入口（Codex CLI skill 命令）
├── agents/                  15 审查角色（3 代码 + 4 计划 + 3 refine + 1 review + 4 ship）
├── templates/               6 文档模板
└── docs/                    设计文档
```

## 技能按阶段分组

```
define/    → refine（提炼）、spec（规格）、brainstorm（头脑风暴）
build/     → plan（计划）、execute（执行）、tdd（测试驱动）、context（上下文）、
             source-driven（文档驱动）、execution-engine（执行引擎）、
             decision-record（决策记录）、git（版本控制）、
             ui-engineering（UI 工程）、browser-testing（浏览器测试）、
             api-design（API 设计）、database（数据库）、service-patterns（服务模式）、
             content-writing（内容写作）、content-layout（版式）
verify/    → review（审查）、debug（调试）、accessibility（无障碍）、integration-testing（集成测试）、
             performance（性能）、security（安全）、code-review-standards（审查标准）、
             content-review（内容审查）、visual-review（视觉审查）、
             receiving-review（接收审查反馈）、simplify（代码简化）
ship/      → ship（发布）、ci-cd（持续集成部署）、deploy（部署）、artifact-export（产物导出）、
             canary（金丝雀监控）、land（合并部署）、doc-sync（文档同步）
maintain/  → observability（可观测性）、deprecation-migration（废弃迁移）、
             context-save（保存上下文）、context-restore（恢复上下文）、learn（跨 session 学习）
reflect/   → retro（回顾）、documentation（文档）
```

## 命令映射

| 命令 | 加载的技能 | 产出 | 文档路径 |
|------|-----------|------|----------|
| `/refine` | define-workflow-refine | 规范 spec | `docs/features/YYYYMMDD-<name>/01-spec.md` |
| `/plan` | build-workflow-plan | 任务计划 | `docs/features/YYYYMMDD-<name>/02-plan.md` |
| `/build` | build-workflow-execute + artifact_type 对应技能 | 软件/内容产物+验证+ADR | `docs/features/YYYYMMDD-<name>/adr/` |
| `/review` | verify-workflow-review + artifact_type 对应审查 | 审查报告 | `docs/features/YYYYMMDD-<name>/review.md` |
| `/ship` | ship-workflow-ship + artifact-export（非 software） | 发布/导出记录+README | `docs/features/YYYYMMDD-<name>/ship.md` |
| `/save` | maintain-workflow-context-save | 工作上下文 checkpoint | `.claude/checkpoints/YYYYMMDD-HHMMSS-{title}.md` |
| `/restore` | maintain-workflow-context-restore | 恢复上下文 | — |
| `/learn` | maintain-workflow-learn | 学习记录管理 | `.claude/learnings.jsonl` |

### Codex CLI 命令

Codex 中技能通过 `$<skill-name>` 调用：

| 命令 | 加载的技能 | 产出 |
|------|-----------|------|
| `$refine` | define-workflow-refine | `docs/features/YYYYMMDD-<name>/01-spec.md` |
| `$plan` | build-workflow-plan | `docs/features/YYYYMMDD-<name>/02-plan.md` |
| `$build` | build-workflow-execute + artifact_type 对应技能 | 软件/内容产物+验证+ADR |
| `$review` | verify-workflow-review + artifact_type 对应审查 | `docs/features/YYYYMMDD-<name>/review.md` |
| `$ship` | ship-workflow-ship + artifact-export（非 software） | `docs/features/YYYYMMDD-<name>/ship.md` |
| `$save` | maintain-workflow-context-save | `.claude/checkpoints/YYYYMMDD-HHMMSS-{title}.md` |
| `$restore` | maintain-workflow-context-restore | 恢复上下文 |
| `$learn` | maintain-workflow-learn | `.claude/learnings.jsonl` |

Codex 的技能入口在 `.agents/skills/` 目录，每个命令对应一个薄包装技能。

spec 必须声明 `artifact_type`，默认 `software`；可选 `software` / `document` / `article` / `deck` / `visual`。后续阶段按该字段加载软件、内容、版式、审查或导出技能。

`/build` 会读取 `02-plan.md` 总控计划；大型/并行任务还会读取 `plans/*.md` 子计划，并只在 `Parallel Execution Matrix` 证明 `parallel_safe` 时并行分派。

多产物扩展技能采用角色化方法论：先定义角色责任、长期原则和决策框架，再给出流程和验证证据；它们不是工具清单。

`/refine` 使用 Unified 原生 External Scan：按 `artifact_type` 搜索已有方案、事实来源、设计/技术模式，并把结果分层为 Fact / Pattern / Inference / Unknown / Adopt / Reject，再交给 Idea Scout Army 审查。

## 文档产出链

```
docs/features/YYYYMMDD-<name>/
├── 01-spec.md              ← /refine
├── 02-plan.md              ← /plan
├── plans/*.md              ← /plan（大型/并行任务的子计划）
├── adr/<num>.md            ← /build（决策时）
├── review.md               ← /review
├── ship.md                 ← /ship
├── canary-report.md        ← ship-workflow-canary
├── deploy-report.md        ← ship-workflow-land
└── README.md               ← /ship 后聚合

docs/bugs/<name>/
├── 01-root-cause.md        ← verify-workflow-debug Phase 1-3
└── 02-fix-plan.md          ← verify-workflow-debug Phase 4
```

## 约定

### 命名规范
- 技能目录：`<阶段>-<角色>-<技能名>/` —— 每个目录下恰好一个 `SKILL.md`
- 阶段：`define` / `build` / `verify` / `ship` / `maintain` / `reflect`
- 角色：`workflow` / `frontend` / `backend` / `quality` / `cognitive` / `infrastructure` / `team` / `content` / `visual` / `artifact`
- 技能名：kebab-case，描述动作（如 `tdd`、`debug`、`api-design`）

### SKILL.md 格式
- 每个技能必须包含中文语义章节：入口/出口条件、可操作流程、常见说辞表、红旗清单、验证清单
- 验证失败处理用于 workflow / gatekeeping / 高风险技能；不强制所有辅助技能重复该章节
- 引用其他技能用技能名，不用文件路径
- 强制纪律类技能（TDD、调试、审查、发布）必须有 Iron Law 章节
- 辅助文件仅在内容超过 100 行时才创建

### 命令
- `commands/` 下每个命令一个 `.md` 文件（Claude Code 斜杠命令）
- `.agents/skills/` 下每个命令一个 `SKILL.md`（Codex CLI 技能命令）
- 命令加载技能，但不重复技能内容

## 验证

```bash
./validate
```

## 边界

### 始终要做
- 新增技能必须遵循命名规范
- 引用 CANON.md 而不是重复宪法条款
- 引用其他技能名而不是复制其内容
- 用 `templates/` 下的模板作为起点
- 调用技能前先通读整个技能
- 实现非平凡变更前先陈述假设

### 绝不能做
- 不能添加"空泛建议"而非可操作流程的技能
- 不能添加仅服务特定项目/团队/领域的技能——这类技能属于独立插件
- 不能在技能间重复内容——用引用代替
- 不能在技能中放松宪法条款。技能只能增加纪律，不能减少
- 不能在未理解其行为塑造目的的情况下修改红旗表或常见说辞表
- 不能替换"human partner"的措辞——这是有意为之，不可与"the user"互换
- 不能引入第三方依赖。Unified 设计为零依赖

## 开发注意事项

技能支持多平台挂载：

- **Claude Code**: `skills/` 是真实技能目录，可挂载到 `.claude/skills/`
- **Codex CLI**: `.agents/skills/` 只保留 8 个薄包装命令，底层加载 `skills/` 中的真实技能

修改**实时生效**。这意味着：
- 对 SKILL.md 的修改在下一次调用时立即生效
- 重构期间的破坏性变更可能导致并行 session 出错
- 大规模修改前先在隔离环境中测试

---
> Source: [ZeroZ-lab/unified-skills](https://github.com/ZeroZ-lab/unified-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
