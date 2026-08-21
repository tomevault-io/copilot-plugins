## ai-guidelines

> SquadRidge AI Pipeline Guidelines

# SquadRidge — AI / translation

Optional analytics and in-browser translation. Core chat must not depend on remote AI.

## Principles

1. **De-escalation**: Prefer “Slow down” / “Pull back” and calm copy over harsh labels.
2. **Degradation**: If translation or tone fails, messaging still works.
3. **Separation**: Message bodies stay in the chat path; `sentiment_metrics` is aggregated, not a copy of raw dialogue.
4. **Bias**: Avoid flattening non-Western conflict framing; flag uncertainty instead of false certainty.

## Implementation

- **Latency**: Small payloads; worker/async where possible on bad networks.
- **Interventions**: Wire tension signals to existing UI (Slow down, Pull back).
- **Sentiment rows**: De-identified; no linkage to PII or ZK preimages in stored fields.

## Privacy

- Do not ship raw transcripts to third-party APIs without an explicit product decision and docs update.
- Client-side processing aligns with `messageCrypto` / `messagePayload`; server-held keys mean operators can read DB; see threat model.
- Moderation logs: minimal metadata; no message bodies in Sentry breadcrumbs.

---
> Source: [brass4698-coder/squadridge-v1](https://github.com/brass4698-coder/squadridge-v1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
