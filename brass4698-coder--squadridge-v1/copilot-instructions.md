## squadridge

> SquadRidge Core Application Rules

# SquadRidge — core rules

Cross-border dialogue app on the Enclave[ZK] stack (React, Vite, Supabase).

## Architecture

1. **ZK verification**: Semaphore proofs for identity checks. Do not log or expose raw PII.
2. **Sessions**: Time-bounded rooms; encrypt payloads; keep data minimal.
3. **Low bandwidth**: Fewer round-trips; handle offline and flaky networks.
4. **De-escalation UX**: “Power of Pause,” one-tap Slow down, calm copy.

## Frontend

- Functional components; hooks for state/side effects; strict TypeScript.
- **Look**: Deep Navy `#0A0F1E`, Space Grotesk headings, DM Sans body, `clamp()` for type.
- **Controls**: Teal `#0E9AA7` primary, Amber `#F5A623` warnings; clear hover/focus; WCAG AA contrast.
- **SVG**: Clear, consistent icons; no flags or military insignia.

## Backend

- Postgres + Supabase (RLS on by default). Matchmaking and queues live in SQL/RPC unless a worker is added.
- Optional Redis in `docker-compose.yml` is for local experiments only.
- Realtime: Supabase channels; respect membership and RLS.

## Security

- Message encryption follows `docs/security/threat-model.md` (application-layer keys today, not Signal-style E2E).
- Translation / tone features must fail soft: chat works if those paths error.
- Data design: minimize retention and harm (see threat model).

---
> Source: [brass4698-coder/squadridge-v1](https://github.com/brass4698-coder/squadridge-v1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
