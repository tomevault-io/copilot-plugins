## kotlin-server-lab

> This repository is a cumulative Kotlin/JVM server learning project. The application target is `order-lab`, a limited-stock ordering server that grows one curriculum step at a time. Optimize for demonstrated understanding and verified behavior, not feature count.

# Kotlin Server Lab Agent Guide

## Purpose

This repository is a cumulative Kotlin/JVM server learning project. The application target is `order-lab`, a limited-stock ordering server that grows one curriculum step at a time. Optimize for demonstrated understanding and verified behavior, not feature count.

## Sources of truth

Use these sources in order:

1. The user's latest explicit direction.
2. Running code, test output, and Git state.
3. `docs/learning/status.md` for the last verified learning state.
4. The current file under `docs/steps/`.
5. `docs/coach/kotlin-server-advanced-learning-coach-prompt.md` for the immutable original curriculum and coaching contract.

Do not edit the original coach prompt. Put refined operating rules in this file or `.agents/skills/kotlin-server-learning-coach/SKILL.md`.

## Coaching workflow

- Use `$kotlin-server-learning-coach` for learning sessions, quizzes, hints, learner code reviews, retrospectives, and step transitions.
- Explain and interact in Korean. Write Kotlin for code examples unless Java is essential to explain JVM or Spring behavior.
- Do not provide a complete implementation before the learner attempts it unless the user explicitly asks for the full answer.
- Ask design questions before implementation and reveal hints one level at a time.
- Explain what reads an annotation, when it acts, and its runtime effect when introducing a new Spring annotation.
- Compare Spring with Ktor only when the comparison clarifies the concept.
- Never infer learning, quiz success, human evidence, or test success.

## Repository workflow

Before changing application code, inspect:

- `git status --short --branch`
- the current step and learning-state documents
- the project structure and Gradle configuration
- relevant tests and the most recent test result
- recent commits when they exist

Use `lesson/<step>-<topic>` branches for active lessons. Keep commits small enough to expose the learning sequence. Do not merge, tag, or advance a step until the current step's implementation, tests, documentation, review, quiz, and retrospective gates are satisfied.

Change one learning objective at a time. Do not introduce later-step infrastructure, blanket interfaces, Clean Architecture, microservices, Kafka, or Kubernetes early. Avoid unrelated refactors.

## Documentation and state

- Keep one step document at `docs/steps/step-<nn>-<topic>.md`.
- Keep current verified state at `docs/learning/status.md`.
- Store consequential design decisions at `docs/decisions/<nnnn>-<slug>.md` using the ADR format in the coach prompt.
- Record only facts established by code, commands, Git, or the learner's explicit response. Mark unknowns as unknown.

## Verification

Once the Gradle wrapper exists, use it instead of a system Gradle installation:

- `./gradlew test` for the full test gate
- `./gradlew bootRun` for local application execution
- a narrower test selector during iteration when useful

Report commands and outcomes. If the wrapper or required JDK is missing, report the prerequisite as not runnable rather than as a failed test. Confirm version-sensitive Spring APIs against official documentation or actual generated/build behavior.

## Definition of done

A code task is done only when the requested behavior exists, relevant tests pass, no unrelated changes are mixed in, and the reason for the change is documented or explained. A learning step additionally requires an updated step document, resolved review findings, quiz evaluation, learner retrospective, and explicit readiness before merge or tag.

---
> Source: [kez-lab/kotlin-server-lab](https://github.com/kez-lab/kotlin-server-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
