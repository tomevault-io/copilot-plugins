## siyuan-cli

> - 在本 repo 内运行 self CLI 时，使用 `pnpm run siyuan ...`。

项目当前处于 Alpha 内部开发阶段

---

## Project-local rules

- CLI 内部文档、提示使用英文。
- 在本 repo 内运行 self CLI 时，使用 `pnpm run siyuan ...`。
- 不要用全局 `siyuan` 代替本地开发版；全局安装可能不是最新代码。

## Key paths

- 项目代码：`src/`
- CLI 内置文档：`src/docs/`（随 package 发布）
- CLI 内置 SKILL：`skills/`（install 后落在全局 `~/.agents/skills/`）
- 已安装 package 视角：运行时代码与类型声明看同 package 下的 `dist/`

## 安装本项目的 Agent 使用者视角

- 先看 `siyuan-cli` SKILL
- 再运行 `pnpm run siyuan --help`
- CLI 会继续引导 Agent 查看内置文档，以及同 package 下 `dist/` 中的运行时代码

## Useful tools

- `rg` / `grep` — Grep
- `fd` — Find file
- `sspec tool view-tree` — List file tree
- `sspec tool mdtoc` — Inspect long md file
- `slsp` — LSP CLI (`slsp --help`)
- `sspec change [new|list|archive]`

## Test CLI

本项目应当使用 `dev` 的思源空间开发测试，避免访问用户主空间。
测试过程中若 dev workspace 未启动，应当告知用户，要求他启动。

## Gotchas

- When editing the siyuan-cli skill, edit `<cwd>/skills/siyuan-cli` (source) instead of `~/.agents/skills/...` (distribution)  

---

<!-- SSPEC:START -->
# .sspec Agent Protocol

SSPEC_SCHEMA::6.0

## 0. Structure

A spec-driven workflow, via `sspec` CLI and `.sspec/`.

**Core Principle**: The user MUST be able to predict the outcome before implementation begins.
When uncertain, align — never proceed with unclarified assumptions.

```
.sspec/
├── project.md     # Identity, conventions, notes
├── spec-docs/     # Knowledge: in-code-but-scattered or outside-code
├── changes/<n>/   # spec.md | tasks.md | memory.md [+ design.md | revisions/ | reference/]
├── requests/      # User intent records
└── tmp/           # Informal drafts
```

## 1. Dispatch

`read(project.md)` → classify → act:

| Input | Action |
|-------|--------|
| Directive (`@resume`, `@memory`, etc.) | Execute → §4 |
| Request under `.sspec/requests` | Assess scale → §2 |
| Resume existing change | `read(memory)` → infer phase from State → load phase SKILL → continue |
| Create request | `sspec request new` |
| Create spec doc | `sspec doc new` |
| Micro (≤3 files, ≤30min, obvious) | Do directly |
| Mini (user opts out of formal change) | Clarify+Design thinking → `sspec tmp new` → §2.0 |

**Trigger-word → SKILL**:

| User says | Load |
|-----------|------|
| clarify, 搞清楚, 理解一下 | `sspec-clarify` |
| design, 设计, 出方案 | `sspec-design` |
| align, 对齐, 确认一下 | §3 protocol |
| plan, 拆任务 | `sspec-plan` |
| implement, 动手, 开始做 | `sspec-implement` |
| review, 检查, 看看 | `sspec-review` |
| mini change, 不要 change, 直接推进 | §2.0 |

**Standing rules**:
- Follow `Core Principle`.
- Important discovery → `memory.md` Knowledge immediately
- Session end → MUST update memory.md (State + Milestones) · `sspec howto write-memory`
- @align gate decisions → SHOULD update memory.md Knowledge
- Time uncertain → `sspec tool now`
- Template HTML comments with BCP 14 keywords (MUST, SHOULD, MAY per RFC 2119) are persistent constraints — never delete them.

## 2. Change Lifecycle

Each phase has a SKILL. MUST read it before starting.

