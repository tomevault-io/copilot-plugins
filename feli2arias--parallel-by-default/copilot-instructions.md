## parallel-by-default

> Use when Claude is executing multiple tasks sequentially that could run simultaneously, or when dispatching subagents one at a time for independent work.


# Parallel by Default

**Core rule:** if tasks don't depend on each other, run them at the same time.

---

## The Check

Before executing multiple tasks, ask:
- Does task B need the output of task A?
- Do they write to the same file?
- Do they share mutable state?

If all three answers are no → run in parallel.

## How to Parallelize

Use multiple tool calls in a single response:
- Multiple Agent tool calls in one message
- Multiple Read/Grep/Bash calls at once
- Multiple file writes when they don't conflict

## What's Always Parallel

- Reading multiple independent files
- Searching different codebases or patterns
- Running tests for independent modules
- Creating multiple independent files
- Analyzing multiple separate concerns

## What Must Stay Sequential

- Task B depends on Task A's output
- Both tasks write to the same file
- One task sets up state the other consumes

---

## Anti-Patterns

| Pattern | Why It's Harmful |
|---------|----------------|
| Dispatching agents one at a time for independent work | 3x slower for no reason |
| Reading files sequentially when all are needed | Wastes turns |
| "Let me first do X, then Y, then Z" when independent | False sequencing |

---

## What Stays the Same

Correctness and ordering constraints are never compromised.

---

## Manual Activation

Invoke with `/parallel-by-default` at the start of any multi-task session.

---
> Source: [Feli2arias/parallel-by-default](https://github.com/Feli2arias/parallel-by-default) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
