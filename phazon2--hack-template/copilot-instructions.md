## hack-template

> - Cloud container is scratch space, never storage.

# Execution rules for this repo

## Environment
- Cloud container is scratch space, never storage.
  Only pushed git state survives.
- WebSearch works (server-side). WebFetch may be blocked.
- git push works. Do not report it as a blocker without trying.

## Deploy — read before claiming no public URL is possible
Git-triggered deploy does NOT require container egress.
Provider watches the repo, builds on its own infra on push.
Chain: push -> GitHub -> provider builds -> public URL.
That URL is also the webhook receiver.
Never conclude a public URL is impossible.

## Evidence discipline
- A mock passing does not license a real call.
  Mocks encode assumptions; they are scaffolding, not evidence.
- Receipts, probe outputs, CI artifacts must NEVER be
  generated from a mock.
- A mock's only job: make the day-you-get-tokens work be
  "change base URL, run probe" — not "start building."

## Credentials
- Never handle my personal credentials.
- Never drive a signup, 2FA, or SSO flow.
- If something needs an account, stop and tell me —
  I do it myself in five minutes on my phone.

## Blockers
If you hit a blocker, state the exact error and whether it is:
config-fixable / do-it-myself / genuinely human-only.
Do not collapse those three into "blocked."

---
> Source: [phazon2/hack-template](https://github.com/phazon2/hack-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
