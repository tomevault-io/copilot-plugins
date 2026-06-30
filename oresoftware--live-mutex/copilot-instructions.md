## live-mutex

> - **ALWAYS use `.ts` extension for test scripts** - Never use `.js` for new test files

# Cursor Rules for Live-Mutex Test Directory

## Test File Extensions

- **ALWAYS use `.ts` extension for test scripts** - Never use `.js` for new test files
- Convert existing `.js` test files to `.ts` when modifying them
- Suman tests are already in `.ts` format - maintain this standard
- When creating new test utilities or helpers, use `.ts` extension

## Test Structure

- Tests should run serially with independent brokers on unique ports
- Each test should be isolated and not depend on other tests
- Test timeouts should be handled gracefully
- Use proper TypeScript types for all test code

---
> Source: [ORESoftware/live-mutex](https://github.com/ORESoftware/live-mutex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
