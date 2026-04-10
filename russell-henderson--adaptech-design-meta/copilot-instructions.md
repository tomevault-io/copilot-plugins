## adaptech-design-meta

> This repository contains:

# Adaptech Design Company — AGENTS.md (Codex Operating Rules)

This repository contains:
- Repo root docs (planning + lint config): `F:\adaptech-design\`
- Next.js app: `F:\adaptech-design\web\`

Codex should treat `web/` as the primary working directory for all build/run commands.

## Working Directory Rules (critical)

1) Default working directory for commands:
- Always run Node/Next/pnpm/npm commands from `web/`:
  - `cd F:\adaptech-design\web`

2) Reading the plan:
- The canonical build plan is `F:\adaptech-design\TODO.md`.
- When executing work items, read `..\TODO.md` from within `web/`.

3) Editing boundaries:
- Prefer edits inside `web/`.
- Only edit repo root when a task explicitly targets docs/config at root (e.g., `TODO.md`, `AGENTS.md`, lint config).

4) If a command fails due to wrong directory:
- First fix is: `cd web` and re-run.

## Execution Workflow (tight loop)

Follow this cycle for each TODO section:
1) Identify the next checklist block in `..\TODO.md`.
2) Implement the smallest coherent slice that can be verified.
3) Verify with the appropriate command(s).
4) Summarize: files changed + commands run + next step.

Codex workflows should always include a reproduction/verification recipe when changing behavior. :contentReference[oaicite:1]{index=1}

## Planning for Complex Work (ExecPlans)

If a TODO item is a “complex feature” or “significant refactor” (multiple files, new architecture, or multi-hour work):
- Write an ExecPlan before implementing.

Place ExecPlans at: `F:\adaptech-design\PLANS.md` (append new plans as sections).
An ExecPlan must include:
- Goal
- Constraints
- Architecture (file tree + component map + data flow)
- Step-by-step execution plan
- Verification commands
- Rollback plan

This pattern is recommended for long-running Codex execution. :contentReference[oaicite:2]{index=2}

## Quality Gates (must pass)

For any meaningful change, run:
- Typecheck: `pnpm -C web typecheck` (or `pnpm -C web tsc --noEmit` if no script yet)
- Lint: `pnpm -C web lint` (or `pnpm -C web eslint .` if needed)
- Build (when relevant): `pnpm -C web build`
- Dev smoke test: `pnpm -C web dev` (manual quick check)

If tests/scripts do not exist yet, add them to `web/package.json` as part of the work.

## Engineering Standards (site is premium, bold, experimental)

Non-negotiables:
- Accessibility: keyboard-first, focus visible, ARIA where needed, reduced-motion support.
- Performance: avoid scroll jank, avoid layout thrash, lazy-load heavy media, keep animations performant.
- Maintainability: component boundaries, clear naming, avoid mega-components.
- SEO: proper metadata, semantic headings, structured data when applicable.

## Animation Standards

Default:
- Use Framer Motion for UI transitions and component micro-interactions.
- Use GSAP + ScrollTrigger only for sections that benefit from scroll storytelling.
- Always respect `prefers-reduced-motion` (reduce to minimal fades/transitions).
- Never add animation that blocks interaction or reduces readability.

## Content and CMS Standards

If/when integrating a headless CMS:
- Keep content types strongly typed.
- Validate inputs at boundaries (Zod or equivalent).
- Ensure previews and safe fallbacks for missing fields.
- Avoid hard-coding content that should be authored.

### Navigation Contract (required)

- Navigation is content-driven only:
  - `web/src/content/routes.ts` is the canonical route type source.
  - `web/src/content/nav.ts` is the canonical primary nav source.
- No ad-hoc nav arrays are allowed in components.
- All nav updates must go through content modules, not inline JSX lists.
- Header and overlay styles must use `web/src/components/site/headerStyles.ts`.
- Active-state logic must go through `resolveNavActive` in `web/src/lib/nav.ts`.

## File/Component Conventions (web/)

Preferred structure inside `web/src`:
- `src/app` (Next App Router)
- `src/components` (UI + sections)
- `src/lib` (utils, CMS client, fetchers)
- `src/styles` (globals, tokens)
- `src/content` (local content, if any)
- `src/assets` (static assets if not in /public)

Keep imports using the configured alias:
- `@/*` maps to `web/src/*`

## Codex Safety and Permissions

- Prefer granting additional directories via `--add-dir` rather than loosening sandbox settings. :contentReference[oaicite:3]{index=3}
- Network access should be used only when required (dependency install, fetch). Keep scope minimal.
- Never run destructive commands without clear justification and a confirmation step:
  - deleting files, rewriting git history, mass refactors without plan.

## How to Start a Session (recommended command)

When starting Codex for this repo, prefer:

- Start in the app directory:
  - `codex --cd "F:\adaptech-design\web" --add-dir "F:\adaptech-design"`

Then prompt:
- “Read `F:\adaptech-design\TODO.md` and implement top-to-bottom. After each top-level section, stop and summarize changes and verification commands.”

## “Stop Points” (when to pause)

Stop and ask for direction (or clearly surface options) if:
- A TODO item requires choosing between two architectures.
- A dependency choice affects long-term maintainability.
- A change risks regressions or significant churn without an ExecPlan.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/russell-henderson)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/russell-henderson)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
