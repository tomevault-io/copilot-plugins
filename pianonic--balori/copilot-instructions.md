## balori

> Living notes about non-obvious things in this repo. Keep this short and only add things that are noteworthy enough to matter for future work.

# Notes for Claude (and humans)

Living notes about non-obvious things in this repo. Keep this short and only add things that are noteworthy enough to matter for future work.

## Project

Expo SDK 54 React Native app using TypeScript, Expo Router (file-based routing), and bun as the primary package manager. Also works with npm/yarn.

## Running

```sh
bun run start          # dev server
bun run start --tunnel # for Expo Go on different networks
bun run web            # web only
```

For Expo Go on a physical device, SDK 54 is required (SDK 55 not yet supported by current Expo Go versions).

## Workflow rules (enforced)

- **Never work on `main`.** Create an issue (labeled) → branch `feature/<issue#>_PascalCase` or `fix/<issue#>_PascalCase` → PR (labeled) with `Closes #<issue>` → squash-merge + delete branch.
- **Use CLI generators whenever one exists.** `npx create-expo-app`, `npx expo install`, `gh issue create`, `gh pr create`, etc.
- **No AI / Claude attribution** in commits or PRs. Ever.
- **No test plans in PRs.** PR body is Summary + `Closes #<issue>` only.
- **Commit subject**: short imperative.
- **PR labels**: `bug`, `enhancement`, `refactor`, `stale`.

---
> Source: [PianoNic/Balori](https://github.com/PianoNic/Balori) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
