## kai-security

> This document contains critical information about working with this codebase. Follow these guidelines precisely.

# Development Guidelines

This document contains critical information about working with this codebase. Follow these guidelines precisely.

## Core Development Rules

1. Package Management
   - ONLY use uv, NEVER pip
   - Installation: `uv add package`
   - Running tools: `uv run tool`
   - Upgrading: `uv add --dev package --upgrade-package package`
   - FORBIDDEN: `uv pip install`, `@latest` syntax

2. Code Quality
   - Type hints required for all code
   - Public APIs must have docstrings
   - Functions must be focused and small
   - Follow existing patterns exactly
   - Line length: 88 chars maximum

3. Testing Requirements
   - Framework: `uv run pytest`
   - Async testing: use anyio, not asyncio
   - Coverage: test edge cases and errors
   - New features require tests
   - Bug fixes require regression tests

4. Code Style
    - PEP 8 naming (snake_case for functions/variables)
    - Class names in PascalCase
    - Constants in UPPER_SNAKE_CASE
    - Document with docstrings
    - Use f-strings for formatting

- For commits fixing bugs or adding features based on user reports add:
  ```bash
  git commit --trailer "Reported-by:<name>"
  ```
  Where `<name>` is the name of the user.

- For commits related to a Github issue, add
  ```bash
  git commit --trailer "Github-Issue:#<number>"
  ```
- NEVER ever mention a `co-authored-by` or similar aspects. In particular, never
  mention the tool used to create the commit message or PR.

## Development Philosophy

- **Simplicity**: Write simple, straightforward code
- **Readability**: Make code easy to understand
- **Performance**: Consider performance without sacrificing readability
- **Maintainability**: Write code that's easy to update
- **Testability**: Ensure code is testable
- **Reusability**: Create reusable components and functions
- **Less Code = Less Debt**: Minimize code footprint

## Coding Best Practices

- **Early Returns**: Use to avoid nested conditions
- **Descriptive Names**: Use clear variable/function names (prefix handlers with "handle")
- **Constants Over Functions**: Use constants where possible
- **DRY Code**: Don't repeat yourself
- **Functional Style**: Prefer functional, immutable approaches when not verbose
- **Minimal Changes**: Only modify code related to the task at hand
- **Function Ordering**: Define composing functions before their components
- **TODO Comments**: Mark issues in existing code with "TODO:" prefix
- **Simplicity**: Prioritize simplicity and readability over clever solutions
- **Build Iteratively** Start with minimal functionality and verify it works before adding complexity
- **Run Tests**: Test your code frequently with realistic inputs and validate outputs
- **Build Test Environments**: Create testing environments for components that are difficult to validate directly
- **Functional Code**: Use functional and stateless approaches where they improve clarity
- **Clean logic**: Keep core logic clean and push implementation details to the edges
- **File Organsiation**: Balance file organization with simplicity - use an appropriate number of files for the project scale

## System Architecture

RLM (Recursive Language Models) replaces `llm.completion(prompt)` with an
agentic loop: the LLM gets a Python REPL where context is loaded as a
variable, writes code to inspect/process it, and can make sub-calls to
other LLMs. This is a ReAct-style loop (reason → act → observe → repeat).

```
RLM.completion(prompt)
│
├── LMHandler (TCP socket server on 127.0.0.1:<auto-port>)
│   ├── default_client (depth=0, primary model)
│   ├── other_backend_client (depth=1, cheaper model)
│   └── clients dict (named model overrides)
│
├── Environment (REPL)
│   ├── LocalREPL — exec() in sandboxed namespace
│   ├── DockerREPL — container with HTTP proxy
│   └── ModalREPL — cloud sandbox with HTTP broker
│
└── Iteration Loop (core/rlm.py)
    ├── LLM call via lm_handler.completion()
    ├── Code extraction via find_code_blocks()
    ├── Code execution via environment.execute_code()
    ├── Answer detection via find_final_answer()
    └── History formatting via format_iteration()
```

**Agentic sub-agents**: Root orchestrator (depth=0) can spawn typed
sub-agents via `spawn_*()` functions injected into the REPL. Each
sub-agent is a fully agentic RLM at depth=1 with its own system prompt,
tools, and iteration budget. `llm_query()` calls from sub-agents go to
depth=2 (single-shot). See `docs/rlm.md` for the full design.

## Core Components

- `core/rlm.py`: Main RLM class — iteration loop, completion lifecycle,
  persistent mode
- `core/lm_handler.py`: TCP socket server routing LM requests by
  model/depth
- `core/comms_utils.py`: Socket protocol — LMRequest/LMResponse,
  send/recv utilities
- `core/types.py`: Dataclasses — UsageSummary, RLMIteration,
  RLMChatCompletion, REPLResult, CodeBlock
- `environments/local_repl.py`: LocalREPL — sandboxed exec(),
  llm_query, persistent namespace
- `environments/base_env.py`: BaseEnv abstract class, NonIsolatedEnv,
  IsolatedEnv, SupportsPersistence protocol
- `environments/docker_repl.py`: Docker-based isolated REPL
- `environments/modal_repl.py`: Modal cloud sandbox REPL
- `clients/`: LM client implementations (OpenAI, Anthropic, Gemini,
  Azure, Portkey, LiteLLM)
- `agents/`: Agentic layer — AgentConfig, registry, spawn functions
  (see `docs/rlm.md` Part 4–6)
- `utils/`: Parsing (code blocks, final answers, formatting), prompts,
  dependency analysis, state managers
- `schemas.py`: Shared data schemas
- `state_manager.py`: State management
- `exceptions.py`: Custom exception types

## Pull Requests

- Create a detailed message of what changed. Focus on the high level description of
  the problem it tries to solve, and how it is solved. Don't go into the specifics of the
  code unless it adds clarity.

- Always add `ArthurClune` as reviewer.

- NEVER ever mention a `co-authored-by` or similar aspects. In particular, never
  mention the tool used to create the commit message or PR.

## Python Tools

## Code Formatting

1. Ruff
   - Format: `uvx ruff format src`
   - Check: `uvx ruff check src`
   - Fix: `uvx ruff check src --fix`
   - Critical issues:
     - Line length (88 chars)
     - Import sorting (I001)
     - Unused imports
   - Line wrapping:
     - Strings: use parentheses
     - Function calls: multi-line with proper indent
     - Imports: split into multiple lines

2. Type Checking
   - Tool: `uvx ty check src`
   - Requirements:
     - Explicit None checks for Optional
     - Type narrowing for strings
     - Version warnings can be ignored if checks pass

## Error Resolution

1. CI Failures
   - Fix order:
     1. Formatting
     2. Type errors
     3. Linting
   - Type errors:
     - Get full line context
     - Check Optional types
     - Add type narrowing
     - Verify function signatures

2. Common Issues
   - Line length:
     - Break strings with parentheses
     - Multi-line function calls
     - Split imports
   - Types:
     - Add None checks
     - Narrow string types
     - Match existing patterns

3. Best Practices
   - Check git status before commits
   - Run formatters before type checks
   - Keep changes minimal
   - Follow existing patterns
   - Document public APIs
   - Test thoroughly

---
> Source: [firstbatchxyz/kai-security](https://github.com/firstbatchxyz/kai-security) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
