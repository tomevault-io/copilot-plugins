## pi-extension-starter

> This repository is a starter for building `pi` extensions as source-first npm packages.

# AGENTS

## Objective

This repository is a starter for building `pi` extensions as source-first npm packages.

When implementing new features here, treat the following materials as the source of truth:

- `docs/pi-guide-1-base.md`
- `docs/pi-guide-2-runtime.md`
- `docs/pi-guide-3-release.md`
- `README.md`
- `src/index.ts`
- `src/commands/settings-command.ts`
- `src/state/session-state.ts`
- `src/tools/starter-checklist-tool.ts`
- `src/ui/footer.ts`
- `src/ui/settings-panel.ts`

The `docs/pi-guide-*.md` guides were derived from the official `pi` materials in:

- `packages/coding-agent/docs/extensions.md`
- `packages/coding-agent/docs/packages.md`
- `packages/coding-agent/examples/extensions/README.md`

## Rules for Creating Pi Extensions

- Keep the package source-first: the main entrypoint must remain in TypeScript and be listed in `package.json > pi.extensions`.
- Prefer `defineTool()` for reusable and testable tools.
- Use `@sinclair/typebox` for schemas and `StringEnum` from `@mariozechner/pi-ai` when a string enum compatible with Google providers is needed.
- Register commands with `pi.registerCommand("name", { ... })`.
- For custom messages, use `pi.registerMessageRenderer()` together with `pi.sendMessage()`.
- For interactive UI, prefer `ctx.ui.custom()` with `SettingsList` and `getSettingsListTheme()` when the use case is configuration or toggles.
- Reapply state-derived behavior on `session_start` and `session_tree`, and react to model changes on `model_select` when the extension depends on that.
- Use `ctx.ui.setStatus()` for a short summary and `ctx.ui.setFooter()` when you need to replace the entire footer.
- If the extension has state that must survive `fork`, `tree`, and session reconstruction, persist it in tool result `details` or session entries, not only in loose process state.
- Runtime dependencies must go in `dependencies`. Core `pi` dependencies must stay in `peerDependencies` with `"*"`.
- Do not introduce a mandatory build step for `pi` to load the extension unless there is a concrete reason.

## Quality

- After changes that are not documentation-only, run `npm run check`.
- To validate the real extension flow, also use `pi -e ./src/index.ts`.
- Before publishing, confirm `npm run pack:check`.
- Keep repository text files normalized to LF via `.gitattributes` so `biome format` and Git hooks behave consistently across platforms.
- Never bypass Git hooks with `git commit --no-verify`. If a commit is blocked by hooks, fix the underlying issue or ask the user how to proceed instead of skipping validation.

## Delivery

- Update `README.md` when the extension's public API changes.
- Update `docs/pi-guide-*.md` when new architecture decisions or `pi` patterns are incorporated into the starter.
- Update this `AGENTS.md` when the starter's baseline files change.

---
> Source: [vinirabli/pi-extension-starter](https://github.com/vinirabli/pi-extension-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
