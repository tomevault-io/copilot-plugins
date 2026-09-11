## agentgateway-diff

> Guidance for AI coding agents working in this repository.

# AGENTS.md

Guidance for AI coding agents working in this repository.

## What this project is

`agentgateway-diff` compares two agentgateway configuration files and reports
the security-relevant drift between them: authentication removed or weakened,
CORS opened, rate limits dropped, TLS turned off, admin surfaces exposed, new
unauthenticated routes, and newly hardcoded secrets. It grades the change A-F
and exposes CI gates so a config change that quietly opens a hole fails the
build.

## Ground rules

- Standard library only for the runtime. PyYAML is an optional accelerator;
  the vendored `miniyaml.py` parser must remain the zero-dependency fallback,
  and both paths must stay tested.
- Target Python 3.9+.
- Every new detection rule needs a test in `tests/` and, where useful, a case
  in the bundled `samples/`.
- Keep detection conservative about false positives, but never downgrade the
  severity of an auth removal, a TLS disable, or an admin exposure below
  `critical`.

## Layout

- `agentgateway_diff/surface.py` extracts a normalized security surface.
- `agentgateway_diff/diff.py` computes changes and classifies severity.
- `agentgateway_diff/grade.py` maps regressions to an A-F score.
- `agentgateway_diff/report.py` renders console/json/markdown/badge.
- `agentgateway_diff/cli.py` wires it together and owns the exit codes.

## Checks

Run the suite before proposing changes:

```
python -m unittest discover -s tests -v
```

---
> Source: [shriramkv/agentgateway-diff](https://github.com/shriramkv/agentgateway-diff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-10 -->
