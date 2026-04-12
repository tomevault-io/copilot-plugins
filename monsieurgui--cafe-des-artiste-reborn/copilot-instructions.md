## cafe-des-artiste-reborn

> _All rule numbers are stable; cite them when you comment or create tasks._

Project Constitution
_All rule numbers are stable; cite them when you comment or create tasks._

This document defines the rules of engagement for building the Java Discord Music Bot. Adherence to these rules is mandatory for all contributions.

---

## 1 — Core Philosophy: One Task, One Commit
The fundamental principle of this project is that the AI will only ever perform **one logical task at a time**. A task is either writing code, fixing a bug, or updating project management files. A task results in a single commit. We will never combine code changes and administrative changes in the same commit. This simplifies the AI's job and prevents errors.

## 2 — The Development Workflow (Strict)
The project advances in a strict two-step cycle:

1.  **The Code Step:** The AI is assigned a `feat`, `fix`, or `refactor` task from `plan.md`. The AI's only output is a patch for the application code and a corresponding commit message.
2.  **The Housekeeping Step:** Immediately following the Code Step, the AI is assigned a `chore(project): update management files` task. The AI's only output is a patch for `plan.md` and/or `memory.md` and a corresponding commit message.

This cycle repeats. This ensures project management is never forgotten because it is an explicit, required step.

---

## 3 — Mission
Build and maintain a **Java-based modular Discord music bot** following the rules and workflow defined herein. Key objectives include glitch-free audio playback, high responsiveness, a clear separation of concerns between services, and simple, one-command deployment via Docker Compose.

## 4 — Scope & Non-scope
*   **In scope:** All Java source code, Docker assets (`Dockerfile`, `docker-compose.yml`), build files (`pom.xml` or `build.gradle`), and the `plan.md` and `memory.md` files.
*   **Out of scope:** CI/CD pipeline configurations, external infrastructure management, end-user documentation.

## 5 — Coding Standards
*   **Style:** All Java code must adhere to a consistent style, enforced automatically. (e.g., Google Java Style Guide).
*   **Formatting:** All code must be formatted using a build tool plugin like **Spotless**.
*   **Linting:** All code must pass static analysis with **Checkstyle** with no errors.
*   **Imports:** No wildcard imports (e.g., `import java.util.*;`). Imports must be organized.
*   **Documentation:** All public classes, methods, and non-obvious fields must have **Javadoc** comments.

## 6 — Dependency Policy
*   **Management:** Dependencies are managed exclusively in the `pom.xml` (Maven) or `build.gradle` (Gradle) file of each module.
*   **Approval:** To add a new dependency, it must first be an approved task in `plan.md`. New dependencies are recorded in `memory.md` during the Housekeeping Step.

## 7 — Project Structure (Immutable)
The project is divided into the following modules. Within each module, the standard Maven/Gradle directory structure (`src/main/java`, `src/test/java`) must be used. Do **not** move, rename, or add modules without an approved task.

/bot-app/ # Main application entrypoint, DI, and core JDA event listeners
/bot-core/ # Handles Discord commands and UI-facing logic (embeds)
/bot-player/ # Handles audio playback (LavaPlayer), queue logic
/bot-common/ # Contains shared data structures (POJOs, interfaces)
/infra-config/ # Contains Dockerfiles, docker-compose.yml, and other config


## 8 — Inter-Service Communication
*   All communication between services **must** be asynchronous via **RabbitMQ**.
*   No direct synchronous calls (e.g., HTTP requests) between Java services.
*   Message contracts (POJOs) are defined in `bot-common/`.

## 9 — Commit Format (Conventional Commits)
All commit messages must follow the Conventional Commits specification:
`<type>(<scope>): <subject>`

*   **Allowed `<type>`:** **feat**, **fix**, **refactor**, **test**, **docs**, **chore**.
*   **`<scope>`:** The name of the module (e.g., `player`, `core`) or `project` for housekeeping.
*   **Bug Fixes:** When the `<type>` is `fix`, you must first analyze the provided `logs.txt` file (if it exists) to diagnose the root cause.
*   **Example (Code Step):** `feat(player): implement song skipping logic with LavaPlayer`
*   **Example (Housekeeping Step):** `chore(project): update plan and memory for skip feature`

## 10 — IO Contract (Strict & Absolute)
When executing any task, your reply must contain **only** the following, in this exact order:
1.  A single, unified git diff in a `patch` fenced code block.
2.  The full, correctly formatted commit message on the line immediately following the diff block.

_No extra prose, greetings, logs, or explanations are permitted in the response. This rule is absolute._

## 11 — `plan.md` Rules
*   The top unchecked task in `plan.md` is always the next action.
*   A `chore(project)` task will **always** follow a `feat`, `fix`, or `refactor` task.
*   When executing a `chore(project)` task, you will generate a patch that moves the previously completed task to the `## Done` section and appends the commit hash.

## 12 — `memory.md` Rules
*   This file is an append-only log of immutable facts, decisions, and dependency additions.
*   Updates to this file only happen during the `chore(project)` Housekeeping Step.
*   Never edit or delete existing lines.

## 13 — Testing & CI
*   All new features or fixes must be accompanied by unit tests using **JUnit**.
*   The entire project must compile and all tests must pass when run with `./mvnw clean install` or `./gradlew build`.
*   If a change causes the build or tests to fail, abort the patch and request clarification.

## 14 — Safety Rails
*   **SR-14.1:** Never modify this `CLAUDE.md` file.
*   **SR-14.2:** Do not modify CI/CD workflow files.
*   **SR-14.3:** Do not expose service ports in `docker-compose.yml` unless a task explicitly requires it.
*   **SR-14.4:** Never commit secrets, tokens, or API keys.

## 15 — Uncertainty & Clarification
If you encounter an impossible constraint, missing information, or ambiguity:
1.  Commit nothing.
2.  Respond with the single word `BLOCKED:` followed by a concise, specific question.

---
_End of constitution_

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/monsieurgui)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/monsieurgui)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
