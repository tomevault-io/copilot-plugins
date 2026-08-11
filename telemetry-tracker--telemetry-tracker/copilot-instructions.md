## pr-before-open

> Run Bugbot locally and verify merge gates before opening a GitHub PR


# Before opening a pull request

1. Run locally: `pnpm lint && pnpm test && pnpm build` (plus migrations if schema changed).
2. **Bugbot (paused):** the GitHub `bugbot-review` gate is temporarily a no-op (`BUGBOT_REVIEW_ENABLED=false` in [`.github/workflows/bugbot-review.yml`](../.github/workflows/bugbot-review.yml)). `/review-bugbot` remains optional. Re-enable by flipping that flag to `true` and ensuring the Cursor Bugbot app is active for this repo.
3. Only then push and open the PR.

**Merge gates:** `develop` uses **0 human approvals** + required checks `build`, `bugbot-review` (currently no-op success while paused), `maintainer-review`. Maintainer-authored PRs pass `maintainer-review` automatically.

---
> Source: [Telemetry-Tracker/telemetry-tracker](https://github.com/Telemetry-Tracker/telemetry-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
