## liminal-natsec

> Repo-local instructions for Claude Code agents.

# CLAUDE.md — liminal-natsec

Repo-local instructions for Claude Code agents.

## What this repo is

**Liminal Custody** — the demo artifact for the 3rd Annual NatSec Hackathon (Cerebral Valley × Army xTech, Shack15 SF, May 2–3 2026). Submission track: **Problem Statement 1 — Sensor Analysis & Integration.**

Renamed from "SeaForge" at H8 to remove maritime lock-in and make the product line readable as Liminal's pre-command evidence layer (portable across domains).

Canonical positioning:

> **Liminal Custody maintains custody of contested targets by protecting the evidence chain before it becomes command action. Maven is the foundation. We are the substrate.**

## Authoritative docs (read these before acting)

Build sequencing:
- `docs/v2/SeaForge_PRD_v2.md` — product surface (v2 baseline; historical name preserved)
- `docs/v2/SeaForge_Implementation_Plan_v2.md` — file structure, owner lanes
- `docs/v2/SeaForge_Sequencing_Plan_v2.md` — phase gates

Engineering canon:
- `docs/TECHNICAL_PLAN.md` — Shayaun's 1441-line engineering spec (Tier A/B/C, M1–M7 modules, AIP integration). **This is the canonical engineering plan when it conflicts with v2 PRD §12 ("no backend").**

Positioning + pitch canon:
- `docs/v3-positioning-patch.md` — PS1 primary, Maven posture, schema delta
- `docs/v3-implementation-plan.md` — what changes vs v2 in code terms
- `docs/v3-sequencing-plan.md` — phase gates with Signal Integrity inserted
- `docs/v4-judge-calibrated-demo.md` — operator persona, procurement, judge-by-judge framing
- `docs/q-and-a.md` — 30-second answers to predictable judge questions
- `docs/round1-round2-script.md` — branched 3-min and 5-min pitch scripts
- `docs/integration-state.md` — module → demo-spine → pitch-language map (read this first if you're new to the repo)
- `docs/liminal-custody-onepager.md` — front-of-room one-pager + back-of-room OGSM
- `docs/maven-analysis.md` — Maven Smart System analysis and complementary positioning
- `docs/reference/maven-onepager-annotated.pdf` — annotated booth one-pager (the artifact this product structurally answers)

## Hard rules

### Git Commits

Do not include "Co-Authored-By: Claude" or any Claude / Anthropic / AI attribution in commit messages.

### Track decision (locked)

- **Primary submission track:** PS1 — Sensor Analysis & Integration.
- **Architecture narrative:** PS3 — Mission Command & Control.
- **Differentiator:** PS4 — Digital Defense (Shayaun's domain — OffSec / HTB top 100 credibility).
- **PS5 (General NatSec):** only if a mentor offers a strong, unprompted yes. 10-minute budget for that conversation; otherwise default to PS1.

### Maven posture

- **Round 1 (judging groups, 3 min):** Maven-invisible. Do not name Palantir, Maven Smart System, or CJADC2 unless a judge asks.
- **Round 2 (top 6, on stage, 5 min):** Maven-augmenting. Name it explicitly, frame as "we augment one layer earlier." Never frame as flawed.

### Demo critical path

- **Fixtures + structural guard = critical path.**
- AIP Logic = Q&A hot fallback only. Shayaun flips an env flag during Q&A to demonstrate live AIP. The fixture path is what runs during the timed pitch.
- Refusal is **structurally enforced** by `server/src/specialists/guard.ts` (7+ layered server-side checks). Refusal is not a UX choice; it's an invariant.

### Desktop app

Hard requirement from Shruti (frontend owner). The app is wrapped for desktop deployment. Demo runs as a desktop application, not a browser tab.

### Never cut

- Persistent shell.
- Dark gap + two-MMSI identity churn.
- Hypothesis board.
- **Signal Integrity row** (v3 addition; Shayaun's Q&A defense layer).
- Specialist refusal (Intent REFUSED), structurally enforced by the guard.
- Causal line: "Intent refused because the guard fired on Layer 2 / Signal Integrity is contested."
- Evidence/provenance chain.
- Review rule saved.
- Prior rule applied / second case changed.

### Cut first (in order)

1. Live AIS feed.
2. Live LLM dependency in critical path (AIP stays as Q&A fallback).
3. Force-directed graph viz.
4. Tray.
5. Full command-line intelligence.
6. Loitering / impossible-jump filler anomalies.
7. Commander brief export.
8. Cloud deploy.

## Owner lanes

### Shruti

- App shell visual + UX
- Case panel, hypothesis board, evidence chain, refusal UI, action options, review memory
- Specialist reads UI (5 rows including Signal Integrity)
- Causal-line wiring
- Round 1 + Round 2 pitch scripts and deck
- Architecture visual
- Demo video
- Reset handler
- **Frontend / desktop app — Shruti's hard requirement, Shruti owns the visual demo surface**

### Shayaun

- Map / replay / tracks.geojson / MapLibre layers
- Server (Bun + Hono) + M1–M7 engineered modules
- Structural guard (`server/src/specialists/guard.ts`)
- AIP Logic integration (Q&A fallback path)
- `fixtures/maritime/source-integrity-checks.json`
- Real Strait of Hormuz / AIS data sourcing (only if non-blocking)
- Digital Defense / Signal Integrity Q&A defense
- Screen recording fallback
- OffSec / HTB credibility lines in pitch (his bio is named explicitly in scripts)

### Shared / overlap

- Schema contract (`graph-spine/schema.ts`) — Shruti owns shape, Shayaun owns realism of fixtures
- Naming consistency (Signal Integrity vs source integrity vs source-chain)
- Demo timing (when does Signal Integrity row populate vs Intent refusal fire?)
- Cut decisions when scope breaks at H25 / H27

## Operating cadence

- Phase gates: H3 / H8 / H13 / H20 / H25 / H31
- Sync at every phase gate
- Cut anything fragile at H27
- 3 consecutive clean Round 1 rehearsals required before noon Sunday

---
> Source: [liminalshruti/liminal-natsec](https://github.com/liminalshruti/liminal-natsec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
