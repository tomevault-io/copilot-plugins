## bugbot-review

> Remind agent to align with Bugbot rules and pre-push review for high-risk areas


# Bugbot-aligned changes

When editing notifications, alerts, billing, ingest, auth, or plan enforcement:

1. Read `.cursor/BUGBOT.md` and the scoped file under `apps/api/.cursor/` or `apps/dashboard/.cursor/`.
2. Keep quota session context, ingest alerts, and bell feed behavior in sync (thresholds, routing, dedupe ids).
3. Split dashboard server-only code into `*-server.ts` — never import `next/headers` from client-shared `lib/`.
4. Add or update Vitest coverage for non-trivial lib behavior.
5. Before suggesting the PR is done, recommend the user run `/review-bugbot` locally for cross-file logic changes (optional while the GitHub `bugbot-review` gate is paused — see `.github/workflows/bugbot-review.yml`).

---
> Source: [Telemetry-Tracker/telemetry-tracker](https://github.com/Telemetry-Tracker/telemetry-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
