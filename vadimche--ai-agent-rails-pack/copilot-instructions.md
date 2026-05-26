## ai-agent-rails-pack

> Follow these rules when working in this repository.

# Repository Instructions for AI Coding Agents

Follow these rules when working in this repository.

## 1. Do not code before intent is clear

Before making changes, create or update a Change Intent containing:

- the problem being solved;
- expected behavior;
- affected modules/files;
- non-goals;
- invariants that must not be broken;
- tests or checks that must pass;
- rollback notes.

If the task is ambiguous, ask for clarification or propose assumptions before editing files.

## 2. Prefer small, reviewable changes

Do not mix unrelated concerns in one change. Keep refactoring, behavior changes, formatting, dependency upgrades and UI changes separate unless explicitly requested.

## 3. Respect existing architecture

Before adding new abstractions or flows, search for similar existing code. Reuse existing patterns unless there is a clear reason not to.

Do not bypass established boundaries between UI, domain logic, infrastructure, persistence and external APIs.

## 4. Protect invariants

Read `templates/invariant-register.md` if present. If a change can affect a listed invariant, explicitly state how the invariant is preserved and which test/check proves it.

## 5. Testing expectations

When changing behavior, add or update tests where practical. If tests are not added, explain why and provide a manual verification checklist.

## 6. No hidden production risk

For changes touching auth, payments, security, data deletion, external APIs, migrations, trading/order logic, healthcare, legal, or financial flows, stop and request explicit human approval before implementation.

## 7. Post-change summary

After changes, provide:

- files changed;
- behavior changed;
- tests run and results;
- risks and limitations;
- rollback path;
- follow-up work.

## 8. No false certainty

Do not claim that code is safe, secure, compliant, or production-ready unless this was proven by tests, review or external verification. Use precise language.

---
> Source: [vadimche/ai-agent-rails-pack](https://github.com/vadimche/ai-agent-rails-pack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-26 -->
