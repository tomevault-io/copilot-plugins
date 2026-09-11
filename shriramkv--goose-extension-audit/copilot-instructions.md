## goose-extension-audit

> Guidance for AI coding agents working in this repository.

# AGENTS.md

Guidance for AI coding agents working in this repository.

## What this project is

`goose-extension-audit` inspects the `extensions` block of a
[goose](https://github.com/block/goose) recipe and reports the security risk
each extension carries: stdio commands that hand control to a shell, unpinned
remote-code installers, `inline_python` that shells out or evaluates arbitrary
code, broad-permission builtins, insecure remote transports, hardcoded
secrets, and missing timeouts. It grades each recipe A-F and exposes CI gates.

## Ground rules

- Standard library only for the runtime. PyYAML is an optional accelerator;
  the vendored `miniyaml.py` parser must remain the zero-dependency fallback,
  and both paths must stay tested.
- Target Python 3.9+.
- Every new rule needs a test in `tests/test_audit.py` and, where useful, a
  case in the bundled `samples/`.
- Prefer precise detection (word-boundary regexes, env-reference exclusions)
  over broad substring matches, but never downgrade an arbitrary-shell or
  arbitrary-code-execution finding below `critical`.

## Layout

- `goose_extension_audit/discover.py` finds recipe files under given paths.
- `goose_extension_audit/audit.py` holds the per-type rules and severities.
- `goose_extension_audit/grade.py` maps findings to an A-F score and does
  worst-of aggregation across recipes.
- `goose_extension_audit/report.py` renders console/json/markdown/badge.
- `goose_extension_audit/cli.py` wires it together and owns the exit codes.

## Checks

```
python -m unittest discover -s tests -v
```

---
> Source: [shriramkv/goose-extension-audit](https://github.com/shriramkv/goose-extension-audit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-10 -->
