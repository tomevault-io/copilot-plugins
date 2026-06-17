## wasp-restart-after-changes

> Restart Wasp server after changes for immediate testing


# Wasp Restart Policy

- Do **not** restart on every edit.
- Restart only when needed for accurate testing, such as:
  - runtime-impacting server/client changes that are not reflected by current process state,
  - env var changes (`.env.client`, `.env.server`),
  - Wasp config/routing/build pipeline changes (`main.wasp`, generated output issues),
  - crashed/unhealthy dev server.
- Default recovery flow: `wasp start`.
- If Wasp reports generated output issues (for example missing `.wasp/out` artifacts, stale generated files, or migration-path errors under `.wasp/out`), run `wasp clean` and then `wasp start`.
- If ports are occupied, stop conflicting processes and then restart Wasp.
- After a needed restart, confirm both client and server are up and share reachable network URL(s) for phone testing.

---
> Source: [cvetelinandreev/reShkolo](https://github.com/cvetelinandreev/reShkolo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
