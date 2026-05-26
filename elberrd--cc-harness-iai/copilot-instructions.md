## cc-harness-iai

> Minimal Claude Code template with a task-driven workflow:

# Project Instructions

Minimal Claude Code template with a task-driven workflow:

1. **Write the PRD** in `ai-docs/PRD.md` (replace all `{{placeholders}}`).
2. (Optional) **Create the design system** with `/design create` — generates `/DESIGN.md` at the repo root.
3. **Generate the task list** with `/create-tasks` — produces `ai-docs/todos/task-master.md` from the PRD.
4. **Execute the next task** with `/dev` — prepares, implements, and audits against `DESIGN.md`.

## Available commands

| Command | What it does |
|---|---|
| `/create-tasks` | Reads `ai-docs/PRD.md` and generates `ai-docs/todos/task-master.md` with sequential tasks and explicit dependencies. |
| `/dev [description \| --task <selector>] [--test] [--worktree] [--no-branch] [--list] [--dry-run] [--quick] [--ship]` | Selects, prepares, and implements a task. Selection: no args → next pending task in `task-master.md`; `--task <id>` → specific task; `--task 5-10` → range; `--task 5,7,9` → batch (sequential); free-form description → ad-hoc task via `ad-hoc-task-creator` (branch `task/adhoc-<slug>`, NOT in `task-master.md`). Modifiers: `--worktree` creates a git worktree and switches into it; `--no-branch` legacy/hotfix mode; `--list` prints pending/blocked tasks read-only and exits; `--dry-run` writes the task file but skips implementation (single task only); `--quick` skips specialist research and the design-system audit; `--test` runs Playwright E2E at the end; `--ship` auto-pushes + opens PR + queues auto-merge (squash + delete branch) once Steps 3/4/4.5 pass. Without `--ship`, the run ends asking the user whether to merge each PR. The design-system audit runs by default when frontend files were touched. |
| `/design [create\|lint\|check\|export\|spec]` | Manages `/DESIGN.md` (Google Labs alpha spec). `create` invokes the curator to compose the file; `check` audits components against the tokens. |
| `/learning [description]` | Records a lesson learned in `ai-docs/lessons.md` (date, context, root cause, how to avoid, tags). Use after any avoidable mistake. |
| `/manual-verify [request]` | Runs a free-form verification you describe — browser checks via Playwright MCP, CLI checks, etc. — and reports anything that needs human action. |

## Layout

```
README.md                   # How to use this template
CLAUDE.md                   # Project instructions (this file)
DESIGN.md                   # Visual tokens (created by /design create — Google Labs alpha spec)

ai-docs/
├── PRD.md                  # Requirements doc (you fill it in)
├── lessons.md              # Lessons learned (updated by /learning)
├── tools.yaml              # Catalog of tools the agents may use
├── todos/
│   └── task-master.md      # Generated task list (created by /create-tasks)
└── actual-todo/
    └── NN-name.md          # Task currently in flight (created by /dev)

.claude/
├── agents/
│   ├── task-master-generator.md     # generates task-master.md from the PRD
│   ├── task-sequencer.md            # prepares the next PRD task file
│   ├── ad-hoc-task-creator.md       # prepares an off-roadmap ad-hoc task file
│   ├── design-system-curator.md     # creates/updates DESIGN.md
│   ├── design-system-checker.md     # audits components against DESIGN.md (auto in /dev)
│   └── quality-checklist-verifier.md # auto-verifies each task's quality checklist (auto in /dev)
├── commands/
│   ├── create-tasks.md              # /create-tasks
│   ├── dev.md                       # /dev
│   ├── design.md                    # /design
│   ├── learning.md                  # /learning
│   └── manual-verify.md             # /manual-verify
├── hooks/
│   └── block-npm-npx.sh             # PreToolUse hook — redirects npm/npx → pnpm (optional)
├── skills/                          # bundled Skills (stack-specific — keep what you use)
└── settings.json                    # agent-teams env + hook wiring
```

