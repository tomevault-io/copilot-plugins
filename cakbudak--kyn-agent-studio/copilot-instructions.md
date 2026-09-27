## kyn-agent-studio

> These rules apply to the whole repository.

# Repository instructions

These rules apply to the whole repository.

## Product boundary

- This is **Kyn.ist Agent Studio**, the standalone OpenAI Build Week cut of Kyn's
  agent-and-automation runtime. The product name must not include “Flight Recorder” or
  present recording as the product's primary purpose.
- It must let a visitor define versioned Actions, Prompts, Skills, Agents, and Flows;
  execute configurable flows; observe authoritative Runs and Steps; pause at human gates;
  and maintain work through approval, diagnosis, bounded repair, and linked reruns.
- The original evidence-bound repair loop remains a real included template. It is one
  example of the platform, never the whole product or the only prescribed journey.
- OpenAI is the model transport. Kyn owns orchestration, action dispatch, authority,
  durable run state, evidence, approval, repair, and replay truth.
- The runtime may use Python's standard library, flat SQLite, and the official OpenAI
  Python SDK. It must not import Kynist, Appiyon, Kynllm, Ainou, PostgreSQL, or copy their
  private implementation or database schemas.
- The public tool surface is a bounded local sandbox. It must not expose shell, filesystem,
  arbitrary network, or production-write authority.
- Never describe fixture, fake-client, or sandbox state as a production integration.

## Data architecture

- SQLite is a flat, product-facing projection. It must not reproduce Parts, Entities,
  Bricks, Frames, graph storage, or another generic ontology.
- Actions, agents, prompts, skills, and flows are explicit resources with immutable
  versions. An Action is a declarative contract over a bounded built-in executor; database
  data can configure and grant an executor but can never register arbitrary server code.
- A Flow is a bounded acyclic graph of Action, Agent, or published Flow nodes with explicit input mappings
  and routes. Every run pins one immutable flow version and therefore all transitive Action,
  Agent, Prompt, and Skill versions. A repair creates a successor version; it never edits
  history.
- Runs, Steps, ordered events, model-call summaries, action receipts, approval requests and
  decisions, diagnoses, repairs, and sandbox effects are durable rows.
- Events and resource-version rows are append-only. Terminal run states are absorbing.
- A hash-linked event ledger is authoritative for the standalone cut; best-effort telemetry
  is not evidence.

## Runtime architecture

- `serve.py` is the composition root for the static application and `/api/v1` HTTP API.
- `backend.service.ControlPlane` is the only product mutation path. HTTP handlers stay thin.
- No SQLite write transaction may remain open during OpenAI or other external I/O.
- There is one Action invocation path for direct Flow nodes and model-requested Actions.
  Every invocation validates the pinned input and output schemas and records success or
  failure with run/node/attempt correlation.
- OpenAI Responses function calls are schema-validated and intersected with the exact tools
  or Actions granted by the run's pinned Skills. Model prose never grants authority.
- Run terminal states are absorbing. Approval is a non-terminal pause; retry/rerun creates
  explicitly linked work and never silently resumes against a newer definition.
- Diagnosis may cite only events from its run. Repair may touch only flow-version paths
  allowed by the pinned repair policy.
- Repair application requires proposal hash, expected flow revision, explicit acknowledgement,
  actor, and reason. The write is revision-fenced and idempotent.
- Deterministic fake model clients are test seams only. A release claim of live agentics
  requires a sanitized real-model proof.

## Security and quality

- The visitor owns the OpenAI API key. The browser keeps it in `sessionStorage` for the
  current tab and sends it only on same-origin model-action requests. The server must not
  load an operator `OPENAI_API_KEY`, persist a visitor key, include it in logs/events, or
  return it. The official SDK client is constructed per bounded model action.
- Mutations require a same-origin request and an isolated workspace cookie. Enforce body,
  model-turn, tool-call, per-workspace, per-address, and global cost bounds.
- Validate JSON strictly, render dynamic values with safe DOM APIs, redact secret-like data,
  and preserve restrictive response headers.
- WCAG 2.2 AA is the target. Keyboard, focus, reduced motion, narrow viewport, loading,
  empty, error, stale-revision, replay, and reset/new-workspace paths are required.
- Verification must include positive, negative, boundary, concurrency/fencing, mutation,
  real-model, and browser journeys. Green UI tests alone do not prove runtime truth.

## Git mandate: forward only

Do not reset, revert, stash, rebase, amend, squash, switch branches, or rewrite history.
Work on the active branch. At each stable state, stage every change with `git add -A` and
create a new commit.

---
> Source: [cakbudak/kyn-agent-studio](https://github.com/cakbudak/kyn-agent-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
