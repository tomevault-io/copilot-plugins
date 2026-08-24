## opalatex

> 0. **MANDATORY**: At the start of a new chat/session in this project, read the project design documentation in [PROJECT_DESIGN.md](./PROJECT_DESIGN.md) before doing project work so you understand the software architecture, design decisions, and client-server connectivity. Within the same chat/session, do not re-read it before every user request unless the design may have changed, the task touches architecture or project design, or you need to refresh context.

# General Rules

0. **MANDATORY**: At the start of a new chat/session in this project, read the project design documentation in [PROJECT_DESIGN.md](./PROJECT_DESIGN.md) before doing project work so you understand the software architecture, design decisions, and client-server connectivity. Within the same chat/session, do not re-read it before every user request unless the design may have changed, the task touches architecture or project design, or you need to refresh context.

0.1. When making substantial architectural or project-design changes, update [PROJECT_DESIGN.md](./PROJECT_DESIGN.md) in the same change so the design documentation stays accurate.

1. Don't make baseless assumptions; before drawing any conclusions, conduct tests and analysis to gather as much information as possible. 

1.1. Do not implement kludges, ad hoc semantic fallbacks, or hidden behavior substitutions without explicit user authorization. In particular, do not "fix" an invalid tool call by silently converting it into a different action. Preserve tool contracts, fail fast with a clear diagnostic, add bounded loop breakers when needed, and ask the user before introducing compatibility hacks.

2. `agenticblocks/` is first-party source in this repository, not a third-party dependency. Change it like any other project code when the framework genuinely lacks a capability: add the feature to the block itself, in a general form that stays useful outside OpalaTex, and cover it with tests. Do not push OpalaTex-specific behavior down into the framework, and do not break existing callers. Genuinely external dependencies (litellm, pydantic, anything under `site-packages`) must still never be patched without explicit user authorization.

3. Don't call litellm directly — always go through agenticblocks. If agenticblocks lacks a feature you need, add it there (see rule 2) instead of reaching around the framework, and tell me what you added.

# Project Guide

## Language

All code, comments, and documentation must be written in **English**.
Hardcoded text in UI components, default values, error messages, and string literals must be written in **English**. User-facing strings that need localization should use the i18n framework (`useTranslation` / `t()`) with English as the default value.

## Architecture

This project uses the **AgenticBlocks.IO** framework, whose source lives in this
repository under [agenticblocks/](./agenticblocks/) — read it there, since the
vendored copy is the one that actually runs. Upstream reference:
https://github.com/gilzamir/agenticblocks

Key things to understand from that source:
- How blocks are structured and composed
- How agents communicate and dispatch events
- Naming conventions used throughout the framework

## Project Context

Read `docs/specs` before making changes. It contains the current project status, known issues, and decisions already made. Do not re-litigate what is documented there.

## Build & Test Commands
Run tests on tests dir after you implement a new feature.


> Fill in your actual commands below — this is the highest-value section.

```bash
python -m pytest
```

---
> Source: [opalacoderdev/OpalaTex](https://github.com/opalacoderdev/OpalaTex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
