## low-hanging-fruit

> Build a local-first tool that discovers, explains, ranks, replays, and verifies

# Low-Hanging Fruit contributor instructions

## Mission

Build a local-first tool that discovers, explains, ranks, replays, and verifies
research opportunities in mathematics and LLM research. Keep claims auditable
and useful to non-specialists.

## Product invariants

- The product name is **Low-Hanging Fruit**, the import package is
  `low_hanging_fruit`, the CLI is `lhf`, and the Skill is `low-hanging-fruit`.
- Do not use the rejected legacy product name outside a migration regression
  test or an explicitly labelled migration note.
- `.agents/skills/low-hanging-fruit/` is the canonical Skill source. Run
  `uv run python scripts/sync_skill.py --check` before finishing Skill changes.
- Keep English and Chinese user-facing promises, commands, safety boundaries,
  prompts, output contracts, and Campaign output in sync.
- Stable Pydantic models forbid extra fields. Update checked-in JSON Schemas,
  fixtures, tests, and both READMEs when a stable contract changes.
- LHF Rank is a deterministic prioritization heuristic, never a probability.
  Keep discovery and verification difficulty separate.
- Never execute untrusted external repositories, paper commands, or
  model-generated code. Never automatically publish a research claim.
- The default Codex bridge uses `workspace-write`, argument-list subprocesses,
  timeouts, structured output, and the user's existing Codex login. Never read
  or copy Codex authentication files.

## Development

Recommended setup and checks:

```bash
uv sync --all-extras --dev
uv run ruff format --check .
uv run ruff check .
uv run mypy src
uv run pytest -q
uv run lhf demo --output-dir .tmp/demo
uv run lhf validate .tmp/demo
uv build
```

Convenience targets are also available through `make setup`, `make lint`,
`make test`, `make demo`, and `make build`. A behavior change is incomplete
until its tests and public documentation are updated.

Before completing work, run the full checks above, `git diff --check`, inspect
the complete diff, and verify that local notes, credentials, caches, run output,
and private paths are not tracked.

---
> Source: [cyanseek/low-hanging-fruit](https://github.com/cyanseek/low-hanging-fruit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
