## aidd-jan

> Act as a top-tier software engineer.

# CLAUDE.md

Act as a top-tier software engineer.
When starting a new task without prior context, use the Explore subagent
(Agent tool with `subagent_type: "Explore"`) to explore the codebase before
writing any code. Spin up multiple Explore subagents in parallel if necessary.

## Shell

- Avoid `$()` command substitution in shell commands — it triggers an extra approval step.
- For commit messages and PR bodies, pass them directly as string arguments or use `gh` CLI flags.

## Commits

constraint CommitConvention {
  Format: `type(scope): description`
  Scope is always required — match the feature area (e.g., `agents`, `signals`, `billing`, `deps`).
  Allowed types: feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert.
}

Examples:
- `feat(agents): add result count badge to source test items`
- `fix(signals): strip markdown links from enriched data`
- `chore(deps): update all dependencies to latest versions`
- `refactor(conform): migrate from FormOptionsProvider to configureForms`

## Validation

constraint BeforeCommit {
  Run `bun run validate` and fix all errors before every commit.
  This runs check:types, check:lint, and check:format in parallel via Turbo.
}

- `bun run check:lint` — oxlint (type-aware)
- `bun run check:types` — react-router typegen + tsgo
- `bun run check:format` — oxfmt

## Tests

- `bun run test` — unit tests (bun test across workspaces)
- `bun run e2e` — end-to-end tests (Playwright)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/janhesters) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
