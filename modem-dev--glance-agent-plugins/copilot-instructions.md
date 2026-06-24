## glance-agent-plugins

> Guidance for coding agents working in this repository.

# AGENTS.md

Guidance for coding agents working in this repository.

## Purpose

This repo contains small, self-contained [glance.sh](https://glance.sh) integrations for different coding agents. Each plugin should be easy to read, easy to install, and easy to test locally.

## Repository shape

- Each agent integration lives in its own top-level directory such as `pi/` or `opencode/`.
- Each plugin directory should contain:
  - `glance.ts`: the plugin implementation
  - `glance.test.ts`: the Vitest suite for that plugin
  - `README.md`: install and usage instructions for that plugin
- Shared test-only helpers and runtime stubs belong under `test/`.

## Plugin expectations

- Keep plugins self-contained inside their own directory.
- Prefer platform-native APIs such as `fetch`, `AbortController`, and `ReadableStream` over adding new dependencies.
- Preserve the style already used in the plugin you are editing instead of forcing a repo-wide formatting style.
- Match the repo's writing style: concise, direct, and low on filler in code comments, README copy, and PR text.
- Keep runtime-specific code minimal and explicit. Document runtime assumptions in the plugin `README.md`.
- Update the plugin `README.md` whenever install steps, behavior, supported commands/tools, or runtime requirements change.
- If a plugin has internal async/session logic that is hard to verify from the public API alone, expose a small `__testing` surface rather than making production code more coupled.

## Behavior requirements

Every plugin should cover the same core lifecycle:

- create a glance session
- surface the session URL to the agent or user
- listen for image events over SSE
- handle reconnects, timeouts, expiry, and cancellation
- return or inject the received image URL in the host agent's expected format

If you fix a behavior bug in one plugin, check the other plugins for the same pattern before stopping.

## Testing requirements

- Every plugin must have a Vitest suite in the same directory as the plugin file.
- Tests should cover both the happy path and failure modes, not just basic registration.
- At minimum, cover:
  - session creation
  - reuse or refresh of persistent sessions
  - SSE image delivery
  - timeout or expiry handling
  - cancellation or shutdown behavior
  - user-facing command/tool responses
- Prefer deterministic tests with mocked `fetch`, streams, timers, and runtime APIs.
- Keep tests local to the plugin. Put only reusable stubs or fixtures in `test/`.

## Tooling and config

When adding a new plugin directory or new test-only runtime stub, update the root tooling so it stays in sync:

- `vitest.config.ts`
- `tsconfig.json`
- `package.json` if new scripts are genuinely needed
- `.github/workflows/test.yml` if CI inputs change

Do not add a build step unless the repository actually needs one.

## Validation

Before opening or updating a PR, run:

```bash
npm test
npm run test:coverage
npm exec -- tsc --noEmit
```

If a command cannot be run, say so explicitly and explain why.

## PR checklist

- The plugin implementation, tests, and README are all updated together.
- New behavior is covered by tests.
- Root test/typecheck config includes any new plugin or stub paths.
- CI still runs the repo validation commands on pull requests.
- The PR description calls out user-visible behavior changes and any runtime-specific caveats in concise language.

---
> Source: [modem-dev/glance-agent-plugins](https://github.com/modem-dev/glance-agent-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
