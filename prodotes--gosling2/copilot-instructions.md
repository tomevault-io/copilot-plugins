## gosling2

> Project protocols for architectural integrity.

# GOSLING2 CONSTITUTION

Project protocols for architectural integrity.

> Assume a technical persona. No fluff, no hand-holding, just the facts.

---

## 1. Spec-First

- Search Open Brain before planning
- Agree on methods, location, and tests before writing code
- Write code only after plan approval
- Non-trivial changes require a spec in `docs/`

---

## 2. Lookup Protocol

- **MANDATORY**: Every turn involving planning or changes MUST start by opening the relevant `docs/lookup/` file via `view_file`. No exceptions.
- Every entry must have: header, location, responsibility, signatures.
- Every method must have a strict Python signature.
- Every entry must have a single, definitive file path.
- **Constitution, Not Mirror**: The lookup docs are the prescriptive contract. Never "patch" the MD at the end of a session to match code you just wrote.
- **Handling Integrity Failures**: If a test says a method is missing from the MD, do NOT blindly add it. Run an AST/grep check. **CRITICAL:** For Router endpoints, do not just search for the Python method name (`get_x`). You MUST verify if the corresponding HTTP route (e.g., `/api/v1/...`) is present in `src/static/js/dashboard/api.js`. If the route is unused by the frontend AND backend, it is Zombie Code: delete it. If it is used without authorization, refactor the caller. New features require updating the MD *first*.


---

## 3. Debug Protocol

- No theorizing without evidence
- Reproduce and analyze exact output before patching
- Services and Repositories must log entry/exit

---

## 4. Open Brain

- Stores permanent lessons and workflow preferences via MCP
- Project state belongs in `docs/`, not the Brain
- Use `[GOSLING2]` or `[GLOBAL]` prefixes for brain operations
- Search brain when starting sessions or hitting familiar bugs

---

## 5. Done Protocol

A task is not done until:

1. `docs/lookup/` is accurate to implementation
2. Logger instrumentation is pervasive
3. Zombie code and orphan files are purged
4. `black .`, `ruff check . --fix`, and `pyright` pass with 0 errors (errors mapping to external libraries like `mutagen` without type stubs are exempt)
5. `pytest` passes on the entire test suite
6. `pytest --cov` shows 100% coverage for new code

---

## 6. No Emojis

- Emojis are strictly forbidden in code, configuration files (JSON, etc.), and UI elements.
- Prevents encoding issues and visual "noise" in terminal/IDE.

---

## 7. Execution Protocol (Strict TDD Per-Method)

### 7.0 Authorization Gate (MANDATORY PRE-CHECK)

Before writing ANY code (tests, implementation, edits), classify the user's message:

| Signal            | Example                                                   | Action Allowed                                                                    |
| ----------------- | --------------------------------------------------------- | --------------------------------------------------------------------------------- |
| **Imperative**    | "fix this bug", "implement X", "add Y"                    | Code authorized. Proceed to 7.1.                                                  |
| **Informational** | "I added a new doc", "check this out", "FYI"              | **READ ONLY.** Read the referenced file, summarize findings, ask what to do next. |
| **Analytical**    | "what do we still need", "report on X", "how does Y work" | **REPORT ONLY.** Analyze and present findings. No code.                           |

If in doubt, **ask**. The default is NO code.

### 7.1 TDD Cadence (The "One Rule Above All")

- **MANDATORY**: Every line of code MUST follow the process in `docs/testing/TDD_STANDARD.md`.
- **The /writing workflow** is the canonical execution protocol, but the definition of "Task" for GOSLING2 is strictly defined by the **per-method TDD loop** in the TDD Standard.
- **"Atomic" means one method**, not one tool call. Use as many tools as needed to implement and test that single method correctly.
- Every response that includes code MUST begin with the **Execution Checklist** progress status.
- If a response contains implementation code for more than one method, it is a **VIOLATION**.
- Cadence per method:
  1. Present the **Feature Chain** (Trigger -> Repo) and get approval.
  2. Write ALL tests for method N directly to the test file. (No chat code blocks).
  3. **HARD STOP.** Wait for explicit user review AND authorization of the tests.
  4. Write the **IMPLEMENTATION** for exactly that one method.
  5. Run formatters and local tests.
  6. Present the diff. Wait for explicit user authorization before the next method.
- **Banker Mode (Rule 113)**: Fail loudly. No defaults (0 for years, row indices for IDs). Error tests must verify DB rollback.
- The goal is **reviewable, authorized changes** -- not minimizing tool calls.

---

## 8. No Chat Promises

- Vows, rules, or promises made in the chat UI are meaningless because they are lost when a new chat starts.
- Never write "From now on I'll do X" in a conversation.
- If a workflow rule is broken, do not apologize. Immediately documented the correction in `AGENTS.md` or Open Brain so it survives the reset.

---

## 9. Deduplication First

- Before adding any new method or service, audit the `docs/lookup/` indices for existing similar logic.
- If an existing path can be generalized (e.g., using a list parameter or optional query builder), you MUST refactor it rather than duplicating.
- Structural duplication in repositories (shared SQL logic) is technical debt and must be flagged for cleanup.

---

## 10. Implementation Walkthrough Protocol (Duck Rule)

- For every atomic method, you MUST provide a brief (2-3 bullet point) "Implementation Walkthrough" before writing ANY code (test or implementation).
- Flag potential sqlite3 / logic traps (integrity errors, N+1 queries, race conditions) during the chat phase.
- This ensures the USER can "feel the wood" and align on the micro-level approach before execution.

---

## 11. Vertical Slice Blueprinting (VSB)

- **The Contract**: Before any code, you MUST define the path from Repo -> Service -> API -> JS in a planning matrix. 
- **The Outcome Matrix**: Map every meaningful [Input x State] permutation (Missing IDs, Nulls, Duplicates) to a hard-outcome. **NO SILENT FALLBACKS.** 
- **Logic Integrity**: If an ID is missing, return a hard 404/LookupError. No "ghost" resolutions.
- **Verification**: Every outcome in the Matrix MUST have a corresponding test case. Final verification must use `docs/testing/TDD_STANDARD.md` criteria.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PROdotes)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/PROdotes)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
