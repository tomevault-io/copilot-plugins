## dryui

> Repo-wide instructions for AI coding agents working in this repository.

# AGENTS.md

Repo-wide instructions for AI coding agents working in this repository.

## Canonical Docs

- Public overview: [`README.md`](./README.md)
- Contributor workflow: [`CONTRIBUTING.md`](./CONTRIBUTING.md)
- CSS discipline and token rules: [`packages/ui/skills/dryui/rules/theming.md`](./packages/ui/skills/dryui/rules/theming.md)
- Accessibility policy: [`ACCESSIBILITY.md`](./ACCESSIBILITY.md)
- Release flow: [`RELEASING.md`](./RELEASING.md)

## Canonical Sources

- Editor install snippets and MCP config examples live in [`apps/docs/src/lib/ai-setup.ts`](./apps/docs/src/lib/ai-setup.ts). Do not duplicate them elsewhere.
- Skill sources live in:
  - [`packages/ui/skills/dryui/`](./packages/ui/skills/dryui/)
  - [`packages/feedback/skills/live-feedback/`](./packages/feedback/skills/live-feedback/)
  - [`packages/cli/skills/init/`](./packages/cli/skills/init/)
- Sync generated skill copies with `bun run sync:skills`. Do not edit `packages/plugin/skills/` or `.cursor/rules/` directly.
- The local plugin source is [`packages/plugin`](./packages/plugin). `/plugins` refers to the in-app Codex or Claude install flow, not a repo directory.

## Repo Rules

- Use `gh-axi` for GitHub and `chrome-devtools-axi` for browser automation.
- DryUI is pre-alpha. Prefer the current shape over compatibility shims unless a task explicitly asks for backwards compatibility.
- Use the DryUI CLI as the default entry point:

```bash
bun install -g @dryui/cli@latest
dryui
```

- Use `dryui check [path]` or MCP `check` for static validation: component contracts, a11y, tokens, CSS discipline.
- For design-quality work (brief, critique, polish, visual review, anti-pattern detection), DryUI delegates to [impeccable](https://impeccable.style). Install alongside DryUI via `dryui init` or `npx impeccable skills install`, then use `/impeccable <command>` from your AI harness. `PRODUCT.md` and `DESIGN.md` at the project root are impeccable-owned; DryUI tools do not read or write them.
- Keep root-level Markdown durable. One-off audits, scratch TODOs, and generated reports belong under `docs/`, `reports/`, or ignored local directories, not the repo root.
- Repo-local editor install output such as `.agents/skills/`, `.github/skills/`, `.opencode/`, and `opencode.json` is not canonical source.

## Isolated Testing

- Use `bun vm:test` (one-shot scaffold + build) or `bun vm` (scaffold + Vite dev with HMR) to exercise the public `bunx @dryui/cli` flow in a throwaway [smolvm](https://github.com/smol-machines/smolvm) microVM. Both wrappers live in the root `package.json`; source and gotchas (ephemeral-only, Vite-under-node, stdout buffering, `--host 0.0.0.0`) are in [`scripts/vm.ts`](./scripts/vm.ts).
- While `bun vm` is running, use `bun vm:exec <cmd>` in any other tab to run commands inside the live VM (e.g. `bun vm:exec dryui list`). It relays via a shared-volume request/response loop; see [`scripts/vm-exec.ts`](./scripts/vm-exec.ts). `smolvm machine exec` itself does not work against ephemerals.
- Install smolvm via `curl -sSL https://smolmachines.com/install.sh | /bin/bash` (use `/bin/bash` explicitly; a Homebrew Intel `bash` first on PATH reports the wrong arch). Ensure the extracted `agent-rootfs` lives at `~/Library/Application Support/smolvm/agent-rootfs`.

## Verification

- After editing `.svelte` files in `packages/ui/`, run `bun run --filter '@dryui/ui' build`.
- For docs work, prefer the root wrappers so local runs match CI: `bun run docs`, `bun run docs:build`, `bun run docs:check`, and `bun run build:docs`.
- After changing skill content or setup guidance, run `bun run sync:skills`.

---
> Source: [rob-balfre/dryui](https://github.com/rob-balfre/dryui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