## Invariant rules

### Tasks
- Before generating tasks, `task-master-generator` analyzes the existing codebase to **avoid duplicating** work that's already implemented.
- Tasks have explicit `dependencies` — a task can only start when ALL its dependencies are `done`.
- **Each task is implemented on its own `task/<NN>-<slug>` branch** (created automatically by `/dev`). Branch existence is the **claim**: a tab/session that finds an existing `task/<NN>-*` branch (locally or on remote) skips that task and picks the next one.
- Before the final commit, `/dev` runs `git fetch && git rebase <base-branch>` to pull in `task-master.md` updates from already-merged parallel branches.
- Each task only modifies its OWN row in `task-master.md` (status: `done`). Dependency state (`blocked` ↔ `pending`) is computed lazily by `task-sequencer` — never written eagerly. This keeps parallel PRs conflict-free.
- When finishing a task via `/dev`, update its status in `task-master.md` to `done`.
- **Working tree limpo antes de claim.** `task-sequencer` recusa começar uma task nova se houver mudanças não commitadas. Mudanças de infra (`.claude/**`, `CLAUDE.md`, `ai-docs/PRD.md`, `ai-docs/tools.yaml`, etc.) devem ser commitadas em `<BASE_BRANCH>` como `chore(workflow): ...` antes de iniciar a próxima task. Step 0.4.0 do `task-sequencer` aborta com classificação META/TASK.
- **Branch obsoleto = não resume.** Branch `task/<NN>-*` cuja Task `<NN>` está `status: done` em `task-master.md` é tratado como obsoleto (Mode E). `task-sequencer` para e prompta cleanup; nunca tenta retomar.
- **Archive ao fim de cada task.** Cada `/dev` move o próprio task file de `ai-docs/actual-todo/` para `ai-docs/todos/` via `git mv` no commit final. `actual-todo/` deve ficar vazio (só `.gitkeep`) entre tasks. Se não estiver, `task-sequencer` Step 0.4.0b aborta com mensagem de cleanup.
- Match the language of the PRD (default: English). If the user writes a Portuguese PRD, generate everything in Portuguese.

### Implementation (apply on every `/dev`)
- **🔒 Use official framework/library types — do not invent your own.** Before declaring an `interface` or `type`, look in `node_modules/<lib>/**/*.d.ts`, the library's own exports, or generated code (Convex/Prisma/Drizzle/tRPC). Reuse via `Pick`, `Omit`, `z.infer`, `ComponentProps<typeof X>`. Details in Step 2 of `dev.md`.
- **🔒 Do not regress framework versions to "make a test pass".** Forbidden: downgrades (React 19 → 18, Next 16 → 14, Tailwind v4 → v3), deprecated APIs, unofficial workarounds. If the test fails and the "known solution" requires this, **stop and warn the user**. Details in Step 3 of `dev.md`.
- **🔒 Automatic design-system audit.** At the end of every `/dev`, if any frontend file was modified, the `design-system-checker` agent is invoked. A `❌ REJECTED` verdict blocks marking the task as `done`.
- **🔒 Automatic quality-checklist verification.** At the end of every `/dev` (Step 4.5), the `quality-checklist-verifier` agent runs concrete checks (no `any`, official types not redeclared, Zod on input handlers, error handling on async sites, Tailwind responsive prefixes, touch-target sizes, naming conventions). The task file's `## Quality checklist` is split into `### AI-verified` (auto-checked) and `### User-verified` (manual). Any `FAIL` blocks the commit until fixed; user-only items are surfaced as a clear "👤 Manual verification TODO" block in the final report.

### Lessons
- Whenever you make an avoidable mistake, record it with `/learning` so `ai-docs/lessons.md` grows into a living guide for the project.

### Agent Teams
- `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` is enabled. Both `task-master-generator` and `task-sequencer` are explicitly authorized to spawn ad-hoc specialist sub-agents (via the `Agent` tool, in parallel) when the work is independent and benefits from parallelism. They design the team they need on the fly.

