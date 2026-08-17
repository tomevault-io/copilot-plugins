## agentgateway-lint

> Guidance for AI coding agents working in this repository.

# AGENTS.md

Guidance for AI coding agents working in this repository.

## What this is

A static linter for agentgateway configuration files. Pure Python, stdlib only
for the core. No network calls, no runtime services. The tool reads a config and
emits findings with an A-F grade.

## Layout

- `agentgateway_lint/loader.py` reads JSON natively and block-style YAML with a
  small built-in reader; uses PyYAML automatically when it is installed.
- `agentgateway_lint/rules.py` holds all lint rules. Add new checks here.
- `agentgateway_lint/finding.py` is the finding and result model, including
  scoring and grading.
- `agentgateway_lint/report.py` renders console, JSON, Markdown, and badge output.
- `agentgateway_lint/cli.py` is the argparse entry point and CI gates.
- `samples/` holds a clean and a risky config used by tests and demos.
- `tests/` is unittest based.

## Rules for changes

- Keep the core dependency free. PyYAML is an optional convenience, never required.
- Every new rule needs a stable `rule` id (namespace/slug), a severity, and a test.
- Be tolerant of unknown config keys. Never raise on fields you do not recognise.
- Do not use em dashes in prose or comments.
- Run `python3 -m unittest discover -s tests` before finishing.

## Conventions

- Rule ids are grouped: `structure/*`, `security/*`, `hygiene/*`.
- Severity weights live in `finding.py`. Adjust there, not per rule.

---
> Source: [shriramkv/agentgateway-lint](https://github.com/shriramkv/agentgateway-lint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
