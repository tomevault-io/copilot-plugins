## amap-skill

> - Core implementation lives in `skills/amap`.

# Repository Guidelines

## Project Structure & Module Organization
- Core implementation lives in `skills/amap`.
- CLI entrypoint: `skills/amap/scripts/amap.ts`.
- Business logic modules: `skills/amap/scripts/lib/` (`validators.ts`, `commands.ts`, `endpoints.ts`, `http.ts`, `config.ts`).
- Tests live in `skills/amap/tests` with JSON fixtures in `skills/amap/tests/fixtures`.
- Reference docs and usage maps live in `skills/amap/references`.
- Root project config is in `package.json`, `tsconfig.json`, and `bun.lock`.

## Build, Test, and Development Commands
- `bun install`: install dependencies from `bun.lock`.
- `bun run test`: run the repository test suite (`cli.test.ts` and `commands.test.ts`).
- `bun run amap:help`: show CLI command help and supported flags.
- `bun skills/amap/scripts/amap.ts geocode --address "<address>" [--city "<city>"]`: run the CLI directly for local verification.
- No separate build step is required; TypeScript is configured with `noEmit`.

## Coding Style & Naming Conventions
- Language: strict TypeScript (`tsconfig.json` uses `"strict": true`).
- Use 2-space indentation and explicit, readable function names.
- Keep command names and CLI flags in kebab-case (for example, `bike-route-address`, `origin-city`).
- Keep policy/validation centralized:
  - Flag validation and defaults in `validators.ts`.
  - Endpoint mapping and success criteria in `endpoints.ts`.
  - Exit behavior in `config.ts` (`CliError`, `ExitCode`).
- Prefer fail-fast checks for invalid inputs before making network calls.

## Testing Guidelines
- Framework: `bun:test`.
- Test files use `*.test.ts` naming under `skills/amap/tests`.
- Add fixture-based tests for API payload variants in `skills/amap/tests/fixtures`.
- For new commands, cover:
  - Valid happy path.
  - Invalid/unknown flag handling.
  - AMap business-error responses.
  - Network/error propagation (`ExitCode.NETWORK`).

## Commit & Pull Request Guidelines
- This checkout does not include local Git history; follow Conventional Commits by default (`feat:`, `fix:`, `refactor:`, `docs:`).
- Keep PRs focused and include:
  - Why: problem and user impact.
  - How: key implementation choices.
  - Tests: exact commands executed and results.
- Link related issues and include CLI command examples when behavior changes.

## Security & Configuration Tips
- Configure `AMAP_MAPS_API_KEY` in environment variables; never commit secrets.
- Treat CLI input as untrusted: validate/sanitize before request construction.
- Avoid logging sensitive values in errors or debug output.

---
> Source: [kaichen/amap-skill](https://github.com/kaichen/amap-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
