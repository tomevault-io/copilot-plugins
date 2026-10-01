## grok-bot-cli

> Publish exclusively through `.github/workflows/release.yml` with GitHub OIDC.

# Agent instructions

Publish exclusively through `.github/workflows/release.yml` with GitHub OIDC.
Its package release script is a workflow entrypoint: never publish locally,
request npm publishing tokens/login, or troubleshoot local publishing auth.
Use the existing Changesets flow and verify Actions plus the registry before
updating installations.

Delete dead code, obsolete scripts, duplicate workarounds, and unused legacy
installs. Check active owners/references first; preserve auth, operator env,
durable data, and intentional duplicate suppression. Prefer existing helpers,
stdlib, and native features over new wrappers or fallback frameworks.

---
> Source: [ScriptedAlchemy/grok-bot-cli](https://github.com/ScriptedAlchemy/grok-bot-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
