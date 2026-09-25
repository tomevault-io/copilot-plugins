## agent-development-specification

> This file guides Claude Code (and other agents) when working in this repository.

# CLAUDE.md

This file guides Claude Code (and other agents) when working in this repository.
<!-- ADS:FILL Keep this file current. It is the project-level entry point: what this project is, how to build it, and the rules that gate every change. -->

## Project

<!-- ADS:FILL One paragraph: what {{PROJECT_NAME}} is, the platform/type, and where the code lives. -->
`{{PROJECT_NAME}}` is a {{PROJECT_TYPE}} built with {{TECH_STACK}}. Source lives in `{{SOURCE_DIR}}`; tests in `{{TEST_DIR}}`.

- Entry point: `{{ENTRY_POINT}}`
- {{KEY_FACT_1}}
- {{KEY_FACT_2}}
- Known environment constraints: {{ENV_CONSTRAINTS}}

## Common commands

```sh
# Build
{{BUILD_COMMAND}}

# Run
{{RUN_COMMAND}}

# Test
{{TEST_COMMAND}}
```

<!-- ADS:FILL Include any non-obvious setup, single-test invocation, or environment gotchas. -->

## Architecture

The implementation follows the module boundaries in `{{DOCS_DIR}}/{{ARCH_DOC}}`:

<!-- ADS:FILL One bullet per top-level module/layer — a single sentence on its responsibility. Mirror §2 of the architecture doc. Do NOT add modules here that aren't registered there. -->
- {{MODULE_OVERVIEW}}

## Project-level rules

- **Read these two files before any code-changing task** — they encode the lessons that gate every decision in this repo:
  - `{{AGENTS_DIR}}/VIBECODING_GUIDE.md` — the practice guide (why & how)
  - `{{AGENTS_DIR}}/SELF_CONSTRAINTS.md` — the hard constraints (what's forbidden, what's required, when to stop)

  For any non-trivial task, run through `SELF_CONSTRAINTS.md §A` (pre-flight self-check) before editing anything.

- **Agent settings live under `{{AGENTS_DIR}}/`.** Any `settings.json` or other configuration you generate for your own use MUST be written to `{{AGENTS_DIR}}/` (or a subdirectory) — never the project root, `.claude/`, or `~/.claude/`. This keeps repo-tracked agent state contained.

- **All project documentation lives under `{{DOCS_DIR}}/`** as a single flat directory. Canonical files:
  - `{{PRD_DOC}}` — product requirements (source of truth for behavior)
  - `{{ARCH_DOC}}` — module boundaries, dependencies, contracts, directory layout
  - `{{ROADMAP_DOC}}` — implementation milestones and acceptance checks
  - `{{CONVENTIONS_DOC}}` — naming, style, testing, repository conventions
  <!-- ADS:IF uiux -->
  - `{{UIUX_DOC}}` — visual tokens + interaction details (source of truth for visuals)
  <!-- ADS:ENDIF -->
  <!-- ADS:FILL Add derived docs (implementation spec, asset checklist) here once they exist. -->

- **Source-of-truth hierarchy** — on conflict, upstream wins:

  ```
  <!-- ADS:IF uiux -->
  {{PRD_DOC}} > {{ARCH_DOC}} > {{UIUX_DOC}} > {{CONVENTIONS_DOC}} > derived docs
  <!-- ADS:ENDIF -->
  <!-- ADS:IFNOT uiux -->
  {{PRD_DOC}} > {{ARCH_DOC}} > {{CONVENTIONS_DOC}} > derived docs
  <!-- ADS:ENDIF -->
  ```

  When an upstream doc changes, regenerate the affected sections of derived docs to stay in sync. Feature changes must update `{{PRD_DOC}}`; module/dependency changes must update `{{ARCH_DOC}}` in the same commit. Do not let code and architecture drift for more than 24 hours.

- **Roadmap execution rules:**
  1. Advance in the order set by `{{DOCS_DIR}}/{{ROADMAP_DOC}}` and the logs in `{{LOG_DIR}}/`. Tick each checkbox in the same commit that completes it.
  2. Prefer subagents for concrete implementation; the main conversation decomposes, reviews, and ticks.
  3. **Milestone-level / multi-file / multi-module = a large task**: write a plan/spec first, then execute against it. Small tasks (single file, single checkbox, pure revision) can be done directly.
  4. After each milestone, write an execution log per the `{{LOG_DIR}}/` template.

---
> Source: [SmartSunruiyang/Agent-development-specification](https://github.com/SmartSunruiyang/Agent-development-specification) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
