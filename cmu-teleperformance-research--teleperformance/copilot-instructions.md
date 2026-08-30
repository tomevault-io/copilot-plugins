## teleperformance

> This file documents the codebase architecture, active conventions, and things to know before making changes.

# CLAUDE.md — CSR Conflict Resolution Simulator

This file documents the codebase architecture, active conventions, and things to know before making changes.

## Quick orientation

The app is an AI-powered customer service training tool. The LLM plays a difficult customer; a human trainee plays the CSR. The backend assembles a system prompt from modular files and routes LLM calls. The frontend is a React SPA with a 4-step session wizard and a split-pane chat + workflow portal.

**Entrypoints:**
- Backend: `backend/main.py` (FastAPI app)
- Frontend: `frontend/src/App.jsx` (top-level state machine and router)
- LLM logic: `backend/services/llm_service.py`

---

## Prompt system architecture

The system prompt is assembled from independent, composable files. **No prompt logic lives in Python strings** (except the COACHING_SIGNAL per-turn injection, which is dynamic).

### Prompt directory layout

```
backend/prompts/
├── scenarios/       # WHO the customer is and WHAT happened — factual context only
│   ├── flight_cancellation.txt
│   ├── baggage_delay.txt
│   ├── loan_delay.txt
│   └── refund_request.txt
│
├── emotions/        # HOW the customer feels and behaves — no scenario facts
│   ├── angry.txt
│   ├── confused.txt
│   ├── demanding.txt
│   └── anxious.txt
│
├── shared/          # Rules that apply to all prompts
│   ├── system_rules.txt     # Role constraint, character integrity, natural speech
│   ├── behavior_rules.txt   # Info reveal, conversation progression, confirmation ack
│   └── output_format.txt    # 2-4 sentence limit, conversational style
│
├── formats/         # Mode-specific JSON response schemas
│   ├── plain.txt            # Non-training: customer_response + feedback: null
│   ├── training.txt         # Training: customer_response + full feedback block
│   └── stream.txt           # Streaming: plain text, no JSON
│
├── evaluation/      # Evaluation and coaching content
│   ├── coaching.txt         # Full scoring rubric for empathy + active listening
│   └── session_report.txt   # End-of-session report prompt
│
└── openers/         # First message sent to start the scenario
    ├── flight_cancellation.txt
    ├── baggage_delay.txt
    ├── loan_delay.txt
    └── refund_request.txt
```

### Assembly order in `build_system_prompt()`

```python
full_prompt = "\n\n".join([
    system_rules,      # shared/system_rules.txt
    scenario_text,     # scenarios/{scenario}.txt  (PORTAL_DATA block stripped)
    emotion_text,      # emotions/{persona}.txt
    behavior_rules,    # shared/behavior_rules.txt
    output_format,     # shared/output_format.txt
    response_rules,    # formats/{mode}.txt
])
# training mode only:
full_prompt += "\n\n" + coaching_rules   # evaluation/coaching.txt
```

### Separation of concerns — strict rule

| What belongs | Where |
|---|---|
| Customer name, situation facts, case details, resolution criteria | `scenarios/` |
| Emotional intensity, escalation triggers, de-escalation behaviour | `emotions/` |
| Role constraint, character integrity, no-meta-phrases rule | `shared/system_rules.txt` |
| Info reveal rules, conversation progression, email acknowledgment | `shared/behavior_rules.txt` |
| Sentence count, conversational style | `shared/output_format.txt` |
| JSON schema for LLM response | `formats/` |
| Scoring rubric (empathy + active listening) | `evaluation/coaching.txt` |

**Never put emotional descriptors in scenario files.** "Communication Style" in the Identity Layer describes speaking register (direct, methodical, formal), not emotional state (frustrated, anxious, impatient). Emotional state comes entirely from the emotion file.

**Never put scenario facts in emotion files.** `angry.txt` must not reference "refund", "flight", or any other scenario-specific noun.

### Per-turn dynamic injections (not in `build_system_prompt`)

Two things are appended dynamically in `call_llm()` and `stream_llm_response()`, not in the base prompt:

1. **COACHING SIGNAL** — the `nextStep` from the previous feedback turn, injected via `append_coaching_signal()` in `llm_service.py`. This is what nudges the customer to react differently to improved CSR behaviour.
2. **CSR_MESSAGE_TO_EVALUATE** — the literal CSR message the coach must evaluate, appended only in training mode.

---

## Scenario and persona registration

### Adding a new scenario

1. Create `backend/prompts/scenarios/{scenario_key}.txt` — scenario facts only.
2. Create `backend/prompts/openers/{scenario_key}.txt` — the first message sent to start the conversation.
3. Create `backend/workflows/{scenario_key}.json` — portal config (steps, screens, policy, customer data).
4. Add an entry to `SCENARIOS` in `backend/services/llm_service.py`:
   ```python
   SCENARIOS = {
       "your_scenario": {
           "file": "your_scenario",
           "opener": "your_scenario",
       },
   }
   ```
5. Add to `VALID_SCENARIOS` and `SCENARIO_LABELS` in `backend/main.py`.
6. Add to the `DOMAINS` array in `frontend/src/components/ModeSelector.jsx`.
7. Create workflow step components in `frontend/src/components/workflow/{your_scenario}/`.
8. Register the scenario in `frontend/src/components/workflow/utils/screenMaps.js`.

### Adding a new persona/emotion

