## ds-skill-extraction-workshop

> <!-- BEGIN:dry-run-worktree-rules -->

<!-- BEGIN:dry-run-worktree-rules -->
# Dry-runs use their own worktree

Before producing snapshot artefacts (`/extract-ds-skill`, a PRD build `/primer-react implement prompts/*.md`, `prompts/issues.md`, `prompts/audit.md`, anything frozen under `dry-runs/`): run `git worktree list`. Outputs go in the matching `.claude/worktrees/dryrun-NN-<label>-iN/` (branch `dryrun/NN-<label>-iN`) — never a durable branch.

Each worktree gets its **own** `node_modules` — after creating it, run `pnpm install --frozen-lockfile` inside the worktree once. **Never symlink `node_modules` to root.** A shared `node_modules` isn't isolation: a single `pnpm install` in any one worktree rewrites root's top-level links to point *into* that worktree (we've watched root's `next`/`react`/`react-dom` get redirected into a sibling dryrun), and then `git worktree remove`-ing that worktree dangles the links and breaks the build for root and *every* worktree at once. pnpm hardlinks from the global store (`~/Library/pnpm/store`), so a per-worktree install is fast and costs almost no disk. `package.json` / `pnpm-lock.yaml` / `pnpm-workspace.yaml` are git-tracked, so the install is self-contained — Turbopack infers the worktree as its own root and has no symlink to distrust.

## Where a change goes

- **Dry-run-specific → dryrun branch**: `dry-runs/<date>-<label>/` snapshots (app files + `harvest-log.md`), generated `app/` artefacts, a freshly-extracted `.claude/skills/<slug>/` not yet promoted.
- **Durable → feature branch → PR → `main`**: `.claude/skills/extract-ds-skill/`, the promoted reference skill `.claude/skills/primer-react/` (lives on main; repaired via PR; copied read-only into worktrees as skill-under-test), `dry-runs/TEMPLATE.md`/`README.md`/`SUMMARY.md`, this file.

A freshly-extracted skill is dry-run-specific until promoted to main; after that, durable.

Unsure? Ask before writing — re-splitting commits later is worse.
<!-- END:dry-run-worktree-rules -->

---
> Source: [vercel-labs/ds-skill-extraction-workshop](https://github.com/vercel-labs/ds-skill-extraction-workshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
