## openwiki-docs-follow-features

> Keep OpenWiki docs (docs/) accurate to this branch whenever features change


# OpenWiki docs follow features

When you add, change, or remove product behaviour in this repo, update OpenWiki documentation in the **same change**.

## Where docs live

- Pages: `docs/**/*.md` (also reachable as `openwiki/` via symlink to `docs/`)
- Brief / checklist: `docs/INSTRUCTIONS.md`
- Agent pointers: `AGENTS.md`, `CLAUDE.md`

## Rules

1. **Branch accuracy** — Document only what exists on the current branch/tree. Do not copy claims from other feature branches (e.g. multi-agent `AGENTIC_PIPELINE` default if this tree has no such product path).
2. **Same change** — Feature code + matching doc updates land together (do not leave docs for a follow-up unless the user explicitly defers).
3. **Map updates** — Touch the right page(s); add a new `docs/` page and link it from `docs/quickstart.md` when a feature needs a deep dive.
4. **Verify** — After edits, ensure relative links resolve and remove stale sections that describe deleted behaviour.

## Quick map

| Code area | Doc page |
|-----------|----------|
| Routes / SSE in `api/app.py` | `docs/api/surface.md` |
| `api/static/` | `docs/webui/integration.md` |
| Sandbox / `orchestrator.py` / `agentic_debug.py` | `docs/sandbox/debugging.md`, `docs/agentic-debug-pipeline.md` |
| Env / Docker / serve scripts | `docs/building/configuration.md`, `docs/building/docker-deployment.md` |
| `tests/development/` | `docs/testing/overview.md` |
| Failure modes / ops | `docs/operations/runbooks.md` |

---
> Source: [zakaseb/icd-c-code-refactorer](https://github.com/zakaseb/icd-c-code-refactorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
