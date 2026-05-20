## phoenix-test-playwright

> - Use tool versions from `.tool-versions`; prefer `mise exec -- <command>` (for example `mise exec -- mix check`).

## Dev

- Use tool versions from `.tool-versions`; prefer `mise exec -- <command>` (for example `mise exec -- mix check`).
- If your shell is already activated by `mise`/`direnv`, plain `mix ...` commands are fine.
- Run targeted tests often while changing files.
- When feature is complete run `mix check`.
- When feature is complete, update `CHANGELOG.md`.
- Always keep docs up to date (exdoc `@doc` + `@moduledoc`, `README.md`)
- If you changed browser transport / websocket behavior, also run `mix test.websocket`.

## phoenix_test upstream compatibility
- `test/phoenix_test/upstream/` is a mirror of upstream `phoenix_test` tests and is used as a compatibility suite.
- Do not add new Playwright-specific behavior tests under `test/phoenix_test/upstream/`.
- Put Playwright-only behavior tests in `test/phoenix_test/playwright_test.exs` (or other non-`upstream/` test files).
- `test/support/router.ex`, `test/support/endpoint.ex`, and `test/support/web_app/` are also copied from upstream and should be changed as little as possible.
- If you must change an upstream-mirrored file, keep the delta minimal and document why.

---
> Source: [ftes/phoenix_test_playwright](https://github.com/ftes/phoenix_test_playwright) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
