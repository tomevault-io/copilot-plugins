## pi-vis

> This file is intentionally short. It contains rules that must always be loaded. Architecture, rationale, and feature-specific details live in `docs/` and must be loaded on demand through `docs/agent-index.md`.

# AGENTS.md — Pi-Vis Agent Instructions

This file is intentionally short. It contains rules that must always be loaded. Architecture, rationale, and feature-specific details live in `docs/` and must be loaded on demand through `docs/agent-index.md`.

## Mandatory workflow

- Before editing code, read `docs/agent-index.md` and then read only the docs it routes you to for the files or behavior you will touch.
- Do not dump broad architecture notes back into this file. If a change needs design context, update the relevant file under `docs/` or add an ADR under `docs/decisions/`.
- Keep changes small and consistent with the existing architecture.
- Update documentation in the same change when you alter architecture, IPC/protocol contracts, state shape, testing strategy, UI conventions, theming, or significant user-visible behavior.
- When adding or moving major subsystems, update `docs/agent-index.md` so future agents know which docs to read.

## Verification requirements

- During development, run focused checks relevant to the change.
- Before reporting completion, run the full verification suite:
  - `npm run typecheck`
  - `npm run lint`
  - `npm test`
  - `npm run test:e2e`
- For documentation-only changes, tests may be skipped; explicitly state that they were skipped because no runtime code changed.
- If any required command cannot be run or fails for reasons outside the change, report the command, the reason, and what remains unverified.
- Add or update tests for behavior changes.
- Add or update E2E/render tests for significant user-facing functionality.

## Hard project rules

- Keep `src/shared/ipc-contract.ts` and `src/shared/pi-protocol/` as the typed source of truth for main↔renderer and pi RPC contracts.
- Do not import private Pi internals from the SDK host except the exact-version
  `resources/pi-session-host/pinned-pi-private.mjs` adapter, which may load only
  Pi 0.83.0's hidden `llama.cpp` built-in registry entry. All injection and
  runtime interaction after that lookup must use Pi's public surface. See ADR
  0006 and `docs/compatibility/pi-0.83.0.md`.
- Preserve project-trust deny-by-default behavior for project-local resources.
- Use semantic theme tokens only. Do not add palette-specific CSS variables or component-level Catppuccin swatch references.
- Use shared SVG icon components from `components/common/icons.tsx`; do not add text glyphs for chrome icons.
- Use `FadeText` for truncatable single-line labels.
- Keep Zustand updates immutable; create new `Map` instances rather than mutating existing maps in place.
- Keep transcript reducer logic pure and side-effect free.
- Use `edit` for precise file edits and `write` only for new files or full rewrites.

## Useful entry points

- Documentation routing: `docs/agent-index.md`
- Project overview: `docs/architecture/overview.md`
- Testing guide: `docs/testing.md`
- Release process: `RELEASING.md`

---
> Source: [rsingapuri/pi-vis](https://github.com/rsingapuri/pi-vis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