```
Clarify  (sspec-clarify)    posture, reusable       exit: ready for spec
Design   (sspec-design)     spec.md [+design.md]    exit: @align gate ■
Plan     (sspec-plan)       tasks.md                exit: @align report →
Implement(sspec-implement)  code + tasks progress   exit: @align gate ■
Review   (sspec-review)     DONE | fix→Implement | amend→revision | follow-up→new change
```

`■` = hard stop, **MUST stop & align**. `→` = output summary, COULD keep going. Failed gate → return, update, realign.
Post-Design gate: spec.md/design.md baselines immutable. Changes → `revisions/NNN-*.md`.
memory.md: maintained throughout, not a phase. → `sspec howto write-memory`

→ `sspec howto handle-review-scope-change`

### 2.0 Mini Change Protocol

Clarify/Design thinking without change entity. Output → `.sspec/tmp/`.

Trigger: user explicitly opts out of formal change.
Flow: clarify → design-level output → `sspec tmp new <topic>` → no gates, no tasks, no memory.
Boundary: no code changes. If implementation needed → upgrade to change or confirm Micro.
Agent MUST NOT self-downgrade to mini — only responds to user intent.

### Scale

| Scale | Criteria | Path |
|---|---|---|
| Micro | ≤3 files, ≤30min, trivially reversible | Do directly |
| Single | ≤1 week, ≤15 files, ≤20 tasks | `sspec change new <name>` |
| Multi | >1 week OR >15 files OR >20 tasks | `sspec change new <name> --root` → sub-changes |

Status in spec.md MUST follow state machine. → `sspec howto update-change-status`

## 3. @align

Structured sync at decision points. **Formalized exchange, not prose.**

**Format rule**: MUST be scannable in 5 seconds.
GOOD: structured (tables, labeled items, code blocks) with high density.
BAD: prose-style, redundant.

| Level | Behavior | When |
|---|---|---|
| `report` | Summary, **keep going** | Plan done, progress |
| `gate` | Summary, **stop and wait** | Design done, implement done, blockers, scope change |

Decisions → natural home: design → spec.md, direction → memory.md Knowledge.
📚 Full mechanics: `sspec-align` SKILL

## 4. Reference

**Directives**: `@change <n>` | `@resume` | `@memory` | `@sync` | `@argue` | `@subagent-audits`

**Spec-Docs**: Knowledge that code alone cannot adequately convey — either in code but scattered or hard to reconstruct (cross-module architecture, UX requirements, design norms, deliberate trade-offs), or entirely outside code (platform rules, API quirks, business constraints). Registered in `project.md` Spec-Docs Index. → `write-spec-doc` SKILL

**CLI**:

| Command | Use |
|---------|-----|
| `sspec change new <name> [--from REQ] [--root] [--scaffold design]` | Create change |
| `sspec change scaffold <type> <change>` | Add file: tasks, design, revision |
| `sspec change find/status <name>` | Inspect change |
| `sspec doc new "<name>"` | Create spec-doc |
| `sspec howto [name...]` | Read HOWTOs (batch) |
| `sspec tool <name> [opts]` | CLI tools (`--prompt` for usage) |

**Tools** (`sspec tool <name>`): `now` · `ask` · `mdtoc` · `view-tree` · `fileinfo` · `patch/write` · `treesitter`
  Frequent: `now`, `mdtoc`, `view-tree`; See `sspec tool <name> --prompt` for usage.

**HOWTO**: `sspec howto list` to browse; batch-read with `sspec howto read <n1> <n2>`.
**SKILL**: Read before starting phase. Referenced file → MUST read. `sspec-*` not loaded → find under `.sspec/skills/`.

**Fence nesting**: When showing content that contains ` ``` `, outer fence MUST use more backticks (e.g. `````). Always outer > inner.
<!-- SSPEC:END -->

---
> Source: [frostime/siyuan-cli](https://github.com/frostime/siyuan-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
