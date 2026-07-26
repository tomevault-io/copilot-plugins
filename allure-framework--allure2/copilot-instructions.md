## allure2

> Use [Allure Agent Mode](docs/allure-agent-mode.md) for all test-related work in this repository.

# Project Guide

Use [Allure Agent Mode](docs/allure-agent-mode.md) for all test-related work in this repository.

- Read `docs/allure-agent-mode.md` before designing, writing, reviewing, validating, debugging, or enriching tests.
- If a command executes tests and its result will be used for smoke checking, reasoning, review, coverage analysis, debugging, or any user-facing conclusion, run it through `allure agent`. It preserves the original console logs and adds agent-mode artifacts without inheriting the normal report or export plugins from the project config.
- Use `allure agent` for smoke checks too, even when the change is small or mechanical.
- Only skip agent mode when it is impossible or when you are debugging agent mode itself.
- If agent-mode output is missing or incomplete, debug that first rather than silently falling back to console-only review.
- Use Allure agent-mode when adding tests for features or fixes so expectations, evidence quality, and scope review are part of the loop.

---
> Source: [allure-framework/allure2](https://github.com/allure-framework/allure2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
