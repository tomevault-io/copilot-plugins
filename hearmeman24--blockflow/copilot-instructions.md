## blockflow

> Local-only pipeline UI for submitting video/image generation jobs to RunPod serverless endpoints.

# sgs-ui

Local-only pipeline UI for submitting video/image generation jobs to RunPod serverless endpoints.

## Tech Stack

- **Frontend**: Next.js 16, React 19, shadcn/ui, Tailwind CSS (dark theme only)
- **Backend**: FastAPI, uvicorn
- **Launch**: `uv run app.py` starts both FastAPI (:8000) and Next.js (:3000)

## Pipeline System

The `/generate` page uses a linear left-to-right pipeline with a tree branching model.

- **Block** is the canonical term (not node, step, or stage)
- One global "Run Pipeline" button — no per-block actions
- Accumulator data model: outputs collected by `PortKind`, resolved as inputs to downstream blocks
- Execute functions receive fresh `inputs` parameter from the pipeline runner

## Adding a Block

1. Create `custom_blocks/<slug>/frontend.block.tsx` exporting `blockDef: BlockDef`
2. Optionally add `custom_blocks/<slug>/backend.block.py` exporting `router: APIRouter`
3. Registration is automatic via codegen (`npm run predev`)

## Block Sizes

sm (280x220, blue), md (360x320, emerald), lg (440x460, violet), huge (540x580, amber)

## Key Files

| File | Purpose |
|------|---------|
| `app.py` | Single entrypoint, starts FastAPI + Next.js |
| `frontend/src/lib/pipeline/` | Registry, types, pipeline-context, tree-utils |
| `frontend/src/components/pipeline/` | Pipeline view, block card, chain renderer |
| `custom_blocks/` | Self-contained block definitions |
| `backend/main.py` | FastAPI app, auto-loads block sidecars |
| `backend/routes.py` | Shared routes: flows + runs only |

## Conventions

- Dark theme only (shadcn/ui, `class="dark"` on `<html>`)
- URL-state routing: filters/sort in URL search params
- Block API routes: `/api/blocks/<slug>/...` only
- No Playwright testing — user tests manually, use `npm run build` for verification

<!-- BEGIN BEADS INTEGRATION v:1 profile:minimal hash:7510c1e2 -->
## Beads Issue Tracker

This project uses **bd (beads)** for issue tracking. Run `bd prime` to see full workflow context and commands.

### Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --claim  # Claim work
bd close <id>         # Complete work
```

### Rules

- Use `bd` for ALL task tracking — do NOT use TodoWrite, TaskCreate, or markdown TODO lists
- Run `bd prime` for detailed command reference and session close protocol
- Use `bd remember` for persistent knowledge — do NOT use MEMORY.md files

**Architecture in one line:** issues live in a local Dolt DB; sync uses `refs/dolt/data` on your git remote; `.beads/issues.jsonl` is a passive export. See https://github.com/gastownhall/beads/blob/main/docs/SYNC_CONCEPTS.md for details and anti-patterns.

## Session Completion

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds
<!-- END BEADS INTEGRATION -->

---
> Source: [Hearmeman24/BlockFlow](https://github.com/Hearmeman24/BlockFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