1. Create `backend/prompts/emotions/{persona_key}.txt`.
   - Include: emotional tone, intensity, communication style, escalation behaviour, de-escalation triggers and behaviour.
   - Do NOT include: scenario facts, customer names, output formatting, sentence count.
2. Add to `VALID_PERSONAS` in `backend/main.py`.
3. Add to `PERSONAS` in `frontend/src/components/ModeSelector.jsx`.

---

## LLM configuration

Provider and model are set via environment variables. The client is built once at import time in `config.py`.

```
LLM_PROVIDER=openai     # or "groq"
MODEL_NAME=gpt-4o
OPENAI_API_KEY=...
GROQ_API_KEY=...
DEBUG_PROMPTS=true      # prints the full assembled system prompt to stdout — use for debugging
```

Both providers use the OpenAI-compatible chat completions interface. Training mode uses `response_format={"type": "json_object"}` to enforce JSON output. Streaming uses `stream=True` with plain text.

---

## Database

**Models** (`backend/models.py`):
- `User` — id, name, username, hashed_password
- `SessionRecord` — user_id, scenario, persona, training (bool)
- `MessageRecord` — session_id, role (user|assistant), content, feedback_json (JSON string, on user turns only)
- `ReportRecord` — session_id, scenario, persona, training, report_json

**Connection** (`backend/database.py`):
- SQLite locally: `sqlite:///./csr_simulator.db`
- PostgreSQL in production: set `DATABASE_URL` env var

---

## Authentication

JWT tokens, 7-day expiry, HS256. Passwords hashed with bcrypt (72-byte limit enforced). `SECRET_KEY` must be set in production — the code has a visible fallback default that must not be used in production.

All chat, session, and report endpoints require a valid `Authorization: Bearer <token>` header.

---

## Frontend architecture

**State management** — all state lives in `App.jsx` via `useState`. No Redux or Zustand. Session data is persisted to `localStorage` to survive page reload.

**View routing** — `App.jsx` switches between views: `landing`, `mode-select`, `profile`, `chat`, `report`. Not React Router.

**ChatWindow** has two panes:
- **WorkflowPortal** (top, resizable via drag handle) — renders the internal portal for the current scenario and step.
- **Chat** (bottom) — the customer conversation. In training mode, a `FeedbackPanel` sidebar shows coaching for the selected turn.

**Two send paths:**
- Training mode → `POST /chat` (JSON, waits for full response + feedback)
- Evaluation mode → `POST /chat-stream` (streaming, `fetch` with `ReadableStream`)

**ModeSelector** is a 4-step wizard: Domain → Scenario → Persona → Mode. Step 2 is skipped if a domain has only one scenario. The config object passed to `onSelect` becomes `sessionConfig` in App.jsx and is forwarded to ChatWindow and the API.

---

## Workflow portal system

Each scenario has a JSON config at `backend/workflows/{scenario}.json`. The portal guides trainees through the correct procedural steps before and after the conversation.

`prompt_metadata.py` extracts a `PORTAL_DATA` JSON block embedded in the scenario prompt file. This block holds customer-specific values (name, booking ref, bag tag, etc.) that are injected into the workflow JSON via `{{key}}` placeholders. This keeps ground-truth facts in one place (the scenario file) and makes them available to both the LLM prompt and the portal UI.

When `load_scenario_prompt()` is called, `strip_portal_data()` removes the `PORTAL_DATA` block from the text before it reaches the LLM, so the LLM only sees the narrative content.

---

## Coaching and evaluation

### Per-turn (training mode)
The LLM evaluates two skills simultaneously with every response:
- **Empathy-First Response** — did the CSR acknowledge emotion before problem-solving?
- **Active Listening** — did the CSR reference specific details from the customer's message?

Each skill is rated `Strong | Developing | Needs Work` with a numeric score (2/1/0). A `nextStep` directive (≤10 words) targets the single highest-priority gap.

`_enforce_feedback_consistency()` in `llm_service.py` is a post-processing safety net that overrides the LLM output for hard rules (e.g., ≤4-word CSR responses must always score Needs Work on both skills).

### End-of-session report
`generate_report()` aggregates turn-level signal ratings into percentages, extracts weak moments, and sends a structured summary to the LLM (using `evaluation/session_report.txt` as the system prompt). The LLM returns a JSON coaching report: `overallPerformance`, `keepDoing`, `keyPatternToImprove`, `actionableImprovement`, `encouragement`.

---

## Known caveats and active areas

- `AVERY_COLLINS_PERSONA` in `ModeSelector.jsx` is a leftover JS object from an earlier architecture. It is not used at runtime — the persona is loaded from `emotions/demanding.txt` on the backend. It can be deleted once confirmed safe.
- `SCENARIO_PERSONA_OVERRIDES` in `ModeSelector.jsx` overrides the persona description text in the UI for `loan_delay + demanding`. This is a display-only override and does not affect prompt assembly.
- Old prompt files (`vc1_prompt.txt`, `vc2_prompt.txt`, etc.) may still exist in `backend/prompts/`. They are no longer loaded by any code and can be deleted.
- `DEBUG_PROMPTS=true` prints the full assembled system prompt on every call, including `start_conversation`. Use this when comparing deployed behaviour against Playground behaviour.

---
> Source: [cmu-teleperformance-research/teleperformance](https://github.com/cmu-teleperformance-research/teleperformance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
