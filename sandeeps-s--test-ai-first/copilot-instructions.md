## test-ai-first

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Spring Boot 4.0.6 application using Java 21 and Gradle (Kotlin DSL). Root package: `com.example.test_ai_first`.

Note: the original package name `com.example.test-ai-first` is invalid — the project uses `com.example.test_ai_first` (underscores).

## Commands

```bash
# Build
./gradlew build

# Run
./gradlew bootRun

# Run all tests
./gradlew test

# Run a single test class
./gradlew test --tests "com.example.test_ai_first.SomeTest"

# Run a single test method
./gradlew test --tests "com.example.test_ai_first.SomeTest.methodName"
```

## Development Approach

Follow **BDD dual-loop TDD**. Every feature increment starts from a failing integration test and is driven inward through unit-level red-green-refactor cycles.

Use the `superpowers:test-driven-development` skill to enforce the inner loop on every implementation task. Use `superpowers:brainstorming` before any new feature work. Use `superpowers:verification-before-completion` before claiming work is done.

### Outer loop (integration)

1. **Red (integration)** — Write one integration/acceptance test that describes the next observable behavior from the outside in. Run it. Confirm it fails for the reason you expect. Do not proceed until the failure message matches your intent.
2. **Inner loop (unit)** — Invoke `superpowers:test-driven-development`. Before writing any production code, write a unit test for it. Run it. Watch it fail. Then write the minimum code to make it pass. Refactor. No production code exists without a failing unit test that drove it — no exceptions, no matter how simple the code seems.
   - When a feature spans multiple layers (e.g. controller → service), run one inner loop per layer, outside-in.
   - Each inner loop is **independent**: a controller test that mocks its service goes green on its own — it does not wait for the real service to exist. Only the integration test waits for all inner loops to be complete, because it wires real objects together.
3. **Green (integration)** — When enough unit-level pieces exist, re-run the integration test. If it still fails, diagnose which piece is missing and drop back into the inner loop. Do not add code without a failing test driving it.
4. **Refactor (integration)** — With the integration test green, refactor across module boundaries if needed. All tests — unit and integration — must stay green.
5. Repeat from step 1 with the next slice of behavior until the task is complete.

### Execution strategy and token cost

Choose the execution approach based on feature size:

- **Small feature (1–3 files, clear spec):** Use `superpowers:executing-plans` (inline). Avoids the cold-start overhead of spawning a fresh subagent per task.
- **Large feature (multiple files, independent tasks):** Use `superpowers:subagent-driven-development`. Isolation matters more at this scale.

When using subagent-driven development, reduce cost with these rules:

- **Model:** Use `haiku` for mechanical implementer tasks (write a test file, implement a single method). Upgrade to a standard model only when a task involves multi-file coordination or debugging.
- **Reviews:** Per-task spec and quality reviews exist because subagents work in isolation — reviews catch drift before it cascades. With inline execution (`executing-plans`), skip per-task reviews entirely and do one review at the end of the feature. With subagent-driven, combine spec and quality into a single review agent per task rather than two separate agents.
- **Plan granularity:** Combine Red and Green into one task per layer (write the test, watch it fail, implement, watch it pass). Do not split them into separate tasks unless the Red step is genuinely uncertain.

---
> Source: [sandeeps-s/test-ai-first](https://github.com/sandeeps-s/test-ai-first) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
