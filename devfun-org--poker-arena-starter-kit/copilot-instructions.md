## poker-arena-starter-kit

> Conventions for any coding agent (Cursor, Codex CLI, Claude Code,

# AGENTS.md — Arena Starter Kit

Conventions for any coding agent (Cursor, Codex CLI, Claude Code,
Aider, GitHub Copilot, OpenHands, Continue, Windsurf, etc.) working
inside this repo.

> Product label is **Arena Starter Kit**. The CLI command stays
> `pokerkit` (e.g. `./pokerkit run`). Don't rename the CLI; don't
> use "PokerKit" as a product name (it's the upstream Python engine).

For the end-to-end "build me a poker bot" workflow, read `SKILL.md` —
that's the canonical agent entrypoint. This file is for project-level
conventions you should follow whenever you edit code here.

## Project shape

- **Purpose**: a starter kit for poker agents on dev.fun Arena's Poker
  Eval benchmark (a public head-to-head benchmark vs 5 server-side
  reference bots; no claim URL, no invitations, no entry fee).
- **Two paths share the same code**:
  - Local dev loop — `pokerkit test`, `pokerkit selfplay`,
    `pokerkit run --dry-run`. Fast iteration on `decide()`.
  - Arena Evaluation — `pokerkit run`. Real benchmark.
- **The only file users edit is `examples/agent.py`** (specifically the
  `decide()` function at ~line 168). Everything else is glue.

## File layout (what to touch, what not to)

```
SKILL.md                      ← agent entrypoint; edit when changing dev loop
AGENTS.md                     ← this file
README.md                     ← human-facing intro; brief

examples/                     ← scripts (CLI black boxes for the agent)
  agent.py                    ← ★ EDIT THIS (decide() at ~line 168)
  cli.py                      ← `pokerkit` command dispatcher
  selfplay.py                 ← local headless self-play vs simple bots
  analyze.py                  ← Arena failure report
  replay.py                   ← HTML replay viewer
  arena_client.py             ← HTTP client (rarely touch)
  mock.py                     ← --dry-run scaffolding
  llm_agent.py                ← Level 5 runtime-LLM decide() (model-agnostic: Anthropic/OpenAI/compat)
  testing.py                  ← 20 scenario fixtures
  research_static_chart.py    ← Auto Research example
  skeletons/                  ← always_fold / always_call / random_action
  STRATEGY.md.template        ← strategy template (copy to root as STRATEGY.md)
  prompt.md                   ← legacy copy-paste prompt (kept for reference)

references/                   ← detail docs loaded on demand by the agent
  poker-eval-arena.md
  decide-function.md
  reasoning-yaml.md
  heuristic-learning.md

assets/                       ← decide() reference implementations
  decide_baseline.py
  decide_ranged.py
  decide_textured.py

docs/                         ← human-facing strategy / play.md
tests/                        ← pytest suite (must all pass before any commit)

.env.example                  ← copy to .env
pyproject.toml                ← uv-managed, version pinned
pokerkit                      ← shell wrapper at repo root
```

## Hard rules

1. **Never push to GitHub** unless the user explicitly asks. This is
   the user's repo, not yours.
2. **`tests/` must always pass** (`uv run pytest tests/ -q`). 18 tests
   covering 20 scenario fixtures today. If you add functionality, add
   tests. If they fail, fix them before considering the work done.
3. **Don't add dependencies** beyond what's in `pyproject.toml`
   without asking. `httpx`, `python-dotenv`, `treys`, `pokerkit` are
   the four core deps; `anthropic` and `openai` are optional `[llm]`
   extras.
4. **Reasoning YAML must be ≤150 chars** on every action submission.
   The format is in `references/reasoning-yaml.md`. If your computed
   YAML overflows, fall back to a known-valid short object — never
   blind-slice to 150.
5. **`amount` semantics**: total chips committed on this street after
   acting (NOT increment). The API will 400 if you send a delta.
6. **Default to L1 heuristic.** Don't call an LLM at runtime unless the
   user explicitly enables the Level 5 runtime-LLM path
   (`examples/llm_agent.py`, cost ~$0.02/decision, ~$60/match).
7. **Introspect at startup.** Call `GET /__introspection` after auth
   and verify endpoints. Read terminal phase/status enums from the
   schema — do NOT hardcode `{"completed","cancelled","failed"}`.

## Where decisions live

| Question | Source of truth |
|---|---|
| What `decide()` should return | `references/decide-function.md` |
| Schema of the live API | `GET /api/arena/__introspection` (call it!) |
| Action enums, phase enums, terminal states | introspection response, not hardcoded |
| Reasoning YAML format | `references/reasoning-yaml.md` |
| When to use L2 / HL / L1 | `references/heuristic-learning.md` |
| Heuristic Learning loop philosophy | `docs/strategy.md` + `references/heuristic-learning.md` |
| Failure analysis output format | `examples/analyze.py` (run it, read output) |

## Commands you'll run a lot

```bash
./pokerkit test                            # 20 unit fixtures, ~50 ms
./pokerkit selfplay --hands 200 --seed 42  # local bots, ~1 s
./pokerkit run --dry-run --max-hands 1     # offline smoke, ~30 s
./pokerkit run --max-hands 50              # Arena preview, ~3-5 min
./pokerkit analyze --out failure_report.txt
./pokerkit replay --latest

uv run pytest tests/ -q                    # run before commit
python -m py_compile examples/agent.py     # quick syntax check
```

## Coding style

- Python 3.11+, type hints encouraged but not required.
- Pure functions where possible; `decide()` MUST be pure (same input →
  same output) so unit tests are reliable.
- Print messages prefixed with `[arena-pokerkit]` for runtime logs.
- Atomic file writes for `.arena-credentials` and `.arena-poker-state`
  (already implemented in `arena_client.py`).
- Keep diffs small. If you need a helper, put it inside the file that
  uses it first. Break into a package only when ≥2 files need it.

## When in doubt

Re-read `SKILL.md`. It tells you the end-to-end flow and the
ask-vs-act boundary. If the user is asking you to do something
`SKILL.md` says you should ASK about, ask.

---
> Source: [devfun-org/poker-arena-starter-kit](https://github.com/devfun-org/poker-arena-starter-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
