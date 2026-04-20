## ohmycode

> OhMyCode is a minimal Claude Code–style clone in roughly 3,000 lines of Python. It supports a REPL, streaming output, tool execution, a permissions pipeline, context compression, a memory system, and skills.

# OhMyCode — AI Agent Development Guide

OhMyCode is a minimal Claude Code–style clone in roughly 3,000 lines of Python. It supports a REPL, streaming output, tool execution, a permissions pipeline, context compression, a memory system, and skills.

## Quick orientation

1. **Architecture and module relationships** → read `agent_docs/ARCHITECTURE.md`
2. **Core interfaces** → read `agent_docs/KEY-MODULES.md`
3. **Gotchas** → read `agent_docs/GOTCHAS.md`
4. **Development conventions** → read `docs/DEVELOPMENT_GUIDE.md`
5. **Design docs** → read project design docs committed in this repository

## References

- Project architecture and module docs under `agent_docs/`
- Development workflow and conventions in `docs/DEVELOPMENT_GUIDE.md`

## Tech stack

Python 3.9+, asyncio, openai SDK, anthropic SDK, rich, prompt_toolkit, tiktoken, httpx, pydantic

## Run and test

```bash
./scripts/setup-cli.sh            # install + register CLI
ohmycode                         # start REPL
ohmycode -p "hello"              # single-shot prompt
python3 -m pytest tests/ -v      # run tests (currently 72)
```

CLI policy: always run via `ohmycode` command.

## Project knowledge (`agent_docs/`)

Persistent, project-level knowledge lives under `agent_docs/` for all tasks:

- `ARCHITECTURE.md` — overall architecture, modules, data flow
- `KEY-MODULES.md` — responsibilities and interfaces of core modules
- `GOTCHAS.md` — pitfalls, counterintuitive design, common mistakes

After changing core project logic, update the relevant docs in `agent_docs/`.

## Skills (`.claude/skills/`)

The project ships seven built-in skills to guide AI agents extending OhMyCode:

- `add-tool` — add a custom tool
- `add-provider` — integrate a new LLM provider
- `add-feature` — add any new feature (points to `docs/DEVELOPMENT_GUIDE.md`)
- `customize-system-prompt` — customize AI behavior
- `customize-response-style` — customize terminal output style
- `commit-conventions` — commit message conventions
- `debug-ohmycode` — troubleshoot issues

## Development conventions

- Keep files < 500 lines and functions < 50 lines
- TDD: write tests → see them fail → implement → see them pass
- Commits: Conventional Commits, e.g. `feat(tools): add xxx`
- New tools/providers are auto-discovered via registries; no edits elsewhere required
- Details: `docs/DEVELOPMENT_GUIDE.md`

## Task management (`dev/`)

Each development task uses a numbered folder under `dev/`:

```
dev/
└── NNN-task-name/
    ├── plan.md       # goals, approach, status
    ├── context.md    # background and progress
    └── tasks.md      # checklist
```

When resuming work, read `plan.md`, `context.md`, and `tasks.md` before coding.

---
> Source: [AlphaLab-USTC/OhMyCode](https://github.com/AlphaLab-USTC/OhMyCode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
