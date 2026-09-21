## ocr

> This file governs how work happens on this repository, for humans and AI

# AGENTS.md — Project Constitution

This file governs how work happens on this repository, for humans and AI
agents alike. Read this before making changes.

## Architecture rules

- Modular monolith. Do not split into microservices without a demonstrated
  scaling or isolation requirement recorded in DECISIONS.md.
- PaddleOCR is the OCR engine. Do not substitute it without an explicit
  decision recorded in DECISIONS.md.
- No database. No Redis/Celery/Kafka/queueing infrastructure. If a future
  requirement seems to need one, stop and propose the change before adding
  it — don't add it because "production systems usually have one."
- OCR inference is CPU/GPU-bound and synchronous. It must never run
  directly inside an `async def` request handler. It is dispatched to a
  bounded worker pool (see app/ocr/engine.py).

## Coding rules

- Type hints required everywhere (`mypy --strict` passes).
- `ruff` for lint + format — no additional overlapping linters/formatters.
- Small, cohesive modules. No generic repositories/factories, no giant
  utility modules, no global mutable state beyond the documented
  worker-process model globals in app/ocr/engine.py.
- No premature abstraction. Add an interface when a second implementation
  actually exists, not before.

## Security / privacy rules

- Data flow is: request → process in memory → respond → discard. No
  persistence of uploaded images, extracted names, DOBs, Aadhaar numbers,
  or raw OCR text, unless a future requirement is explicitly decided and
  recorded in DECISIONS.md.
- Aadhaar data handling has a legal dimension (Aadhaar Act 2016 / UIDAI
  regulations), not just a technical one. Confirm authorization/compliance
  posture before this service is used for anything beyond development —
  this repository does not make that determination for you.
- Logging: only fields in `ALLOWED_FIELDS` in app/core/logging.py may be
  logged. Never name, DOB, Aadhaar number, raw OCR text, or image bytes.
  `log_event` raises on unknown fields by design — do not work around it.
- Error messages returned to API clients must never contain file paths,
  stack traces, or internal/model details.

## Testing requirements

- Every new module needs unit tests for its logic.
- Every new endpoint needs an integration test covering at least one
  success path and one failure path.
- Never commit real Aadhaar images or any real person's data. Fixtures
  are synthetic only (see tests/fixtures/.gitkeep and .gitignore).
- Integration tests use a fake OCR engine (tests/conftest.py) — they do
  not load real PaddleOCR models. Real-model correctness testing is a
  separate, explicitly scoped task (T010/T011), not implicit in "tests
  pass."

## Dependency rules

Before adding a dependency, answer in the PR/commit description:
1. What problem does it solve?
2. Why can't the current stack solve it?
3. What is its runtime cost?
4. Does it increase operational complexity?
5. Is it actively maintained?
6. Is it required for the current task, or speculative?

## Performance rules

- Model load happens once per worker process, never once per request.
- No performance numbers get stated without a benchmark backing them
  (T011). Comments describing expected behavior must say so explicitly
  and not be presented as measured fact.

## Change-control rules

- Work proceeds as atomic tasks (see TASKS.md), each reviewed before the
  next begins.
- No commits without explicit approval. Before proposing a commit, show:
  changed files, summary, test results, diff impact, commit message.
- Architectural decisions are recorded in DECISIONS.md, not made silently
  inside code comments alone.

---
> Source: [khushnam-chauhan/OCR](https://github.com/khushnam-chauhan/OCR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
