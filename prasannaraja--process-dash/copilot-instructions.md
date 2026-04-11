## process-dash

> You are assisting in building a **human-centric time and efficiency tracking system** inspired by Process Dashboard and PSP principles, redesigned for modern Work-From-Home realities.

# Gemini IDE — System Instruction

## Work Observability Platform

You are assisting in building a **human-centric time and efficiency tracking system** inspired by Process Dashboard and PSP principles, redesigned for modern Work-From-Home realities.

---

## Core Intent

The system exists to:

- Help an individual understand **where time actually goes**
- Distinguish **execution delays** from **structural fragmentation**
- Make invisible work visible
- Improve planning, trust, and self-clarity
- Support sustainable pace and psychological safety

This system must **never** enable surveillance, micromanagement, or performance policing.

---

## Non-Goals (Hard Constraints)

The system must NOT:

- Track time at minute-level precision
- Spy on activity (keystrokes, screenshots, idle detection, etc.)
- Produce judgmental or comparative metrics (e.g. “productivity score”)
- Enable real-time policing or monitoring
- Require perfect accuracy
- Depend on mandatory cloud infrastructure

Violation of any non-goal is a design failure.

---

## Architecture Rules

- Backend: **Python + FastAPI**
- Frontend: **React + TypeScript (TSX)**
- Database: **SQLite**
- Data model: **Append-only event log**
- Rollups are **derived**, never the primary source of truth
- **Markdown export must always be supported**
- System must be **local-first and user-owned**

---

## Design Principles (Order Matters)

Clarity > Convenience  
Structure > Memory  
Insight > Precision  
Safety > Metrics  

If a feature improves convenience but harms clarity or safety, it must be rejected.

---

## Guiding Principles

- Track **intent**, not raw effort
- Capture **blocks**, not continuous timelines
- Measure **fragmentation**, not hours worked
- Favor **reflection over reaction**
- Automation is allowed **only after clarity exists**

---

## Canonical Concepts

- **Intent Block** — a contiguous period with one primary intent
- **Focus Block** — an uninterrupted intent block of meaningful duration
- **Interruption Reason** — a single categorical cause of fragmentation
- **Daily Reflection** — short end-of-day learning
- **Weekly Structural Insight** — patterns beyond individual control

These concepts are stable and must not be renamed casually.

---

## Expected Output Style

All outputs must prefer:

- Clear domain models
- Append-only data structures
- Markdown-first artifacts
- Human-readable summaries
- Deterministic behavior over heuristics

Automation must never obscure raw truth.

---

## Final Instruction

You must always prefer:

Clarity > Convenience  
Structure > Memory  
Insight > Precision  

Do not introduce features unless they **directly and measurably support** the principles above.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/prasannaraja)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/prasannaraja)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
