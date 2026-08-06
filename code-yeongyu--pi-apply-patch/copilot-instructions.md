## pi-apply-patch

> Conventions for human contributors and AI agents working on this repository.

# Repository Conventions

Conventions for human contributors and AI agents working on this repository.

## Style

- Terse technical prose. No emojis in commits, issues, PR comments, or code.
- TypeScript strict mode. No `any`, no `unknown` casts where avoidable, no `@ts-ignore`, no `@ts-expect-error`, no enums.
- ESM modules with `.js` suffix in import paths (Node16 resolution).
- Tabs for indentation. Double quotes for strings (matches biome config).
- Tests use vitest with `#given .. #when .. #then` description style or plain `// given / // when / // then` body comments.

## Commands

- `npm install` — install dependencies.
- `npm test` — run vitest once.
- `npm run typecheck` — strict TypeScript check.
- `npm run check` — type check + biome.
- `pi -e ./src/index.ts` — load the extension into a local pi session for manual smoke testing.

## Constraints

- No Bun APIs. Runtime is Node only.
- This extension registers the `apply_patch` tool and only activates it for OpenAI GPT-family models.
- Keep the tool schema, grammar, and descriptions byte-for-byte compatible with Codex unless intentionally updating the golden source.
- No dependency on pi-coding-agent internal modules outside the documented public extension API in `@mariozechner/pi-coding-agent`.

## Don'ts

- No `git add -A` or `git add .`. Stage only the files you changed.
- No `git commit --no-verify`. No force pushes. No history rewriting on shared branches.

---
> Source: [code-yeongyu/pi-apply-patch](https://github.com/code-yeongyu/pi-apply-patch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
