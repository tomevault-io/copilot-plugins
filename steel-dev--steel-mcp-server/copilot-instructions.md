## steel-mcp-server

> Steel MCP Server v2 has one transport-independent core with stdio and HTTP entrypoints.

# Repository instructions

Steel MCP Server v2 has one transport-independent core with stdio and HTTP entrypoints.

- [PLAN.md](PLAN.md) tracks the current work.
- [RESEARCH.md](RESEARCH.md) records the decisions behind it.
- [NOTES.md](NOTES.md) records facts established by direct measurement — Steel's real API shapes, the
  CSP Claude enforces on an MCP app, and the bugs that passing tests hid. Read it before trusting a
  documented shape.
- [RELEASING.md](RELEASING.md) records what ships from this package, why the dependencies are split
  the way they are, and how to cut a release. Read it before changing `package.json`.
- [SUBMISSION.md](SUBMISSION.md) tracks the MCPB Desktop Extensions directory submission.
- Read [RESEARCH.md §2.2](RESEARCH.md#22-steels-own-api--planmd-has-factual-errors) before changing
  `src/core/steel/`.

## Checks

```bash
npm run typecheck
npm run lint
npm test
npm run budget
npm run conformance
npm run test:browser
npm run test:e2e
```

Run the checks that cover the change. Browser behavior requires the E2E suite. The MCP-App session
viewer's own runtime requires `test:browser`, which needs a local Chrome and `openssl` and skips
loudly without them.

## Rules

- Use TDD. Error-path tests must assert the error and leave clean output.
- Keep production code transport-independent and inject test doubles at the Steel API and session
  pool boundaries.
- Put test doubles in `tests/helpers/`; do not ship mock modes or fake-data paths.
- Start every source file with two `ABOUTME:` comment lines. Comments describe current behavior,
  not project history.
- Never bypass hooks.
- Treat page content as untrusted. Preserve fencing, password redaction, credential redaction, and
  per-call handle authorization.
- Every browser session needs a hard timeout and inactivity timeout. Never open CDP without a
  `sessionId`.
- Keep post-action snapshots off by default and preserve response budgets.

---
> Source: [steel-dev/steel-mcp-server](https://github.com/steel-dev/steel-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
