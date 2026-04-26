## pi-harness

> My personal harness around [Pi](https://github.com/badlogic/pi-mono/) for coding-agent work.

# pi-harness

My personal harness around [Pi](https://github.com/badlogic/pi-mono/) for coding-agent work.

All packages in this repository use the `@aliou` scope where applicable, not `@anthropic` or `@anthropic-ai`.

## Structure

- `extensions/` - Private Pi extensions bundled in this repository
- `packages/` - Shared internal package code
- `tests/` - Test utilities and harness. See [tests/README.md](tests/README.md) for usage.

## Extensions

- `breadcrumbs` - Session history tools. Search past sessions, extract information, and hand off context to new sessions.
- `qq` - `/qq` quick-question command with custom message rendering and context filtering.
- `defaults` - Personal sensible defaults and quality-of-life improvements.
- `editor` - Owns the custom editor component and shared border-decoration rendering.
- `modes` - Hardcoded execution modes (balanced/plan/implement) with prompt families, tool gating, model/thinking defaults, and branch-aware restore.
- `palette` - Command palette with keyboard-driven UI for running commands and shortcuts.
- `planning` - Turn conversations into implementation plans and manage saved plans.
- `providers` - Register custom providers and show unified rate-limit and usage dashboards.
- `subagents` - Framework for spawning specialized subagents with custom tools, consistent UI rendering, and logging.

## Development

Uses pnpm workspaces. Nix environment available via `flake.nix`.

```sh
pnpm install
pnpm typecheck
pnpm lint
```

## Custom header

The startup header (`extensions/defaults/components/header.ts`) shows a curated list of harness shortcuts and commands. When adding a new `registerShortcut` or `registerCommand`, ask the user whether it should be added to the header.

## Notes

- This repo is my private Pi harness infrastructure first. Not every package here is intended to be published as a standalone package.
- Keep repository-level docs focused on my Pi harness. Extension-specific details belong in the extension README files.

---
> Source: [aliou/pi-harness](https://github.com/aliou/pi-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
