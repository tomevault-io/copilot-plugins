## harness-forge

> **Harness Forge** — AI coding agent 的工具流程模版平台。5 步向导 → 下载 ZIP（CLAUDE.md, settings.json, 脚手架）。

# CLAUDE.md

## Project Overview

**Harness Forge** — AI coding agent 的工具流程模版平台。5 步向导 → 下载 ZIP（CLAUDE.md, settings.json, 脚手架）。

## Commands

```bash
npm run dev      # Dev server (localhost:3000)
npm run build    # Production build
npm run lint     # ESLint
npm run start    # Serve production build
```

No test framework.

## Tech Stack

- Next.js 16 App Router + React 19 + TypeScript
- Tailwind CSS v4 + shadcn/ui (base-nova, lucide icons)
- Path alias: `@/*` → `./src/*`

## Directory Layout

```
src/
  components/ui/       — shadcn/ui primitives
  components/wizard/   — wizard-specific components
  generators/          — output file generators (engines/claude, codex, cursor)
  templates/           — template presets (solo-dev, gstack-sprint, managed-agents)
  store/               — Zustand store
  types/               — type definitions
```

## Wizard Flow

1. **Project Basics** (`/wizard`) — name, description, tech stack
2. **Architecture** (`/wizard/architecture`) — Session/Harness/Sandbox config
3. **Flow** (`/wizard/flow`) — sprint stages, roles, constraints
4. **Integration** (`/wizard/integration`) — MCP servers, hooks
5. **Generate** (`/wizard/generate`) — preview output files, download ZIP

Data flow: `Template Presets → URL param → Zustand Store → Generators → JSZip → Download`

## Sprint Stages

7 stages (Think → Plan → Build → Review → Test → Ship → Reflect)，融合 Anthropic Managed Agents (Session/Harness/Sandbox)、GStack (7 阶段 sprint + 角色驱动)、Superpowers (强制 TDD + 子 agent 并行)。

每个 stage config 包含: `dimensions` (toggle chips), `depth`/`strategy` (select), `gates` (string array)。

## Stage Action Gates

每个阶段在转换时执行实际命令验证（`command_exit` gate），通过 `transition.sh` 触发。命令从 tech stack 推断（`LANGUAGE_ACTION_DEFAULTS`），在 Generate 页预览可微调。

| Stage | 动作 | blockOnFail |
|-------|------|-------------|
| Think | 文档结构验证 (`__THINK_VALIDATE__`) | false (warn) |
| Plan | 计划文档验证 (`__PLAN_VALIDATE__`) | false (warn) |
| Build | lint (`__BUILD_LINT__`) + typecheck (`__BUILD_TYPECHECK__`) | true (block) |
| Review | diff lint (`__REVIEW_DIFF_LINT__`) | true (block) |
| Test | 测试套件 (`__TEST_COMMAND__`) | true (block) |
| Ship | 生产构建 (`__SHIP_BUILD__`) | true (block) |
| Reflect | 回顾文档验证 (`__REFLECT_VALIDATE__`) | false (warn) |

`blockOnFail: true` → 命令失败阻断转换；`false` → 记录警告到 `.harness/log/gate-warnings.jsonl` 继续推进。

关键文件：
- `src/generators/core/stageArtifacts.ts` — 各阶段默认 artifacts（含 command gates）
- `src/generators/core/constraints.ts` — `LANGUAGE_ACTION_DEFAULTS` + 占位符解析 + `constraints.json` 生成
- `src/generators/engines/claude/hooks.ts` — `transition.sh`/`advance.sh` 的 blockOnFail 逻辑
- `src/app/wizard/generate/page.tsx` — StageActionsPreview 组件

## Architecture

C4 架构图见 [`docs/architecture.md`](docs/architecture.md)。

---
> Source: [songshuangkk/harness-forge](https://github.com/songshuangkk/harness-forge) — distributed by [TomeVault](https://tomevault.io/claim/songshuangkk).
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
