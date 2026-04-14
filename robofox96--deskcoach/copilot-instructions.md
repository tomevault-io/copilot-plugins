## deskcoach

> Purpose: Keep contributions consistent and aligned with project goals.


Purpose: Keep contributions consistent and aligned with project goals.

Style & Structure

Python ≥ 3.11; clear module split: core/, ui/, routines/, storage/.

Functions are small; single responsibility; docstrings explain why, not just what.

Logs are minimal in M1; structured logging later.

Privacy Guardrails (restate)

Never save frames; metrics only.

No outbound network calls in v1.

One-click purge; pause monitoring toggle.

Performance Guardrails

FPS target: 5–10 (6–8 typical).

CPU target: < 15% in normal use.

Confidence gating; pause when uncertain.

UX Guardrails

Sustained-condition detection before nudging.

Cooldowns prevent spam; DND-friendly.

Clear actions: Done / Snooze / Dismiss.

Tickets & PRs

Every task should state: Goal → Inputs (rules/workflows/docs) → Steps → Acceptance.

PR description must reference relevant rule/workflow and include a quick test note.

No scope creep: if a change touches multiple milestones, split tickets.

Testing (M1-level)

Manual scenarios: neutral sit, slouch 60s, lean 70s, out-of-frame, low light, dismiss twice.

Record CPU and nudge counts during a 30–60 min session.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/robofox96) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
