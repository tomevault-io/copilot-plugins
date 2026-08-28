## hunk-codeowners

> This repository contains a local-only Hunk extension that shows CODEOWNERS context for the selected file. Keep v0.1 small. It reads ownership metadata and displays it; it does not change Hunk's changeset or repository files.

# hunk-codeowners agent guide

## Purpose

This repository contains a local-only Hunk extension that shows CODEOWNERS context for the selected file. Keep v0.1 small. It reads ownership metadata and displays it; it does not change Hunk's changeset or repository files.

## Architecture

- `src/index.tsx` registers the pane, toggle command, and `changeset_loaded` handler through `hunkdiff/extension`.
- `src/ownershipState.ts` converts discovery and matching results into an immutable external-store snapshot for React.
- `src/ui/CodeownersPane.tsx` renders the selected file from reactive pane props.
- `src/codeowners/discover.ts` finds the Git root and reads fixed CODEOWNERS locations.
- `src/codeowners/parse.ts` parses rules and diagnostics.
- `src/codeowners/pattern.ts` compiles the supported pattern grammar.
- `src/codeowners/match.ts` resolves last-match-wins ownership.
- `src/codeowners/types.ts` owns the shared core types.
- `tests/fixtures/CODEOWNERS` is the realistic matcher fixture.
- `docs/architecture.md` records the researched Hunk and CODEOWNERS contracts.

Keep CODEOWNERS logic independent of Hunk and React. UI code consumes typed results from the core.

## Commands

```bash
npm install
npm test
npm run typecheck
```

Manual Hunk loading requires Hunk 0.19.0 or newer, which provides extension API generation 6:

```bash
hunk diff --extension /path/to/hunk-codeowners
```

Do not launch Hunk's TUI from an automated or piped check.

## Hunk extension constraints

- Use only public exports from `hunkdiff/extension`.
- Keep the manifest compatibility floor at API generation 6 while Hunk 0.19.0 is the stable release.
  Raise it only when the extension uses a newer API.
- Keep `react`, `@opentui/*`, and `hunkdiff` in `devDependencies`. Hunk provides them at runtime.
- Do not bundle React.
- Register APIs synchronously in the default factory.
- Use pane props for file selection. Do not poll or duplicate selection state.
- Use `ctx.panes.toggle("codeowners")` for the toggle command.
- Preserve Hunk file order and metadata. Do not call `transformChangeset` in v0.1.
- Treat `hunk.config` as untrusted and do not use it for paths or execution.

## CODEOWNERS correctness

- Search `.github/CODEOWNERS`, `CODEOWNERS`, then `docs/CODEOWNERS`.
- Match paths case-sensitively and apply the last matching rule.
- Preserve every owner on the winning line, its pattern, and its one-based source line.
- Ownerless rules explicitly clear ownership.
- GitHub does not support negation, character ranges, or escaping a leading `#`.
- Never guess when syntax falls outside the supported grammar. Return a diagnostic state and add tests.
- Add or update parser and matcher tests for every semantics change.

## Scope boundaries

Do not add GitHub APIs, authentication, `gh`, owner validation, avatars, colors, changeset grouping, filters, owner search, review requests, file editing, networking, telemetry, daemons, shell configuration, or automatic CODEOWNERS changes without a new product decision.

## Style

Use strict TypeScript, explicit discriminated unions, pure functions, immutable snapshots, and small modules. Avoid `any`, state libraries, dependency injection, and runtime dependencies unless a measured correctness gain justifies one. Comments should explain an invariant or a surprising decision.

## Authoritative references

- [Hunk extension loading](https://www.hunk.dev/docs/extend/extensions/)
- [Hunk Extension API](https://www.hunk.dev/docs/extend/extension-api/)
- [Hunk custom panes](https://www.hunk.dev/docs/extend/custom-sidebars/)
- [Hunk extension examples](https://github.com/modem-dev/hunk/tree/main/examples/extensions)
- [GitHub CODEOWNERS](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)
- [Git gitignore patterns](https://git-scm.com/docs/gitignore#_pattern_format)

---
> Source: [gentilfp/hunk-codeowners](https://github.com/gentilfp/hunk-codeowners) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
