## downtime-test

> Flutter cross-platform detox/focus app.

# AGENTS.md

## Project

Flutter cross-platform detox/focus app.

Users create one or more scheduled focus windows. During a focus window, the app restricts access to apps/device functionality as strongly as Android and iOS allow.

Use Flutter for shared code and native Kotlin/Swift code when platform APIs are required.

---

## Rules

* Make the smallest change that solves the task.
* Do not rewrite working code unnecessarily.
* Do not modify unrelated files or formatting.
* Reuse existing components, services, utilities, and patterns.
* Do not make unnecessary refactors.
* Do not add dependencies without asking f**`AGENTS.md`**irst.
* Ask before making significant architectural decisions.
* Never discard or overwrite the user's existing work.

---

## Platform

* Android and iOS may require different implementations.
* Use supported, documented platform APIs.
* Never fake system-level restrictions with UI.
* Never use private or unsupported APIs.
* Do not use AccessibilityService as a workaround without verifying that it is technically appropriate and allowed by Google Play.
* Read `docs/platform.md` before changing system-level blocking.

---

## Task Process

For each task:

1. Read the task completely.
2. Inspect relevant code first.
3. Give the user a short plan.
4. Implement the task.
5. Run appropriate validation/build checks.
6. Review the final changes.
7. Report what changed and any limitations.

Do not repeatedly ask for confirmation during normal implementation.

---

## Files

Do not manually edit generated or build output.

Avoid modifying:

* `.env*`
* `.dart_tool/`
* `build/`
* generated files
* credentials/signing files

Modify the source/configuration that generates them instead.

---

## Git

* Never commit.
* Never reset, revert, or discard user changes.
* Never modify Git history.

---

## Errors

When something fails, diagnose it and try a few reasonable fixes.

Stop and tell the user when solving it requires:

* a major architectural decision
* a new dependency
* unsupported platform behavior
* a significant scope change

Do not make repeated speculative changes.

---

## UX

Make obvious, low-risk UX improvements related to the task, but do not redesign unrelated parts of the app.

---
> Source: [jay51/downtime-test](https://github.com/jay51/downtime-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
