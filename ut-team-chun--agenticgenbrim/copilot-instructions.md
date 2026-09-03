## agenticgenbrim

> This file provides context for Claude Code to understand the project.

# AgenticGenBrim - CLAUDE.md

This file provides context for Claude Code to understand the project.

## Project Overview

**Steel Plate Girder Bridge BrIM Generation Agent**

An agent-based MVP that generates cross-section models of Steel Plate Girder Bridges (RC Deck Slab) using RAG + OpenAI API and outputs them as IFC.

## Repository Structure

```
AgenticGenBrim/
├── src/                    # Source code (see src/CLAUDE.md for details)
├── scripts/                # Utility scripts
├── tests/                  # Tests
├── data/                   # Data (.gitignore)
├── rag_index/              # RAG index (.gitignore)
├── docs/                   # Documentation
├── tasks/                  # Task templates (see tasks/template.md)
├── .claude/                # Claude Code settings (commands/, agents/)
└── Makefile                # Development commands
```

Full directory tree: See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)

## Key Features

1. **RAG**: Converts PDFs such as Japan Road Bridge Specifications (JRA) to text, generates embeddings, and searches for relevant specification chunks during design
2. **Designer**: Receives bridge length L and total width B, and generates a BridgeDesign (structured JSON) based on RAG context
3. **Judge**: Performs deterministic verification calculations (bending, shear, deflection, deck slab thickness, web slenderness ratio, cross beam placement) and generates a PatchPlan via LLM when verification fails
4. **Designer-Judge Loop**: A repair loop that applies PatchPlan on failure and repeats until all verifications pass
5. **IFC Export**: Converts BridgeDesign to Senkei JSON to IFC and passes it to the BrIM environment

## Tech Stack

- **Language**: Python 3.13 / **Package management**: uv
- **LLM**: OpenAI API (Responses API / Structured Output) / **Validation**: Pydantic
- **IFC output**: ifcopenshell / **CLI**: fire / **Formatting/Lint**: Ruff

## Custom commands

### /impl [task description]

A command that performs feature implementation and automatically runs a review.

```bash
/impl RAG検索の精度を改善
```

**Execution steps:**
1. Implements using the `designer-impl` or `ifc-impl` agent depending on the task
2. Validates with `make fmt && make lint`
3. Reviews with the `quality-check` agent
4. Repeats fixes if issues are found

## Custom agents

| Agent           | Purpose                              |
| --------------- | ------------------------------------ |
| `designer-impl` | Designer/RAG related implementation  |
| `ifc-impl`      | IFC conversion related implementation |
| `quality-check` | Code review and quality verification |

See the individual files in `.claude/agents/` for detailed conventions.

## Development workflow

For all CLI commands (Designer, Judge, IFC, batch, etc.), see [docs/USAGE.md](docs/USAGE.md).

For development commands (`make fmt`, `make lint`, `make fix`), see [docs/DEV_GUIDE.md](docs/DEV_GUIDE.md).

## Coding conventions (summary)

- **Naming**: snake_case (vars/funcs), PascalCase (classes), UPPER_SNAKE_CASE (constants)
- **Types**: Annotations required on all functions; use `X | Y` (PEP 604), built-in generics (PEP 585)
- **Pydantic**: Use models instead of `dict`/`tuple` returns; use `StrEnum` for string constants
- **Logging**: Use `logger` from `logger_config`; `print` is prohibited
- **Quality**: No unused/commented-out code; no `try: except: pass`; no magic numbers
- **Files**: Use `pathlib.Path`; use `fire` for CLI; Google-style Docstrings

Full conventions: See [docs/DEV_GUIDE.md](docs/DEV_GUIDE.md)

## Notes

- `.env` for secrets (do not commit); `data/` and `rag_index/` are in `.gitignore`
- Use mocks for LLM-related tests
- Do not automatically run `git add/commit` (only suggest commit messages)

## Documentation

- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) - Directory structure and component overview
- [docs/USAGE.md](docs/USAGE.md) - Setup and usage instructions
- [docs/DEV_GUIDE.md](docs/DEV_GUIDE.md) - Development notes
- [docs/COMPONENT_DESIGNER.md](docs/COMPONENT_DESIGNER.md) - Designer details
- [docs/COMPONENT_JUDGE.md](docs/COMPONENT_JUDGE.md) - Judge details
- [docs/EVALUATION.md](docs/EVALUATION.md) - Evaluation methods
- [docs/json_spec.md](docs/json_spec.md) - Senkei JSON specification

---
> Source: [UT-Team-Chun/AgenticGenBrim](https://github.com/UT-Team-Chun/AgenticGenBrim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
