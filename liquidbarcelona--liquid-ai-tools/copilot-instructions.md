## liquid-ai-tools

> - Break down features into **small, testable iterations**

# Development Philosophy

## Divide and Conquer: Iterative Development

- Break down features into **small, testable iterations**
- Test each step before moving forward

## Backend Development (TDD Approach)
1. Write a **simple test** first
2. Implement the minimal code to pass it
3. Verify it works before continuing

### Test Guidelines
- Keep tests simple and focused
- Check what matters (e.g., status code 400)
- Skip unnecessary assertions (e.g., exact error message text)

## Frontend Development (HTML/CSS)
- **No unit testing required** for UI
- Start minimal: something that works and looks decent
- Iterate and improve the interface gradually
- **Follow existing HTML patterns** in the project
- Prefer native semantics over ARIA patches. HTML first, CSS second, JS last.

## Verification
- Backend changes: run the relevant test suite before considering a task done.
- Frontend changes: verify the page renders correctly.
- Always check for regressions in related functionality.

## Coding Standards
- Laravel/PHP projects: read `~/.claude/laravel-php-guidelines.md` first.
- Deploy scripts and server tasks: follow `~/.claude/sysadmin-guidelines.md`.

## Output
When finishing a task, provide a **short summary** similar to a concise git commit message.

## Writing Style

Precise and natural wording. No generic AI expressions, no grandiose language, no prefabricated phrases. No emojis. In reports, skip formulaic sections like "Executive Summary" or redundant conclusions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LiquidBarcelona) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
