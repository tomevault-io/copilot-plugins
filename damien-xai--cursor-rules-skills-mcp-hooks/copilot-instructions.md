## api-security

> API and server security for SvelteKit handlers and stores


# API security

This is an **Apply to Specific Files** rule. It attaches on server routes and stores.

- Validate every JSON field (type + trim). Use `error(400, ...)` for bad input.
- Treat ids as untrusted. Use `error(404, 'not found')` when a row is missing.
- Never log, return, or hard-code secrets. Env vars stay on the server.
- Export store functions with explicit return types; return copies, not the live array.
- Use `json` and `error` from `@sveltejs/kit` in `+server.ts` handlers.

---
> Source: [damien-xai/cursor-rules-skills-mcp-hooks](https://github.com/damien-xai/cursor-rules-skills-mcp-hooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-14 -->
