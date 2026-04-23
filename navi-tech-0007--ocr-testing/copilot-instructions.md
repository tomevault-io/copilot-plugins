## ocr-testing

> You are modifying a critical OCR pipeline for Squad Space (Free Fire tournament platform).


Context:
You are modifying a critical OCR pipeline for Squad Space (Free Fire tournament platform).
This service must achieve 99.9%+ correctness for team-card detection and must never silently break.

1. Mindset & Scope

Treat every change as production-critical.

You are not allowed to “simplify” requirements, skip validation, or assume inputs are always clean.

Stay strictly within the requested scope.
Do not refactor unrelated modules, rename generic utilities, or touch other features unless explicitly asked.

2. No Documents, Only Code + Minimal Comments

Do NOT generate separate documentation files (no README, no long Markdown design docs, no architecture essays).

It’s OK to add:

Inline comments where logic is subtle.

Very small docstrings for critical functions.

Focus output on: code, types, tests, and wiring to frontend, nothing else.

3. Input / Output Contracts

For every function, handler, or service you write:

Define a clear TypeScript/Pydantic schema for inputs and outputs.

Validate incoming data at the boundary:

Image type, size, dimensions.

JSON shape from LLM.

If something is invalid:

Fail loudly and clearly (structured error), never silently ignore.

Don’t “guess” or auto-correct LLM output beyond what we explicitly described.

4. LLM Interaction Rules (Vision Model)

Always use the exact system + user prompts provided.
Do not rewrite or “simplify” them.

Always:

Set deterministic-ish parameters: temperature = 0, top_p = 1.

Enforce strict JSON parsing:

Parse the response.

If parsing fails → retry once with the retry prompt.

Implement a small helper like safeParseClaudeJson(responseText) to centralize parsing + error handling.

5. Data Validation & Safety Nets

For Step-1 (card detection) and later steps:

Validate:

cards array exists.

Each card has all required fields (card_index, bounds, etc.).

bounds are integers and within image dimensions.

If any card is invalid:

Mark it as invalid or drop it from the result (depending on instructions).

Never let a malformed card propagate to later steps unmarked.

Add defensive programming:

Type guards.

Fallbacks for null/undefined.

Assertions in critical branches.

6. Testing & Frontend Integration

Every change must be testable end-to-end from the frontend.

Create or extend a dedicated dev page in the frontend, e.g.:

/dev/ocr-team-cards

This page must let us:

Upload a real screenshot.

Call POST /ocr/cards/detect.

Visually overlay the detected bounding boxes on top of the image (simple canvas or absolutely positioned divs).

Testing expectations:

Use realistic sample screenshots (not just mocks).

Verify:

Correct number of cards found.

Boxes align visually with team cards.

If possible, add at least:

One backend unit test for the Claude response parser.

One integration test hitting the endpoint with a mocked Claude response.

7. Logging & Observability

Add structured logs for:

Incoming request (size, image dimensions).

Claude call (latency, model name).

JSON parse errors.

Number of cards detected and returned.

Never log:

Full image bytes.

Sensitive tokens / secrets.

Logs should help us debug mismatched or missing cards in production.

8. Config, Secrets, and Feature Flag

All model keys, endpoints, and config must come from environment variables or a config module.

Add a feature flag or config toggle like:

OCR_TEAM_CARD_DETECTION_ENABLED

If the flag is off:

Endpoint should return a clear “disabled in this environment” error.

9. Error Behaviour

On hard failures (e.g., Claude down, JSON repeated invalid, etc.):

Return a well-structured 5xx or 422 JSON error with:

error_code

message

Do not return partial garbage or half-parsed data.

Never swallow exceptions silently; always:

Log them.

Convert to a controlled error response.

10. Code Quality & Consistency

Respect the existing stack:

FastAPI + Python 3.11+ on backend.

Next.js + TypeScript on frontend.

Follow existing lint/formatter rules (black, isort, eslint, prettier, etc.).

Keep functions small and composable:

One job per function: HTTP handler → service → Claude client → parser → validator.

Prefer boring, robust patterns over clever tricks.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Navi-tech-0007) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
