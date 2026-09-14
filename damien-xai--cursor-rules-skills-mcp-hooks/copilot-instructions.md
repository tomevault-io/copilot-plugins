## testing

> Vitest conventions for colocated unit tests


# Testing

This is an **Apply to Specific Files** rule. It attaches on Vitest specs.

- Colocate tests as `*.spec.ts` next to the module they cover.
- Use `describe` / `it` / `expect` from `vitest`. Reset module state in `beforeEach`.
- Cover the behavior you changed. Do not delete or skip tests to make them pass.
- Run `npm test` next to the nearest `package.json` (Vitest `--run`). Do not
  start the dev server.
- Do not add Playwright or a second runner unless the user asks. Live UI
  checks use the `browser-testing` skill against the server the user started.

---
> Source: [damien-xai/cursor-rules-skills-mcp-hooks](https://github.com/damien-xai/cursor-rules-skills-mcp-hooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-14 -->
