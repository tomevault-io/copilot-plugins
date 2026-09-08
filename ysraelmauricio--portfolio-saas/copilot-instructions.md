## portfolio-saas

> **Complements every other document in this set.** This defines *who does what and how they hand off work* — the roles this project's OpenCode setup will implement. The OpenCode-specific configuration syntax, MCP servers, and skills wiring are deliberately **not** in this document — those are finalized in a dedicated session before development starts (`PROJECT_SPEC.md` §27), since OpenCode's own conventions need to be checked fresh at that time rather than assumed now. What follows is the team structure that configuration will be built from.

# AGENTS.md — Multi-Agent Orchestration

**Complements every other document in this set.** This defines *who does what and how they hand off work* — the roles this project's OpenCode setup will implement. The OpenCode-specific configuration syntax, MCP servers, and skills wiring are deliberately **not** in this document — those are finalized in a dedicated session before development starts (`PROJECT_SPEC.md` §27), since OpenCode's own conventions need to be checked fresh at that time rather than assumed now. What follows is the team structure that configuration will be built from.

---

## 1. Reading order for any agent starting a session

1. `CONSTRAINTS.md` — what can never be violated
2. `TASKS.md` — what's already done, what's next
3. `DECISIONS.md` — why things are the way they are, so a past decision doesn't get silently re-litigated
4. Whichever of `DATA_MODEL.md` / `API_SPEC.md` / `ARCHITECTURE.md` / `STYLEGUIDE.md` / `SECURITY.md` is relevant to the specific task

None of this depends on prior conversation history existing in context — that's the entire point of maintaining these documents as the persistent memory (`PROJECT_SPEC.md` §25).

## 2. Roles

| Agent | Responsible for | Reads primarily |
|---|---|---|
| **Orchestrator** | Breaks a phase from `PLAN.md` into tasks in `TASKS.md`, assigns each to the right specialist, sequences dependencies, merges completed PRs once the reviewer approves | `PLAN.md`, `TASKS.md` |
| **Backend** | Everything in `backend/` — migrations, models, services, controllers, jobs | `DATA_MODEL.md`, `API_SPEC.md`, `ARCHITECTURE.md` |
| **Frontend** | Everything in `frontend/` — components, pages, i18n wiring | `API_SPEC.md`, `STYLEGUIDE.md`, `ARCHITECTURE.md` |
| **Database** | Migration correctness, indexes, query performance on anything flagged as high-frequency (`DATA_MODEL.md` §9) | `DATA_MODEL.md` |
| **Reviewer** | Approves or rejects every PR against `TESTS.md`'s Definition of Done before it merges — the only agent with merge authority besides the Orchestrator | `TESTS.md`, `CONSTRAINTS.md` |
| **Security** | Audits payment/auth/webhook code specifically against `SECURITY.md` before those PRs merge — a stricter secondary check beyond the general Reviewer, only invoked for security-sensitive tasks (Phase 3 especially) | `SECURITY.md`, `CONSTRAINTS.md` |
| **Docs/Tests** | Writes the tests specified in `TESTS.md`'s priority list, keeps `DATA_MODEL.md`/`API_SPEC.md` in sync if an approved PR legitimately needed to deviate from them | `TESTS.md` |
| **DevOps** | `infra/`, CI workflows, environment configuration (`ARCHITECTURE.md` §8) | `ARCHITECTURE.md` |

## 3. Handoff protocol

1. Orchestrator creates a task in `TASKS.md` (or picks the next unchecked one), assigns it to a specialist agent, on its own branch (`feat/...`, per `PROJECT_SPEC.md` §25).
2. The specialist implements, writes/updates relevant tests, commits per the Conventional Commits convention, opens a PR.
3. Reviewer checks against `TESTS.md`'s Definition of Done. For anything touching payments, auth, or webhooks specifically, Security also signs off before Reviewer approves.
4. On approval, Orchestrator merges and checks the box in `TASKS.md` — never checked before the merge actually happens.
5. If the task surfaced a genuinely new architectural choice not already covered in `DECISIONS.md`, that entry is added **before** the PR opens, not after — a decision made silently inside a PR is a decision `DECISIONS.md` doesn't know about.

## 4. What no agent does unilaterally

- Merge without Reviewer approval.
- Deviate from `DATA_MODEL.md`/`API_SPEC.md` without updating those documents in the same PR — code and documentation drift apart otherwise, exactly what this whole document set exists to prevent.
- Touch `.env` or any real credential — those are loaded by the human once, in the final pre-development session, never generated or guessed by an agent.
- Treat a `CONSTRAINTS.md` item as negotiable because a task seems to need it — flag it in `DECISIONS.md` and wait, per `CONSTRAINTS.md`'s own instruction.

## 5. Still pending (final pre-development session, not this document)

- Which OpenCode agent-configuration mechanism applies (checked fresh against current OpenCode docs at that time).
- Which MCP servers and skills each role above actually uses.
- Chatbot backup provider cascade (same free-tier logic as the author's existing OpenCode setup).
- Real credentials, loaded directly by the author, never pasted into a conversation.

---
> Source: [YsraelMauricio/Portfolio_SaaS](https://github.com/YsraelMauricio/Portfolio_SaaS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
