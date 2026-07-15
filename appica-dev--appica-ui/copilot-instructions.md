## appica-ui

> A pnpm-workspace monorepo housing the public `@appica/ui-*` packages. One

# Appica UI monorepo — agent guide

A pnpm-workspace monorepo housing the public `@appica/ui-*` packages. One
workspace so far (`pnpm-workspace.yaml`), with more integrations (html, vue, …)
to land as sibling packages:

- **`packages/react`** — the `@appica/ui-react` component library (React 19 + Base UI `@base-ui/react` + Motion + Tailwind v4). Conventions: [packages/react/AGENTS.md](packages/react/AGENTS.md).

**Always read the scoped guide for the area you're working in** before writing code there.

## Docs site lives elsewhere

The documentation site (appica.dev, incl. the UI docs at `/ui`) is its own repo
at `~/Sites/appica-dev` — this repo is packages-only, like the sibling
`appica-icons` and `appica-country-flags` repos. The site consumes this library
as a **packed tarball** (`file:` dependency), not a workspace dependency. After
changing a component, rebuild + repack so the site can pick it up:

```
pnpm pack:react   # build + pack → packages/react/appica-ui-react-<version>.tgz
```

then reinstall in appica-dev (`pnpm install --force` there if the version didn't change).

## Common commands

- `pnpm --filter @appica/ui-react test | build | typecheck`
- `pnpm pack:react` — build + pack the tarball consumed by appica-dev

Interactive git flags (`-i`) aren't available; commit/push only when asked, and branch first if on the default branch.

## Where conventions live

These in-repo AGENTS.md files are the canonical, versioned source of project conventions (shared with collaborators and any agent that reads `AGENTS.md`). The docs site's authoring conventions live in the appica-dev repo's own AGENTS.md/ARCHITECTURE.md. The maintainer's Claude Code memory holds extended rationale and in-flight notes; stable rules are promoted here.

---
> Source: [appica-dev/appica-ui](https://github.com/appica-dev/appica-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
