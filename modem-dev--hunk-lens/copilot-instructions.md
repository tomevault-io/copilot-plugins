## hunk-lens

> Hunk Lens is a small, git-installed Hunk extension that pins the selected split-diff row at the bottom of the terminal. Keep it focused: it is a lens, not a general pane toolkit.

# hunk-lens agent notes

## Purpose

Hunk Lens is a small, git-installed Hunk extension that pins the selected split-diff row at the bottom of the terminal. Keep it focused: it is a lens, not a general pane toolkit.

## Important files

- `index.tsx` contains the complete extension and public registration.
- `index.test.ts` covers terminal-width formatting and registration behavior.
- `package.json` declares the root entry through `hunk.extensions` and the minimum Hunk extension API.

## Working rules

- Use only the public `hunkdiff/extension` contract.
- Keep React, OpenTUI, and `hunkdiff` in `devDependencies`; Hunk supplies them at runtime.
- Do not add a build step. Hunk loads the TypeScript entry directly.
- Keep runtime dependencies empty unless a feature clearly requires one.
- Preserve split-only availability and the fixed three-row geometry.
- Add short JSDoc comments for exported functions and non-obvious helpers.

## Validation

```bash
bun install
bun run format:check
bun run lint
bun run typecheck
bun run test
```

Before a release, also install from the local checkout and exercise a real split diff:

```bash
hunk extension install /path/to/hunk-lens
hunk diff --mode split
```

## Releases and changelog

- Maintain user-visible changes in top-level `CHANGELOG.md` under `## [Unreleased]` using `Added`, `Changed`, and `Fixed` subsections without duplicates.
- For a release, move entries into a dated immutable version section, create a `v<version>` tag, and start a fresh Unreleased section.
- Use the released changelog section as the GitHub release body; verify generated release notes rather than trusting them blindly.
- This extension is distributed from git, not npm. Keep `private: true` in `package.json`.

## Commits

Use Conventional Commits: `<type>[scope]: <description>`.

---
> Source: [modem-dev/hunk-lens](https://github.com/modem-dev/hunk-lens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
