## velaire

> Guidance for agents and contributors working in this repository.

# AGENTS.md

Guidance for agents and contributors working in this repository.

## Project shape

Velaire is a Bun + TypeScript ESM project for a general-purpose agent runtime. Coding is a default preset, not a special case in core runtime code.

Primary layers:

- `src/foundation/`: stable protocol types for messages, model events, tools, results, and errors.
- `src/runtime/`: generic agent loop, transcript/session handling, middleware, timeline, and tool execution orchestration.
- `src/policy/`: capability-based policy decisions, approval persistence, risk metadata, and redaction.
- `src/tools/`: reusable tool implementations grouped by workspace, shell, todo, and user interaction.
- `src/providers/`: Anthropic, OpenAI-compatible, mock, and registry code.
- `src/presets/`: preset composition for prompts, tools, skills, middleware, policy, and UI hints.
- `src/skills/`: skill discovery, manifest parsing, registry, and middleware.
- `src/cli/`: Commander entrypoint, config commands, first-run wizard, and future TUI shell.
- `docs/`: architecture and contributor documentation.

## Development commands

```bash
bun install
bun run check:types
bun run lint
bun test
bun run check
bun run build:bin
```

Use `bun run check` as the main gate before handing off changes. It runs TypeScript, ESLint, and Bun tests.

## Hooks and CI

Install local hooks once per clone:

```bash
bun run hooks:install
```

The pre-commit hook runs `bun run check`. GitHub Actions also installs dependencies with Bun and runs typecheck, lint, and tests.

## Coding guidelines

- Keep `foundation` and `runtime` domain-neutral. Do not put coding-only assumptions there.
- Route tool execution through schema parsing, policy evaluation, optional approval, normalized result handling, redaction/summarization, transcript append, and timeline emission.
- Keep provider-specific raw payloads inside provider adapters; runtime receives normalized model events only.
- Do not expose private chain-of-thought. Timeline and UI may show safe summaries: purpose, risk, blast radius, reversibility, and next status.
- Prefer small, focused files that match existing layer boundaries.
- Keep comments concise and intent-focused. In code files, use concise Chinese comments only when a comment is necessary.
- Avoid unrelated refactors.
- Update docs whenever public CLI commands, config schema, preset behavior, permissions, tools, skills, or release checks change.

## Testing guidance

- Use Bun tests.
- Prefer unit tests for pure reducers, policy rules, schema parsing, provider conversion, and tool utilities.
- Use fixture tests for provider streaming behavior.
- Use integration tests for agent loop behavior, approval, abort, max-step guard, and tool result continuation.
- TUI behavior should be tested through reducer/view-model state where possible, not brittle terminal snapshots.

## Documentation map

Update these when behavior changes:

- `README.md`: English quickstart and release checklist.
- `README.zh.md`: Chinese quickstart.
- `docs/architecture.md`: layer responsibilities and dependency rules.
- `docs/foundation.md`: core contracts.
- `docs/runtime.md`: agent loop, middleware, transcript, timeline.
- `docs/tools.md`: built-in tools and execution pipeline.
- `docs/providers.md`: provider adapter contracts.
- `docs/policy.md`: capability, risk, approval, settings rules.
- `docs/ui.md`: TUI/event-store design.
- `docs/skills.md`: discovery, manifest, slash commands.
- `docs/code-convention.md`: style and quality gate.

## Release readiness

Before release, verify:

1. `bun install --frozen-lockfile` succeeds in CI.
2. `bun run check` passes.
3. `bun run build:bin` creates `dist/bin/velaire`.
4. CLI smoke checks pass: `velaire --help`, `velaire --version`, config model list, and mock non-interactive run.
5. README and docs match current implementation.

---
> Source: [leijinynag/velaire](https://github.com/leijinynag/velaire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
