## cleantabs

> Use this file for repo-specific guidance that is not obvious from the file tree. Link to existing docs instead of duplicating them.

# CleanTabs Agent Notes

Use this file for repo-specific guidance that is not obvious from the file tree. Link to existing docs instead of duplicating them.

## Project shape

- CleanTabs is a WXT-based browser extension with a React popup UI.
- Product behavior, rule examples, and user-facing flows are documented in [README.md](README.md). Read that first for feature intent.
- Main execution surfaces:
  - `entrypoints/background.ts`: service worker, alarms, periodic cleanup, browser event wiring.
  - `entrypoints/popup/App.tsx`: popup shell and main UI tabs.
  - `lib/`: domain logic, storage helpers, rules, matching, tab and stash types.
  - `components/`: popup feature components.
  - `components/ui/`: shared presentational primitives; avoid mixing extension logic into these files.

## Commands

- Package manager: `pnpm`
- Install deps: `pnpm install` (`postinstall` runs `wxt prepare`)
- Dev: `pnpm dev`
- Dev for Firefox: `pnpm dev:firefox`
- Build: `pnpm build`
- Build for Firefox: `pnpm build:firefox`
- Typecheck: `pnpm compile`
- Tests: `pnpm test`

When validating a change, prefer the narrowest useful check. For logic changes in `lib/match-pattern.ts`, run `pnpm test`. For most TypeScript or UI changes, run `pnpm compile`.

## Code conventions

- Use the `@/*` path alias from [tsconfig.json](tsconfig.json) instead of deep relative imports when it improves clarity.
- Keep business rules in `lib/` and keep `components/` focused on rendering and user interaction.
- Treat storage access as a centralized concern in `lib/storage.ts`; prefer updating helpers there over introducing ad hoc storage calls.
- Rule persistence is defined by `lib/rule.ts`. UI-only fields such as transient editor state should not leak into persisted storage.
- URL pattern behavior is defined in `lib/match-pattern.ts`; keep [lib/match-pattern.test.ts](lib/match-pattern.test.ts) aligned with any matcher change.
- If a change affects alarms, permissions, CSP, or browser manifest behavior, inspect [wxt.config.ts](wxt.config.ts) in the same edit.

## Pitfalls

- The background workflow intentionally avoids acting on focused tabs and can skip pinned or grouped tabs based on settings. Preserve those guards when changing tab cleanup logic.
- Stash writes use helper APIs in `lib/storage.ts`; do not bypass them with blind overwrites.
- This is an extension project, so changes that look local can affect Chrome and Firefox builds differently. Keep browser-specific behavior explicit.

## Useful references

- Product behavior and rule syntax: [README.md](README.md)
- Extension configuration: [wxt.config.ts](wxt.config.ts)
- Background workflow: [entrypoints/background.ts](entrypoints/background.ts)
- Popup entry: [entrypoints/popup/App.tsx](entrypoints/popup/App.tsx)

---
> Source: [kagtua/CleanTabs](https://github.com/kagtua/CleanTabs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