### Optional prerequisites
- `--test` requires the **Playwright MCP** to be configured in `.mcp.json` or `.claude/settings.json`.
- `/design *` requires `npx` (Node.js) — the `@google/design.md` CLI is downloaded on demand.
- `DESIGN.md` is optional. If absent, `design-system-checker` only warns and does not block tasks.

## Multi-tab workflow

`/dev` always isolates a task on its own `task/<NN>-<slug>` branch. The branch is the **claim**: any tab seeing an existing `task/<NN>-*` branch (locally or on remote) skips that task. This is what makes multi-tab work safe.

### Pick the workflow that matches your situation

**Single tab, sequential (simplest):**
- Run `/dev`. It creates the branch in the current working tree, implements, commits, suggests PR.
- Switch to the next task: `git checkout main` → `/dev` again (creates the next branch).

**Single tab, isolated (one task at a time but in a clean filesystem):**
- Run `/dev --worktree`. It creates the branch inside a fresh worktree at `../<repo>-task-<NN>-<slug>` and uses Claude Code's `EnterWorktree` tool to switch your current session into it.
- Implementation continues there. At the end, the conclusion suggests `ExitWorktree({ action: "keep" })` to return, and `git worktree remove <path>` after the PR merges.

**Multiple tabs, true parallelism (3 ways — pick whichever you prefer):**
1. **From an existing tab on `main`:** `/dev --worktree` claims the next task and creates a worktree using our `task/<NN>-<slug>` naming. Then open *another* terminal and either `claude --worktree <name>` or open a second worktree manually for a different task.
2. **From a fresh terminal:** `claude --worktree <name>` (Claude Code's built-in CLI flag). This uses Claude's own naming (`worktree-<name>` branch under `.claude/worktrees/<name>/`), which works but doesn't match our `task/<NN>-<slug>` convention — the task-sequencer will warn (Mode C) and ask how to proceed.
3. **Manually with full naming control:** `git worktree add ../<name> -b task/<NN>-<slug> main`, then `cd ../<name> && claude`, then `/dev` to resume on the existing branch.

In all parallel paths, the claim protocol holds — two tabs on `main` running `/dev` simultaneously won't collide on the same task.

### Inheritance
Worktrees automatically inherit `.claude/`, `CLAUDE.md`, agents, commands, skills, hooks, and `settings.json` from the primary tree. No duplication needed.

For gitignored files you want copied into new worktrees (like `.env.local`), add them to a `.worktreeinclude` file at the repo root (Claude Code reads it natively).

### Known limitations
- **`lessons.md`** is a single shared file. Simultaneous `/learning` calls from multiple tabs may collide — the second writer may need to manually merge. Risk is low in practice.
- **`--test` in parallel** may collide on dev-server port / database. Orchestrate test runs manually if you need them in parallel.
- **Spawning multiple Claude sessions** is an OS-level operation — `/dev --worktree` switches the *current* session into a worktree but cannot spawn a new tab for you.

## Project-specific notes

> Fill this in once you adopt the template — it is the first context every
> `/dev` run reads. Describe what an implementing agent must always know.

- **Stack:** {{frameworks, language, key libraries}}
- **Backend / database:** {{e.g. Convex, Postgres + Prisma, Supabase}}
- **Package manager:** {{npm | pnpm | yarn | bun}}. Note: the bundled
  `.claude/hooks/block-npm-npx.sh` hook assumes **pnpm** and rewrites
  `npm`/`npx` calls. If you use another package manager, delete that hook
  file and the `hooks` block in `.claude/settings.json` (see `README.md`).
- **Conventions:** {{folder layout, naming, validation library, auth pattern,
  i18n — anything the agents should never have to rediscover}}

---
> Source: [elberrd/cc-harness-iai](https://github.com/elberrd/cc-harness-iai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-26 -->
