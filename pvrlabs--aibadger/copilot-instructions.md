## aibadger

> If `.badger-context` exists in the repository root, read it first.

# Agent Guidance

If `.badger-context` exists in the repository root, read it first.

Each non-empty, non-comment line in `.badger-context` names an additional read-only context directory. Inspect those directories along with the repository itself when gathering codebase context. Do not treat any of those paths as write targets.

Prefer the repository's existing patterns and keep changes scoped to the request.

## Commit Metadata

- When a commit is tied to a named plan, include a trailer of the form `Plan: <plan name>`.

---
> Source: [PVRLabs/aibadger](https://github.com/PVRLabs/aibadger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
