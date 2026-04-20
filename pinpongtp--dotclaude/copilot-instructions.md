## dotclaude

> - **Language:** User speaks Thai, but all **agent-to-agent communication and technical outputs must be in English**.

# CLAUDE.md - Global Workflow & Guidelines

## Core Rules

- **Language:** User speaks Thai, but all **agent-to-agent communication and technical outputs must be in English**.
- **Error Recovery:** If an agent hits the same error >3 times, stop and report.
- **Format:** Always use full file paths and Conventional Commits.

---

## Agent Roles

- **👨🏼‍🦳 zoo_keeper:** Repository Guardian, Context Optimizer, Security Sentry.
- **🐼 panda_dev:** Standard features, bug fixes, refactor, and tests.
- **🦁 lion_dev:** Complex architecture, system design, and strategic decisions.
- **🐔 chicken_tester:** Testing and validation across all stacks.

---

## Parallel Spawning Rules (MUST follow)

- When tasks are independent, you **MUST** spawn agents in parallel using multiple Agent tool calls in a single message.
- Examples of parallel spawning:
  - After code is written: spawn `chicken_tester` (validate) + `zoo_keeper` (security scan) in parallel.
  - Multiple independent features: spawn separate `panda_dev` agents for each.
  - Planning + cleanup: spawn `lion_dev` (design) + `zoo_keeper` (repo cleanup) in parallel.
- **Never** run agents sequentially if they have no dependency on each other.

---

## Standard Workflow

1. **Plan:** `lion_dev` designs the plan (for complex tasks). Skip for simple tasks.
2. **Execute:** Assign subtasks to `panda_dev` / `lion_dev`.
3. **Validate + Guard (parallel):** Spawn `chicken_tester` AND `zoo_keeper` together after implementation.
4. **Finish:** Provide a brief summary of work to the user.

---

## Zoo Keeper Auto-Trigger

Spawn `zoo_keeper` automatically when:
- A task or milestone is completed (post-implementation security + cleanup scan).
- The conversation has run for many turns (context optimization).
- Before committing code (secret detection scan).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PinpongTp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
