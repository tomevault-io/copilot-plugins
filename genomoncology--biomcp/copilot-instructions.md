## biomcp

> BioMCP is a Rust CLI and MCP server with a Python docs/contract harness. The

# AGENTS.md — BioMCP

BioMCP is a Rust CLI and MCP server with a Python docs/contract harness. The
Rust runtime lives under `src/`; Python tests, docs checks, and executable
contracts exercise the shipped CLI/MCP surface.

## Repository contract shape

- Behavioral contracts are mustmatch executable docs at:

  ```
  spec/*.md
  ```

  Run them with `make spec`. The spec gates use the standalone `mustmatch test`
  binary through `scripts/run-specs.sh`; standup belongs in that helper script,
  not in ad hoc docs.
- Unit/static layers run with `make test`: Rust nextest first, then the Python
  CLI/MCP/docs contract lane.

## Contract quick reference

spec/*.md
make lint
make test
make spec
rust-standards
python-standards
cli-design
mustmatch
testing-mindset

## Gates

Run the standard gates directly:

```
make lint
make test
make spec
```

There is intentionally no `make check` convention. `make lint` includes the
quality ratchet, `make test` includes both Rust and Python contract tests, and
`make spec` is the routine mustmatch contract gate.

## Skill rail

Use this hybrid repo rail for dispatched work:

- rust-standards
- python-standards
- cli-design
- mustmatch
- testing-mindset

## Hygiene

Do not commit secrets, PHI, absolute local paths, planning notes, or March
runtime artifacts. Keep `.march/` runtime state, `.march-runtime/`, local caches,
and generated build outputs out of git.

---
> Source: [genomoncology/biomcp](https://github.com/genomoncology/biomcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
