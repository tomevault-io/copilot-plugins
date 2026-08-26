## bazaar

> Bazaar is a pre-development project. Do not infer or invent its product scope; use `context/STATUS.md` for the currently verified state.

# Bazaar Project Bootstrap

Bazaar is a pre-development project. Do not infer or invent its product scope; use `context/STATUS.md` for the currently verified state.

## Startup routing

1. When the parent AI workspace is accessible, follow its root `AGENTS.md` and `ROUTER.md`.
2. Otherwise, when GitHub access is available, read `Cryptographer2077/ai-workspace` root `AGENTS.md` and `ROUTER.md` before shared context is needed.
3. If neither source is accessible, continue with these project instructions and disclose when an answer may depend on unverified shared context.
4. Read `context/INDEX.md`, then load only relevant Bazaar context.
5. Read `context/STATUS.md` when current scope, progress, or the next move matters.

If Bazaar does not own the task, return to the workspace `ROUTER.md` when accessible. Otherwise stop and identify the missing route instead of searching unrelated repositories.

## Synchronization

Before editing a desktop checkout, inspect Git status and fetch the remote. Fast-forward a clean branch that is only behind. Stop and report local changes, divergence, authentication failures, or any state requiring a merge or rebase; never silently discard, stash, reset, merge, or rebase user work.

After an approved durable update, validate it and synchronize this independent repository. A mobile or remote session must use `Cryptographer2077/bazaar` rather than assuming the parent `ai-workspace` repository contains this project's files.

## Memory

- Keep verified Bazaar-specific requirements, decisions, status, and research in focused files under `context/`.
- Do not copy raw chat transcripts into the repository.
- Do not commit secrets, credentials, local caches, virtual environments, IDE state, or generated artifacts.

## Pickup and handoff

When asked to `pickup`, read `context/STATUS.md`, inspect Git status and recent history, and summarize the verified objective, completed work, blockers, and next move without changing files.

When asked to `handoff`, update `context/STATUS.md` with verified progress, blockers, Git state, and the recommended next move. Synchronize under the established publishing policy.

---
> Source: [Cryptographer2077/bazaar](https://github.com/Cryptographer2077/bazaar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
