## bejeweled-solver

> This document defines **strict behavioral, architectural, and coding constraints** for OpenAI Codex when contributing to this repository.

# AGENTS.md

## OpenAI Codex – Agent Operating Instructions

This document defines **strict behavioral, architectural, and coding constraints** for OpenAI Codex when contributing to this repository.

Codex is authorized to generate, refactor, and test code **only within the boundaries defined here**. Any deviation is considered a failure of compliance.

---

## Project Summary

This project is a **computer-vision–driven Bejeweled-style solver** that operates externally to the game client.

High-level pipeline:

1. Capture screen frames (no memory hooks, no APIs)
2. Convert pixels into a symbolic board representation
3. Simulate game rules deterministically
4. Evaluate all legal moves
5. Execute the best move using automated input

This is a **reasoning system**, not a game clone.

---

## Codex Operating Mode

Codex must operate in **SAFE / ARCHITECTURE-PRESERVING MODE** at all times.

Codex must:
- Respect existing folder structure
- Preserve separation of concerns
- Favor clarity and correctness over novelty
- Avoid speculative or “clever” redesigns

If instructions are ambiguous, Codex **must stop and ask**.

---

## Non‑Negotiable Architectural Boundaries

### Layer Separation (Absolute Rule)

| Layer        | Purpose                                  | Allowed Imports                     | Forbidden Imports                   |
|-------------|------------------------------------------|-------------------------------------|-------------------------------------|
| `vision/`   | Screen capture & gem recognition          | cv2, numpy, mss, PIL                | AI, heuristics, game logic          |
| `game/`     | Deterministic rules & simulation          | Python stdlib only                  | CV, ML, UI automation               |
| `ai/`       | Evaluation & decision-making              | numpy, math, ML libs                | OpenCV, screen capture              |
| `control/`  | Mouse / keyboard execution                | pyautogui, time                     | AI logic, board evaluation          |

If Codex produces code that violates this table, it is **invalid output**.

---

## Single Source of Truth

- `BoardState` (symbolic representation) is the **only** object passed between layers
- Images must never enter `game/` or `ai/`
- AI logic must never reference screen coordinates or pixels
- Control logic must never evaluate move quality

---

## Determinism Requirements

- Game simulation must be deterministic by default
- All randomness must be:
  - Explicit
  - Seedable
  - Injectable
- No global mutable state
- No hidden side effects

If behavior cannot be unit tested, it is incorrect.

---

## Coding Standards

### Python Version
- Python 3.11+

### Style
- Explicit type hints required for public functions
- Prefer `@dataclass(frozen=True)`
- Avoid mutation unless strictly necessary
- No magic numbers; use constants or config

### Naming
- `snake_case` for variables and functions
- `PascalCase` for classes
- No abbreviations unless industry-standard

---

## What Codex MAY Do

Codex is allowed to:
- Implement modules as explicitly requested
- Add unit tests for new or modified logic
- Refactor internals without changing public interfaces
- Add helper utilities under `utils/`
- Improve performance **after** correctness is proven

---

## What Codex MUST NOT Do

Codex must NOT:
- Redesign the system architecture
- Merge layers into a single module
- Introduce new frameworks or dependencies without approval
- Add ML-based solutions without a deterministic baseline
- Remove logging, debug hooks, or test scaffolding
- Modify folder structure without permission

---

## Vision-Specific Rules

- OCR is optional and discouraged for gem recognition
- Template matching or color classification preferred
- Vision modules must expose **confidence scores**
- Debug frame output must be preserved

Never “guess silently” in vision logic.

---

## AI-Specific Rules

- Provide heuristic evaluators before ML
- AI decisions must be explainable
- Each evaluated move must have a score and rationale
- Search depth and time limits must be configurable

ML models (if added) must:
- Train on simulated boards only
- Never train on live screen data
- Be optional at runtime

---

## Control Layer Rules

- All inputs must be human-like
- Include configurable delays and jitter
- Never spam inputs
- Safety checks must prevent invalid moves

---

## Testing Obligations

Codex must add or update tests when modifying:
- Match detection
- Cascades / gravity
- Move generation
- AI ranking logic

Game simulation must be testable **without vision or control layers**.

---

## Logging & Debugging

- Use structured logging (not print)
- Log AI decision scores and rankings
- Log vision confidence failures
- Fail fast on invalid states

---

## Documentation Expectations

Each module must include:
- Purpose
- Inputs and outputs
- Assumptions and limitations

Complex logic must include reasoning comments.

---

## Definition of Success

A contribution is successful only if:
- Board states are correct and reproducible
- AI choices are explainable
- Behavior is deterministic under test
- Failures are observable and debuggable

---

## Final Directive to Codex

Do not optimize early.  
Do not be clever.  
Do not assume.

**Make state explicit.  
Make behavior testable.  
Make decisions explainable.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jparrott-sts)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jparrott-sts)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
