## agent-deck

> Read `PLAN.md`, `docs/ARCHITECTURE.md`, `docs/SECURITY.md`, and `WORK_LOG.md`

# Agent Deck engineering rules

Read `PLAN.md`, `docs/ARCHITECTURE.md`, `docs/SECURITY.md`, and `WORK_LOG.md`
before changing implementation code.

- Keep this project isolated from `codex-quota-lab` and research repositories.
- Never read, copy, persist, log, or transmit OpenAI, Codex, Anthropic, GitHub,
  or relay credentials. Spawn official CLIs and let them use their own login.
- Session text may cross the paired phone channel because that is the product's
  purpose. It must use an authenticated encrypted connection and must never be
  written to bridge logs, hook logs, crash reports, analytics, or pairing state.
- Bind to loopback by default. A non-loopback listener must require TLS,
  certificate pinning, a paired device token, and explicit user configuration.
- Expose typed Codex/Claude actions only. Do not add a generic remote-shell API.
- Preserve the local CLI's sandbox and approval policy. Never auto-approve,
  bypass permissions, or weaken a session's policy from the bridge.
- Every state-changing mobile request needs a client request ID. Unknown
  completion is not retry authority; reconcile session state before replay.
- Missing, stale, malformed, or unsupported state is `unknown` or an explicit
  error. Do not invent session, quota, goal, or completion values.
- Keep Codex App Server version/capability handling explicit because parts of
  its protocol are experimental. Additive adapters are preferred.
- Android baseline is API 26 unless the user changes it. The APK must be
  side-loadable and its core path must not require Google Play Services.
- Update docs and append `WORK_LOG.md` after each verified milestone.
- Before handoff, run formatting, lint, typecheck, unit/integration tests,
  plugin validation, Android tests, and an APK build. Report skipped gates.

---
> Source: [dltsum/agent-deck](https://github.com/dltsum/agent-deck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
