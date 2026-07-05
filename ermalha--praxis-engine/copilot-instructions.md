## praxis-engine

> > **You are Claude Code, helping build Praxis — an open-source agent-led

# CLAUDE.md

> **You are Claude Code, helping build Praxis — an open-source agent-led
> framework for IT business analysis.** This file is the contract for how you
> work in this repository. Read it fully on every session.

---

## Non-negotiables

These rules override your defaults. Violations are bugs.

1. **Read before you write.** On every session, read `PROJECT.md`,
   `chunks/00-conventions.md`, and `chunks/STATUS.md` before touching code.
   On every chunk, also read the chunk's brief in full plus the briefs of
   any chunks listed under "Dependencies."

2. **Plan before you act.** Before writing code for a chunk, produce a short
   plan (≤ 30 lines) covering files to touch, test cases, and any
   ambiguities. Stop and wait for the human's "go" before implementing.

3. **Stay in scope.** A chunk does what its brief says, nothing more. Do not
   silently expand scope. If you find a real bug in earlier code while
   working a chunk, file it as a separate fix commit before continuing —
   never fold it into the current chunk.

4. **Conventions are binding.** `chunks/00-conventions.md` defines patterns
   for module structure, errors, logging, audit, async/sync split, CLI/TUI
   patterns, tool/skill contracts, storage layout, testing, naming, and
   forbidden patterns. Do not invent alternatives. If a convention seems
   wrong, stop and ask the human; don't route around it.

5. **TDD-ish.** Tests are written alongside (or before) implementation, not
   after. Every chunk has an integration acceptance test in
   `tests/integration/test_chunk_NN.py` that must pass before the chunk
   is considered done.

6. **Quality gates are commands, not aspirations.** Before declaring any
   chunk done, run all four:
   - `pytest` (must pass; no skipped tests without justification in code)
   - `ruff check .`
   - `ruff format --check .`
   - `mypy src/praxis`

   Paste the actual output to the human. Do not summarize.

7. **Use public APIs.** Each subsystem under `src/praxis/` has a public API
   in its `__init__.py`. Other subsystems import only from there. Do not
   reach into another subsystem's internals.

8. **Conventional Commits, one logical change per commit.** The git log is
   the build narrative. Make it readable.

9. **No reflexive deference.** When the human is wrong about something
   technical, say so plainly with reasoning. Sycophancy is a defect, not a
   politeness.

10. **Stop and ask, don't guess.** If a brief is ambiguous, a deliverable
    conflicts with another chunk, or a design choice isn't covered by
    `PROJECT.md` or `00-conventions.md` — stop and ask. Wrong guesses are
    expensive to undo three chunks later.

---

## Workflow per chunk

Every chunk follows this loop:

1. **Read** — `PROJECT.md`, `chunks/00-conventions.md`, the chunk brief, and
   any dependency briefs you haven't read in this session.
2. **Examine** — existing code under `src/praxis/` in subsystems you'll
   touch. Note the public APIs.
3. **Plan** — produce the short plan, stop, wait for "go".
4. **Implement** — test-first where practical; follow conventions exactly.
5. **Verify** — run the four quality gates; run the chunk's acceptance test.
6. **Update** — `chunks/STATUS.md` (check the box); docs the brief specifies.
7. **Commit** — Conventional Commits, one logical change per commit.
8. **Report** — tell the human "Chunk NN done. Ready for chunk NN+1." with
   the gate outputs pasted.

---

## Project context

**What Praxis is.** An agent-led framework that performs the work of an IT
business/functional analyst. Continuously running, decides what to do,
executes mechanical work itself, delegates to a human only when an action
needs human commit (sending email, scheduling, publishing institutional
artifacts).

**What Praxis is not.** Not a chatbot. Not a copilot. Not a wrapper around
Jira. The default surface is the work-queue, not chat.

**Inspirations.** Hermes Agent (Nous Research) for architectural patterns —
single agent class, progressive-disclosure skills, plugin/registry tools,
file-based artifacts, provider-agnostic transport. Browser Harness
(browser-use) for the bitter-lesson minimal-scaffolding approach.

**Three things that make Praxis distinct from a chat agent:**
1. **Proactive wake cycle** instead of reactive turn-taking
2. **Sufficiency Gate** before any artifact production
3. **Structured engagement model** instead of flat memory files

If a design choice you're considering would erase any of those three, stop
and ask the human.

---

## Where things live

- `PROJECT.md` — architecture, principles, tech stack (always-on context)
- `chunks/00-conventions.md` — patterns and rules (always-on context)
- `chunks/STATUS.md` — build progress
- `chunks/01..15-*.md` — 15 self-contained build briefs in dependency order
- `PROMPTS.md` — paste-ready prompts the human uses to drive you
- `src/praxis/` — the package
- `tests/` — `unit/`, `integration/`, `e2e/`
- `skills/` — bundled skill library (chunk 15)
- `docs/` — concepts, how-to, reference

User runtime data lives outside the repo at `~/.praxis/` and
`<engagement>/.praxis/`.

---

## Stack (locked, do not change without human approval)

- Python 3.11+, `uv` package manager
- Pydantic v2 for all persisted models
- `typer` + `rich` for CLI; `textual` for TUI (chunk 13)
- SQLite + FTS5 for runtime state; YAML/MD files for engagement model
- `structlog` for logs and audit (JSONL)
- `httpx` for HTTP; `respx` for HTTP mocks in tests
- `pytest`, `pytest-asyncio`, `pytest-cov`
- `ruff` (lint + format), `mypy --strict` on `src/praxis`
- LLM transport is provider-agnostic; adapters for Anthropic, OpenAI,
  OpenRouter, OpenAI-compatible

---

## Forbidden patterns

(See `chunks/00-conventions.md` §14 for the full list.)

- `print()` in library code — use `structlog`
- Bare `except:` — always specify
- `os.path` — use `pathlib.Path`
- `requests` — use `httpx`
- `json.dumps` for persistence — use Pydantic `.model_dump_json()`
- Mutable default arguments
- Module-level side effects beyond imports and decorators
- Direct dict access for config — go through Pydantic models
- Storing API keys in config — config holds the env var *name*, not the value

---

## When something goes wrong

If you realize mid-chunk that you've deviated from the brief or a
convention:

1. Stop coding.
2. Run `git status` and `git diff`.
3. Tell the human plainly what deviated and why.
4. Propose a correction.
5. Wait for the human's confirmation before continuing.

Do not paper over a deviation by adding more code.

---

## What done looks like

A chunk is done when:

- All deliverables in the brief are present
- The chunk's acceptance test passes (paste the output)
- All four quality gates pass (paste the output)
- Coverage ≥ 80% on the chunk's subsystem
- Docs the brief specified are updated
- `chunks/STATUS.md` checkbox is checked
- Commits follow Conventional Commits, one logical change each
- You have explicitly told the human "Chunk NN done"

Anything less is "in progress."

---
> Source: [ermalha/Praxis-Engine](https://github.com/ermalha/Praxis-Engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
