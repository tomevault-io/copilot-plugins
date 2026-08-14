## dhscanner-runner

> This is the top-level `dhscanner.vps` repo (the optimized backend / orchestration

# AGENTS

This is the top-level `dhscanner.vps` repo (the optimized backend / orchestration
layer for dhscanner). Most of the actual language-handling code lives in the
`dhscanner` git submodule and, transitively, in its own submodules.

## Grammar / parser-coverage work

For anything related to **improving parser coverage** (adjusting `.y` grammar
files, smart constructors in `TsParserActions.hs`, the per-iteration
regression / strict-progress gate, the agent-driven `agent_loop.py` workflow,
etc.) the canonical guide is **not** this file — it is the `AGENTS.md` shipped
inside the parsers submodule:

- [`dhscanner/dhscanner.1.parsers/AGENTS.md`](dhscanner/dhscanner.1.parsers/AGENTS.md)

That document owns:

1. The coverage goal and the regression-repo methodology.
2. The "one rule adjustment at a time" iteration contract and the strict
   progress gate (every previously-passing file still passes; every previously
   failing file now passes or fails at a strictly greater column; at least
   one file strictly progresses).
3. Guardrails (`cabal build` succeeds, Happy reports no new shift/reduce or
   reduce/reduce conflicts vs the previous baseline).
4. The active cleanup conventions inside `TsParser.y` /
   `TsParserActions.hs` (camelCase = handled, snake_case = TODO; one-line
   mapping comment; reusable list/option helpers; etc.).
5. The mapping from native TS nodes to the normalized **dhscanner.ast**
   (direct vs instrumented rules, and the `<dhscanner-instrumentation>[<tag>]`
   callee-name convention routed through `Actions.instrumentationCall`).

The companion **stand-alone dev stack** for this work also lives in that
submodule (`dhscanner/dhscanner.1.parsers/compose.yaml`). It is intentionally
separate from the prebuilt stack used by this repo's `compose/compose.*.yaml`
files: it builds parsers from source and binds host ports `4000` / `4001` so
it can coexist with the prebuilt vps stack on the same machine. The
agent-driven loop (`dhscanner/dhscanner.1.parsers/agent_loop.py`) drives that
stack — it performs `docker compose stop parsers` / `build parsers` /
`up -d parsers` / `/healthcheck` per iteration on its own; you only do the
one-time bring-up by hand:

```powershell
cd dhscanner/dhscanner.1.parsers
$env:OPENAI_API_KEY = 'sk-...'
docker compose up -d --build
python agent_loop.py
```

**Do not** duplicate the parser-coverage rules here — edit them in the
submodule's `AGENTS.md` so they stay co-located with the grammar they govern.

## Everything else

For changes that are genuinely top-level (the FastAPI app under
`dhscanner.infra/app/`, the
CLI in `cli/`, the workers, the coordinator, the compose files under
`compose/`, etc.) there is currently no extra agent contract beyond the
project's normal lint / type-check / test CI:

- `pylint` (see `.github/workflows/pylint.yaml`)
- `mypy` (see `.github/workflows/mypy.yaml`)
- `tests` (see `.github/workflows/tests.yaml`)

If a sub-area grows its own per-iteration discipline (the way the parsers
submodule has), prefer adding an `AGENTS.md` next to that code and linking
it from here, rather than expanding this file.

---
> Source: [OrenGitHub/dhscanner.runner](https://github.com/OrenGitHub/dhscanner.runner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
