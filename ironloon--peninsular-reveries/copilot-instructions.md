## peninsular-reveries

> `README.md` is the source of truth for game principles and site values.

# Project Guidelines

## Workflow

`README.md` is the source of truth for game principles and site values.

For project architecture, game quality standards, and testing conventions, load the `review` skill in `.pi/skills/review/` before doing substantial repo work.

## Planning Overlays

- When a plan touches visual assets, route art work through the `creative-assets` workflow or `scripts/generate-pixel-art.ts` with explicit iteration steps. Do not treat code agents as freeform art generators.
- Plans involving CSS visual design should include visual checkpoints for key states such as idle, animating, and complete.
- If NPCs share the player character's structure, specify whether NPCs animate identically to the player or remain static.
- If a plan uses `GameScreen` with a non-`.active` activation mechanism, explicitly state how the active screen overrides base `transform`, `visibility`, and hit-testing behavior.
- If a game includes a menu or settings modal, plan for a baseline home or quit path, controls help when gameplay exists, audio settings, reduce-motion control, and an info or credits path.

## Knowledge Persistence

- Record reusable project knowledge in the appropriate in-repo file. Do not rely on editor-specific memory paths.
- When you learn something reusable about this project, record it in the appropriate in-repo file instead:
  - Architecture, conventions, build and deploy patterns -> `.pi/skills/review/references/architecture.md`
  - Game quality, layout, pacing, visual rules -> `.pi/skills/review/references/game-quality.md`
  - Testing conventions, validation gates, CI behavior -> `.pi/skills/review/references/testing.md`
  - Workflow, session expectations, environment -> `AGENTS.md`
- Keep additions concise. If the insight is game-specific rather than project-wide, note the game slug inline.

## Styling Guidance

- This repo uses a hybrid CSS model. Prefer Remix `css()` mixins in `app/` for shared server-rendered UI, page layouts, shared game shell helpers, modal shells, and accessibility utilities.
- Keep `public/styles/*.css` for global foundation CSS and large game-specific visual systems, especially where client DOM code depends on stable class hooks or art-heavy selector or animation sections.
- Do not reintroduce duplicated per-game shell rules such as hiding site chrome, full-screen `main` layout, base screen transitions, base settings-modal overlay behavior, or `.sr-only` when `app/ui/game-shell.tsx` or other shared UI modules already own them.
- When updating a server-rendered component in `app/`, prefer co-located style objects or shared modules in `app/ui/` over adding new page-specific rules to `public/styles/main.css`.
- Treat `public/styles/main.css` as the global foundation stylesheet for tokens, theme overrides, theme toggle styling, and view-transition or reduced-motion glue.

## Session Expectations

- Default to human-ready completion for non-trivial feature work. Do not stop at code edits alone when the clear user intent is ready for me to try it. Continue through generated-file sync, repository validation, and low-risk regression fixes unless the user explicitly scopes the work smaller.
- Treat `pnpm test:local` as the preferred end-state check before handoff when the change is large enough to affect app behavior beyond a single isolated file.
- Treat secret hygiene as blocking release work. Do not leave real credentials, tokens, private keys, or committed `.env` secrets in the working tree or reachable history when the task is about review, release readiness, pushing, or security cleanup.
- If a validation run finds an objective blocker to human testing that was introduced or exposed by the current work, fix it before stopping when the fix is clear and low-risk.
- Ask the user a clarifying question only when the answer would materially change the implementation, or when proceeding would require a destructive, irreversible, or product-direction choice. If a reasonable default exists, use it and keep going.
- If the user says some version of continue until done, interpret done as human-ready unless genuinely blocked.
- When work is validated and human-ready, always finish end-to-end: stage the intended files, create a concise commit, push, and report the resulting short SHA to the user. Do this without another confirmation round unless the working tree is mixed or risky; in that case, stop short of commit or push and explain the blocker briefly.
- After completing work, report what changed, what was verified, and the short SHA of the commit that was pushed.

## Environment Context

- Dev OS: Windows
- CI OS: Ubuntu (`ubuntu-latest`)
- Shell: PowerShell. Set `[Console]::OutputEncoding = [System.Text.Encoding]::UTF8` at session start to avoid mojibake with Unicode characters.
- Node: `24.14.1`
- pnpm: `10.33.0`
- npm: `11.12.1` (bootstrap only)
- Editor: Pi terminal coding agent with pi-subagents extension
- GitHub CLI: `gh` is available and should be assumed authenticated. If auth fails, help the user recover and use `gh --help` for command discovery.
- Pi agents: `.pi/agents/gnd-diver.md` (implementation). Use `/run gnd-diver` to invoke.
- Pi skills: `.pi/skills/` — `review`, `creative-assets`, `gnd-chart`, `gnd-critique`, `gnd-navigate`. Load via `/skill:name`. Use `/skill:gnd-navigate` to dispatch a plan to gnd-diver subagents.
- No Docker or container workflow is expected. Use direct Node execution.

## Key Commands

- `pnpm dev` -> local dev server
- `pnpm check` -> lint and typecheck
- `pnpm test:local` -> full local validation gate
- `pnpm build` -> static build
- `pnpm sync:attributions` -> regenerate `ATTRIBUTIONS.md`

`pnpm install` configures the repo-owned pre-commit hook, and the hook runs `pnpm test:local` automatically.
Prefer `pnpm exec <tool>` over `npx <tool>` for repo-local CLIs. If install scripts are blocked, review the dependency and then use `pnpm approve-builds` or update `onlyBuiltDependencies` in `pnpm-workspace.yaml`.

---
> Source: [ironloon/peninsular-reveries](https://github.com/ironloon/peninsular-reveries) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
