## 03-project-structure

> The repository is organized in a monorepo using pnpm workspaces: [pnpm-workspace.yaml](mdc:pnpm-workspace.yaml). It contains the following:

# Project Structure

The repository is organized in a monorepo using pnpm workspaces: [pnpm-workspace.yaml](mdc:pnpm-workspace.yaml). It contains the following:

- **apps/api**: [package.json](mdc:apps/api/package.json)
- **apps/web**: [package.json](mdc:apps/web/package.json)
- **pacakges/types**: [package.json](mdc:packages/types/package.json)

With the root package.json being: [package.json](mdc:package.json)

- When executing command in specific workspaces, use the `--filter @dafthunk/web` for example in order to execute a pnpm command in the web workspace.

---
> Source: [tukeceshi/z3cz](https://github.com/tukeceshi/z3cz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
