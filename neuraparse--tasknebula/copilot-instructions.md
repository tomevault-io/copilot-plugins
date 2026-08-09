## tasknebula

> Public repository hygiene and local-only workspace boundaries


# Keep the mobile workspace local

- The root `mobile/` directory is intentionally local-only. Never add or
  force-add it, stage/commit/push any descendant, or copy its source into a
  tracked artifact.
- Do not list `mobile/` in public workspace manifests, lockfiles, CI, or release
  steps. Work inside it only when the user explicitly requests local mobile
  work; use the workspace and lockfile contained inside that directory.
- Run `pnpm hygiene:check` before committing or publishing repository changes.

---
> Source: [neuraparse/taskNebula](https://github.com/neuraparse/taskNebula) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
