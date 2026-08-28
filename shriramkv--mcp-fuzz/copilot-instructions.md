## mcp-fuzz

> Guidance for coding agents working in the `mcp-fuzz` repository.

# AGENTS.md

Guidance for coding agents working in the `mcp-fuzz` repository.

## What this is

A stdlib-only, Python 3.9+ robustness harness that fuzzes MCP servers over stdio
and grades them A to F. No third-party runtime dependencies. Keep it that way:
if a change needs a dependency, reconsider the change.

## Layout

- `mcp_fuzz/transport.py` - spawn a server subprocess, send raw/JSON lines,
  read with timeouts on a background thread, detect crashes.
- `mcp_fuzz/handshake.py` - MCP `initialize` sequence and the liveness probe.
- `mcp_fuzz/cases.py` - the fuzz-case catalogue. Add new cases here.
- `mcp_fuzz/runner.py` - runs cases, records observations. No grading logic.
- `mcp_fuzz/grade.py` - turns observations into weighted scores and a letter.
- `mcp_fuzz/report.py` - console, JSON, Markdown, shields.io badge.
- `mcp_fuzz/cli.py` - argparse CLI and exit-code policy.
- `mcp_fuzz/sample_server.py` - robust demo server (must keep scoring A).
- `mcp_fuzz/fragile_server.py` - intentionally broken server (must keep crashing).
- `tests/` - stdlib `unittest`, no external test runner required.

## Build and test

```bash
python -m unittest discover -s tests -v      # run all tests
python -m mcp_fuzz --sample                  # smoke test (expect grade A)
python -m mcp_fuzz --fragile --fail-on-crash # expect a crash and exit 1
```

CI runs the same on Python 3.9 to 3.12; there is nothing to install.

## Conventions

- Separation of concerns is deliberate: the runner only observes, `grade.py`
  only scores, `report.py` only renders. Do not compute grades in the runner.
- A new fuzz case is a `Case` in `cases.py` with a `send(server, ctx)` function
  and an `expects` of `"error"`, `"notification"`, or `"tolerant"`. Give it a
  unique dotted id (`category.short_name`) and, for `"error"` cases, the set of
  acceptable JSON-RPC codes.
- The sample server must continue to score A and the fragile server must
  continue to be detected as a crash. Both are asserted in `tests/`.
- Survival must always dominate the grade. A server that crashes or hangs
  should never score well regardless of other dimensions.
- British spelling in prose; no em dashes.

## Safety

This is a defensive testing tool for a server you own. It sends only
protocol-level malformed input over stdio to exercise error handling. Do not
extend it toward attacking third-party services or hosts.

---
> Source: [shriramkv/mcp-fuzz](https://github.com/shriramkv/mcp-fuzz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
