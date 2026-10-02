## opencode-jev-router

> This project provides a standalone local Responses/Messages API proxy and an

# Working on this repo

This project provides a standalone local Responses/Messages API proxy and an
in-process OpenCode plugin. Jev selects reasoning effort for supported OpenAI
and Anthropic models. `README.md` covers setup for both paths,
`docs/behavior.md` their wire behavior, and `examples/opencode.jsonc` the
plugin configuration.

## Commands

Use Node.js 24.x from the repo root.

| Task | Command |
| --- | --- |
| Install dependencies | `npm ci` |
| Typecheck and run offline tests | `npm run check` |
| Run a focused test | `npx vitest run test/rewrite.test.ts` |
| Build the CLI in `dist/` | `npm run build` |
| Smoke-test the packed plugin in OpenCode V2 2.0.18 | `npm run smoke:plugin:v2` |
| Watch source changes | `npm run dev` |

The tests use a fake upstream and mocked Jev; they need no API keys. To run the
real proxy, copy `.env.example` to `.env`, set `JEV_API_KEY` and a
Responses API-compatible upstream (`JEV_ROUTER_UPSTREAM_*`), then run
`npm run build && npm start`. The plugin instead wraps existing OpenCode models
configured under `wrap` and reuses their routes and credentials. `npm run eval:live`
calls external services.

## Behavior to preserve

- `src/models.ts`, `src/wire*.ts`, and `src/rewrite.ts`: Pin the outbound model to the resolved
  request profile. Keep request-level `reasoning.effort` at the profile default
  or validated `BASE_EFFORT` override, preserve historical reasoning updates in
  their original positions, and insert Jev's selected update before the next user
  message, or after tool results for a continuation without a new user message
  (never adjacent to another update). This follows OpenAI's reasoning
  continuation guidance and preserves an eligible prefix, not a guaranteed cache
  hit; the reported response effort is not the selected effort. Usage observation
  must not alter streaming bytes or backpressure. On Anthropic, keep top-level
  `output_config.effort` fixed and insert an effort-only system message before
  the newest user message, including tool-result-only user turns; pin thinking
  to adaptive and preserve historical updates.
- `src/wire*.ts` and `src/server.ts`: Reject unsupported modes, truncation,
  and input shapes with a local 400 before calling Jev or the upstream.
- `src/jev.ts` and `src/server.ts`: Jev timeout or failure may fall back to a
  validated effort. Client disconnect must instead abort work without starting
  upstream generation.
- `src/forward.ts` and `src/headers.ts`: Preserve upstream status/body passthrough,
  incremental SSE with backpressure, and header filtering. `src/evidence.ts` logs
  metadata only: no prompts, tool output, credentials, or raw upstream errors.
- `src/plugin.ts` and `src/router.ts`: The V2-only plugin shares classification and
  rewriting with the proxy. `src/plugin.ts` default-exports `id`/`setup()`;
  `src/plugin-v2.ts` adapts `src/plugin-runtime.ts` using type-only imports from
  `@opencode/plugin`. Optional plugin `decisionsLogPath` and CLI
  `JEV_DECISIONS_LOG_PATH` both write metadata-only `JevDecision` events using
  shared formatting; the CLI also prints request evidence to stdout. Log failure
  must never affect generation. Do not infer a selected effort from OpenCode's
  provider or reported response effort alone.

## Worktree setup

Orca waits for `scripts/setup.sh` (`orca.yaml`). The script installs dependencies
and carries local environment files and optional links into a new worktree
without overwriting existing files. If you change setup behavior, run
`node --test scripts/verify-setup.mjs`.

---
> Source: [robertn702/opencode-jev-router](https://github.com/robertn702/opencode-jev-router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
