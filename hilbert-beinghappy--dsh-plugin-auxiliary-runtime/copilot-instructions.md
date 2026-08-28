## dsh-plugin-auxiliary-runtime

> This repository implements `docs/task-book.md`, which is the normative product contract.

# Repository instructions

This repository implements `docs/task-book.md`, which is the normative product contract.

## Product boundaries

- Package name `dsh-plugin-auxiliary-runtime`, published version `0.1.1`, implementation branch `codex/auxiliary-runtime`. Standalone adoption evidence lives on `codex/independent-adoption` and does not bump a new Release.
- Official `tokenUsage` remains the authority for Agent-loop calls. This plugin owns only the auxiliary domain and exposes `official`, `auxiliary`, and `combined` values with provenance intact.
- Never register or replace the `tokenUsage` projection, append Session events, fabricate assistant events, create or resume a hidden Session, patch installed `node_modules`, or replace official bundle rows.
- Persist through the official `storageDomain` service under domain `auxiliary_runtime`. The bundle patch inserts only this plugin's own `auxiliary-runtime` row.
- Bind every call to an already-live Session and fence records with `session.header.createdAt`. Never call `sessions.create`.
- Persist no prompts, messages, model output, custom answers, credentials, environment values, or filesystem paths. Durable rows contain only identifiers, purpose, status, token buckets, normalized failure facts, and timestamps.
- Cancellation uses `AbortController`; provider failures preserve official `LlmFailure` codes. Active calls and reservations are process-local. Orphaned `running` rows become `interrupted` on the next initialization.
- Limits are honest reservations over recorded auxiliary usage. Do not claim the provider's final usage is knowable before dispatch.
- The supported deployment is one Host process per `DSH_HOME`; document the official storage-domain single-process limitation.

## Package and compatibility

- Consumer-facing metadata contains no `workspace:` dependencies.
- Pin the exact official dsh component range actually tested. Unknown hosts fail closed with an actionable capability result.
- Use the repository package manager and Node `^22.19.0 || >=24`.
- Keep credentials, Session data, Profiles, `.env`, AppleDouble files, caches, temporary homes, and generated packages out of Git and release assets.

## Verification

- Add focused success, failure, concurrency, idempotency, cancellation, crash-recovery, limits, privacy, and capability tests.
- Test pack contents and install/boot/remove/reinstall under an isolated `DSH_HOME` against unmodified official dsh releases.
- Standalone T1 (no SeekTTY, no Clarify) uses the published `0.1.1` tgz plus `SHA256SUMS` on stock `@deepseek-ai/dsh@0.1.1-rc.2`. The T1 Host pin is the separately observed `dsh --version` plus `evidence.dshVersion`. Plugin-internal version reporting is unconfirmed on stock rc.2 (`hostConfirmed=false`); do not call that host-confirmed. Offline inspector tests run in CI; the real `--write` lifecycle is a local Codex job and must not touch `~/.dsh`.
- A joint Release still requires real integration with Clarify and SeekTTY, Grok and Codex review, clean CI, tgz plus `SHA256SUMS`, and post-Release download/reinstall verification. Standalone T1 does not create or recommend a new joint baseline.

---
> Source: [Hilbert-beinghappy/dsh-plugin-auxiliary-runtime](https://github.com/Hilbert-beinghappy/dsh-plugin-auxiliary-runtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
