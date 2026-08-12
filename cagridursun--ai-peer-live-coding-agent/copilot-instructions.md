## live-coding-interview-spring-fw

> Live coding interview rules (Java + Spring Boot). Question-agnostic. Force simplicity, explainability, and strong tests. AI is a copilot, not autopilot.


# Live Coding Interview Rules (Java + Spring Boot) — General + Simple

## North Star
Deliver a **working, locally runnable** web service with a **small, high-quality test suite** in a **2-hour** live coding interview.
Success = predictable behavior + clear structure + explainable decisions.

## Hard Simplicity Constraints (Non-Negotiable)
- Keep architecture to **3 layers max**:
  - Controller (HTTP) → Service (business rules) → Repository/Store (in-memory)
- Avoid extra patterns unless asked:
  - ❌ hexagonal/ports-adapters
  - ❌ DDD aggregates/repositories/specifications
  - ❌ factories/builders for everything
  - ❌ excessive interfaces
  - ❌ reflection / metaprogramming
  - ❌ complex generics
- Default persistence: **in-memory**.
- Default concurrency: assume requests can be concurrent → use safe data structures if stateful.
- Keep file count low and names obvious.

## First 10 Minutes (How to start)
1. **Restate** the problem in 3–5 sentences.
2. Ask **2–5 clarifying questions** only if needed.
3. List:
   - key entities
   - endpoints
   - edge cases
   - explicit non-goals
4. Create a **short README** (no code) and then **start coding**.

Time cap: **README must be ≤ 10 minutes**.

## Delivery Order (Bias to Working Software)
1. Boot app and health-check run
2. Implement **one endpoint end-to-end** (happy path) + tests
3. Implement the second endpoint + tests
4. Add validation + consistent errors + tests
5. Harden edge cases + minimal refactor
6. Optional extras only if asked (idempotency, filtering, pagination, etc.)

If stuck: prioritize a minimal working slice and test it.

## API Behavior Rules
- Return consistent JSON.
- Use standard status codes:
  - 200 OK (GET success)
  - 201 Created (POST create)
  - 400 Bad Request (validation/format)
  - 404 Not Found (missing resource)
  - 409 Conflict (invalid state / duplicate / unexpected step)
- Provide a consistent error response:
  - `code` (string)
  - `message` (human-readable)
  - `details` (optional list, e.g. field errors)
- Use `@RestControllerAdvice` to centralize error mapping.

## Validation Rules
- Use Jakarta Bean Validation (`@Valid`) for request DTOs.
- Validate:
  - required fields
  - allowed values (enums/sets)
  - numeric ranges if applicable
- Prefer returning field-level errors in `details`.

## State & Concurrency Rules (When applicable)
If the problem involves sessions, workflows, or mutable state:
- Treat as concurrent by default.
- Use `ConcurrentHashMap` and atomic operations (`compute`, `computeIfAbsent`) where helpful.
- Enforce state rules explicitly:
  - answering after completion -> 409
  - invalid transition -> 409
  - unexpected question -> 409

## Testing Rules (Must be strong but minimal)
Tests are a core deliverable. Prefer fewer, higher-signal tests.

Minimum expected:
- **HTTP integration tests** with MockMvc:
  - happy path for each endpoint
  - 400 validation error
  - 404 not found
  - 409 conflict scenario (if there is state/step logic)
- **Service unit tests** for critical business rules / branching logic.

Testing principles:
- Deterministic (no sleeps, no time-sensitive flakiness)
- Readable names and clear assertions
- Assert behavior, not implementation details

## AI Usage Rules (Critical)
- Do not generate code I cannot explain line-by-line.
- Use AI for:
  - scaffolding
  - DTOs + validation annotations
  - test skeletons
- After AI output, do an **Ownership Pass**:
  - simplify names and structure
  - remove unused abstractions
  - ensure status codes and errors are correct
  - add at least **one** meaningful test improvement manually

Never say “Cursor generated it” as justification. Always justify with tradeoffs.

## Communication / Defense Checklist (What to say out loud)
When making a choice, use:
- “I’m choosing X because…”
- “I’m intentionally not doing Y because time / scope / non-goal…”
- “If this were production, next step would be…”

## Definition of Done (Before you move on)
For each requirement:
- endpoint works via curl
- tests cover happy + error
- consistent errors and status codes
- code is easy to explain
- no extra features added without request

## Structured TODO Mode (For Interview Control)
This mode should only be activated after:
- The problem has been restated
- Requirements clarified
- README is finalized

When starting implementation after the README phase:

1. Create a `/todo` directory at project root.
2. Break down the implementation into **small, interview-scoped tasks**.
3. Limit to **maximum 8 TODO items**.
4. Each TODO must represent a meaningful, incremental slice of working software.

Naming format:
- `01-<short-description>.md`
- `02-<short-description>.md`
- etc.

Each TODO file must contain:

# TODO <number>: <Title>

## Status: TODO | IN_PROGRESS | DONE

## Description
Short explanation of the goal of this task.

## Acceptance Criteria
Clear bullet list describing what must be true for this task to be DONE.

## Notes
Optional section for important implementation considerations.

## Curl requests
Add a sample curl request if an endpoint is implemented. Curl request must be in `curl -s -X "POST|GET|PUT|etc." {endpoint url} + {request body - if it is require} ` as well as `Invoke-RestMethod -Method  "POST|GET|PUT|etc." -Uri ...` for windows command line. 

Rules:
- Keep each TODO small enough to complete in 10–20 minutes.
- Do not include infra, Docker, CI/CD, or production hardening unless explicitly requested.
- Do not pre-create all implementation code at once.
- After generating the TODO list, STOP and wait for confirmation before implementing.
- Implement exactly one TODO at a time.
- After completing a TODO:
  - Update its Status to DONE.
  - Briefly summarize what changed.
  - Wait for next instruction.

Definition of Good TODO Breakdown:
- Scaffold minimal project
- Implement first endpoint happy path
- Add tests for first endpoint
- Implement second endpoint
- Add validation + error handling
- Add edge-case tests
- Final cleanup/refactor

Avoid:
- Over-granular tasks (e.g., "create DTO class")
- Over-ambitious tasks (e.g., "design extensible rule engine framework")

The TODO system exists to enforce incremental delivery, not to simulate a Jira backlog.

---
> Source: [cagridursun/ai-peer-live-coding-agent](https://github.com/cagridursun/ai-peer-live-coding-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
