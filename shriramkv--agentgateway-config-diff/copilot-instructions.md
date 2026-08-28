## agentgateway-config-diff

> Guidance for AI coding agents working in this repository.

# AGENTS.md

Guidance for AI coding agents working in this repository.

## What this project is

`agentgateway-config-diff` is a command-line tool that diffs two agentgateway
configs and flags security-relevant drift, with an A-F grade and CI-gated exit
codes. It is deliberately stdlib-only Python (3.9+), MIT licensed, and has zero
required dependencies.

## Hard constraints

- Do not add runtime dependencies. The standard library only. PyYAML may be
  used when present but must always remain optional, with the built-in reader
  as the fallback. Any code path must work with `--prefer-stdlib`.
- Keep Python 3.9 compatibility. No syntax or stdlib APIs newer than 3.9.
- Every change ships with tests. Run `python -m unittest discover -s tests -v`
  and keep it green.

## Layout

- `agentgateway_config_diff/loader.py` reads YAML or JSON into plain Python.
  The built-in YAML subset parser lives here.
- `agentgateway_config_diff/model.py` turns a parsed config into a normalized
  security model (routes, ports, admin surface, inline secrets).
- `agentgateway_config_diff/diff.py` compares two models and emits findings.
- `agentgateway_config_diff/grade.py` scores findings and assigns a grade.
- `agentgateway_config_diff/report.py` renders console, JSON, Markdown, badge.
- `agentgateway_config_diff/cli.py` is the argparse entrypoint and gates.
- `agentgateway_config_diff/samples/` holds the baseline and drifted demo files.
- `tests/` holds the unittest suite.

## Adding a detection rule

1. Extract whatever fact you need in `model.py` (add it to `RouteView` or
   `Model`), keeping the extractor tolerant of key-spelling variants.
2. Emit the finding in `diff.py` with an id, a severity from
   `critical / high / medium / low` for regressions or `info` for
   improvements and neutral facts, a scope string, and old and new values.
3. Add a test in `tests/test_diff.py` that proves it fires and that the
   opposite direction is treated as info, not a regression.
4. If it is security-relevant, add a row to the table in `README.md`.

## Conventions

- Improvements never lower the grade. Only regressions carry penalty weight.
- Redact secret values anywhere they surface in output.
- Prefer durable, meaning-level comparisons over textual ones.

---
> Source: [shriramkv/agentgateway-config-diff-](https://github.com/shriramkv/agentgateway-config-diff-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
