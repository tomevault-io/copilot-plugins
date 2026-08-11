## nightmarenet

> NightmareNet is an **Autonomous AI Self-Improvement Platform** using a sleep-inspired training paradigm. It forces neural networks to learn invariant structures through 4-phase cycles: Wake → Dream → Nightmare → Compress.

# NightmareNet — Agent Instructions

## Project Overview

NightmareNet is an **Autonomous AI Self-Improvement Platform** using a sleep-inspired training paradigm. It forces neural networks to learn invariant structures through 4-phase cycles: Wake → Dream → Nightmare → Compress.

**Stack**: Python ≥3.9, PyTorch, Transformers, FastAPI, Next.js 14 (frontend)

## Repository Structure

```
nightmarenet/           # Core Python package
  api/                  # FastAPI endpoints (app.py, schemas.py)
  compression/          # Knowledge distillation
  data/                 # Dataset loading & processing
  distortions/          # Dream & nightmare distortion engines
  evaluation/           # Robustness evaluation & metrics
  training/             # Training loops, DDP, experiment tracking
  utils/                # Config, early stopping, type hints
frontend/               # Next.js 14 + Framer Motion + Tailwind
  src/components/       # React components (cyberpunk UI)
  src/lib/api.ts        # TypeScript API client
tests/                  # pytest suite (206+ tests)
configs/                # YAML training configs
scripts/                # CLI entry points
```

## Critical Build & Test Commands

```bash
# Activate venv (ALWAYS use .venv312 — Python 3.12 + CUDA)
.venv312\Scripts\Activate.ps1       # Windows
source .venv/bin/activate            # Unix

# Run all tests
pytest tests/ -v --tb=short

# Run specific test module
pytest tests/test_distortions.py -v

# Lint (must pass with 0 errors)
ruff check .

# Type check
mypy nightmarenet/

# Start API server (--env-file ensures .env propagates to reload workers)
uvicorn nightmarenet.api.app:app --reload --port 8000 --env-file .env

# Frontend
cd frontend && npm run dev           # Dev server on :3000
cd frontend && npm run build         # Production build
```

## Code Style & Conventions

- **Line length**: 100 chars
- **Ruff rules**: E, F, W, I, N, UP, B (ignore UP007, UP045)
- **Imports**: isort via ruff — stdlib, third-party, local (alphabetical within groups)
- **Type hints**: Use `Union[X, Y]` not `X | Y` (Python 3.9 compat)
- **DO NOT** use `from __future__ import annotations` in any file that uses FastAPI `Body(...)` — it breaks Pydantic v2 at runtime
- **Docstrings**: Google style, only on public APIs
- **Error handling**: Raise with context (`raise X from e`), not bare `raise`

## Architecture Constraints

1. **No `from __future__ import annotations`** in `nightmarenet/api/` — Pydantic v2 + FastAPI incompatibility
2. **Python 3.9 minimum** — no walrus operator assumptions, use `Union[X, Y]` over `X | Y`
3. **Optional deps are optional** — guard imports with try/except: `accelerate`, `wandb`, `tensorboard`
4. **FastAPI Body singletons** — `_DISTORTION_BODY` and `_ROBUSTNESS_BODY` are module-level to avoid B008
5. **CORS** — configured via `NIGHTMARENET_CORS_ORIGINS` env var (default: `*`)
6. **Health** — optional `NIGHTMARENET_HEALTH_TEST_COUNT=1` enables subprocess `pytest --collect-only` in `/api/v1/health` (off by default; avoid in production)
7. **Pipeline registry** — `NIGHTMARENET_MAX_PIPELINE_RUNNERS` caps in-memory runners (default 64); completed runs are evicted first
8. **Next.js proxy** — `frontend/next.config.ts` uses `NEXT_API_REWRITE_URL` for where `/api/`* rewrites; see `frontend/README.md`

## Testing Requirements

- **206+ tests passing** — never commit code that reduces the test count
- Run `pytest` before claiming any task is complete
- Run `ruff check .` before claiming lint cleanliness
- Tests live in `tests/` — mirror the package structure
- Use `monkeypatch` for env vars, never modify `os.environ` directly

## API Endpoints


| Method | Path                          | Purpose                                       |
| ------ | ----------------------------- | --------------------------------------------- |
| GET    | `/api/v1/health`              | Health check (status, version, tests)         |
| POST   | `/api/v1/generate/dream`      | Dream distortion `{text, strength, seed}`     |
| POST   | `/api/v1/generate/nightmare`  | Nightmare distortion `{text, strength, seed}` |
| POST   | `/api/v1/evaluate/robustness` | Multi-strength eval `{text, strengths[]}`     |


## Frontend Notes

- **Framework**: Next.js 14 (App Router), TypeScript, Tailwind CSS v4
- **Design**: Cyberpunk dark theme — void (#020617), dream (#818CF8), nightmare (#EF4444), neural (#06B6D4)
- **Animations**: Framer Motion throughout
- **API client**: `frontend/src/lib/api.ts` — typed client for all 4 endpoints
- **Key constraint**: Tailwind v4 uses `@theme inline` block, not `tailwind.config.js`

## Git Workflow

- Conventional commits: `feat:`, `fix:`, `docs:`, `test:`, `refactor:`, `chore:`
- Run tests + lint before any commit
- Keep commits atomic — one concern per commit

## Known Issues & Gotchas

- `Accelerator` import is optional — guarded with `Accelerator: Any = None` fallback
- `evaluator._log_eval()` guards against `self.tracker is None`
- Rate limiting on API uses `slowapi` — 60 req/min on generate, 10 req/min on evaluate

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **NightmareNet** (4310 symbols, 7685 relationships, 161 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> If any GitNexus tool warns the index is stale, run `npx gitnexus analyze` in terminal first.

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `gitnexus_impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `gitnexus_detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `gitnexus_query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `gitnexus_context({name: "symbolName"})`.

## Never Do

- NEVER edit a function, class, or method without first running `gitnexus_impact` on it.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis.
- NEVER rename symbols with find-and-replace — use `gitnexus_rename` which understands the call graph.
- NEVER commit changes without running `gitnexus_detect_changes()` to check affected scope.

## Resources

| Resource | Use for |
|----------|---------|
| `gitnexus://repo/NightmareNet/context` | Codebase overview, check index freshness |
| `gitnexus://repo/NightmareNet/clusters` | All functional areas |
| `gitnexus://repo/NightmareNet/processes` | All execution flows |
| `gitnexus://repo/NightmareNet/process/{name}` | Step-by-step execution trace |

## CLI

| Task | Read this skill file |
|------|---------------------|
| Understand architecture / "How does X work?" | `.claude/skills/gitnexus/gitnexus-exploring/SKILL.md` |
| Blast radius / "What breaks if I change X?" | `.claude/skills/gitnexus/gitnexus-impact-analysis/SKILL.md` |
| Trace bugs / "Why is X failing?" | `.claude/skills/gitnexus/gitnexus-debugging/SKILL.md` |
| Rename / extract / split / refactor | `.claude/skills/gitnexus/gitnexus-refactoring/SKILL.md` |
| Tools, resources, schema reference | `.claude/skills/gitnexus/gitnexus-guide/SKILL.md` |
| Index, status, clean, wiki CLI commands | `.claude/skills/gitnexus/gitnexus-cli/SKILL.md` |

<!-- gitnexus:end -->

---
> Source: [Adit-Jain-srm/NightmareNet](https://github.com/Adit-Jain-srm/NightmareNet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
