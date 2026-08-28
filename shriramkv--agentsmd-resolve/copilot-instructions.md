## agentsmd-resolve

> Guidance for coding agents working in the `agentsmd-resolve` repository.

# AGENTS.md

Guidance for coding agents working in the `agentsmd-resolve` repository.

## What this is

A stdlib-only, Python 3.9+ tool that computes the effective AGENTS.md for a path
by merging the nested chain of AGENTS.md files. No third-party runtime
dependencies; keep it dependency-free.

## Layout

- `agentsmd_resolve/discover.py` - find AGENTS.md files, build the governing chain.
- `agentsmd_resolve/resolve.py` - parse files into blocks, merge with nearest-wins,
  detect overrides, expose explain and require checks. No I/O beyond reading files.
- `agentsmd_resolve/report.py` - console, JSON, Markdown renderers.
- `agentsmd_resolve/cli.py` - argparse CLI and exit-code policy.
- `agentsmd_resolve/sample.py` - builds a nested sample repo for demo and tests.
- `tests/` - stdlib `unittest`, no external runner.

## Build and test

```bash
python -m unittest discover -s tests -v
python -m agentsmd_resolve --demo                       # smoke test
python -m agentsmd_resolve --root . --path agentsmd_resolve
```

CI runs the same on Python 3.9 to 3.12; nothing to install.

## Conventions

- Merge semantics: the nearest AGENTS.md wins per heading; farther definitions
  are overridden, not discarded silently, and must show up in the overrides list.
- Survival of provenance matters: every block in the effective output must be
  traceable to a source file and depth.
- Keep discovery, resolution, rendering, and the CLI in separate modules.
- British spelling in prose; no em dashes.

## Security

This tool only reads files under a repository root the user points it at. It
does not execute repository code or fetch anything over the network.

---
> Source: [shriramkv/agentsmd-resolve](https://github.com/shriramkv/agentsmd-resolve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
