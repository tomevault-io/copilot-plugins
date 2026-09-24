## ai-maestro

> > This is the **AI Maestro kit** itself (the framework), not a project managed by it.

# AI Maestro — repo guidance

> This is the **AI Maestro kit** itself (the framework), not a project managed by it.

## What lives here

| Path | What it is |
| --- | --- |
| `board/` | This repo's own real workboard (`data.json`/`archive.json`/`plan.json`/`plan.md`/`specs/`), plus the board and plan formats (`board.schema.json`, `plan.schema.json`, `README.md`). The two are separable on purpose: the live data (`data.json`/`archive.json`/`plan.json`/`plan.md`/`specs/`/`reports/`) is **git-ignored and local-only** — it never gets pushed to the public repo — and `npm pack` excludes it too (see `package.json`'s `files`) so it never ships or gets vendored into a new project. `maestro setup` seeds a fresh project's board and plan from `starters/orchestrated-project/board/` instead. CI and `prepublishOnly` therefore validate the starter board, not this one. |
| `agents/` | The generic agent roster (source of truth; projects render copies). |
| `skills/` | Reusable skills (source of truth; projects render copies). |
| `render/sync.mjs` | Generates native Claude Code and Codex files from one config + context. |
| `scripts/validate-board.mjs` | Board integrity check. |
| `starters/` | Capsules users copy into their repos. |
| `docs/` | The method and guides. |

## Rules for changing the kit

| Rule | Detail |
| --- | --- |
| **Product-neutral** | No company/vendor/cloud specifics in the core — those belong in a project's `context.md` or in examples. |
| **Runtime parity** | Keep canonical `agents/` and `skills/` runtime-neutral; renderer and setup changes must remain usable in both Claude Code and Codex. |
| **Frontmatter** | Agents and skills carry `name` + `description` frontmatter. Keep roles non-overlapping. |
| **Validate the board(s)** | After touching either: `node scripts/validate-board.mjs board/data.json` (this repo's own) and/or `node scripts/validate-board.mjs starters/orchestrated-project/board/data.json` (the starter new projects are seeded from). It checks the plan's scope gate too. |
| **Never hand-edit a board or plan** | `board/data.json` goes through `maestro ticket` (`scripts/board-write.mjs`); `board/plan.json` goes through `maestro plan` (`scripts/plan-write.mjs`). Both take the same directory lock, validate, and write atomically — never read-file/edit-file/write-file. `maestro ticket import` bulk-adds a planned board in one write; it only ever ADDS. `board/plan.md` is **generated** from `plan.json` on every plan write: edits to it are discarded. |
| **Deterministic renderer** | Keep it deterministic — the lock file must not depend on timestamps. |
| **Version** | **Never bump it in a PR** — CI rejects that. The version belongs to a *release*, not a change: on `main`, `npm version patch` bumps `package.json`, syncs `VERSION`, commits, and tags in one step. See [Releasing](./CONTRIBUTING.md#releasing). |

---
> Source: [my-chiefmind/ai-maestro](https://github.com/my-chiefmind/ai-maestro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
