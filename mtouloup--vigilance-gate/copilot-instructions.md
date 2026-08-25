## vigilance-gate

> > **This file is the persistent memory and operating manual for this repository.**

# CLAUDE.md — VIGILANCE T5.3 Agentic Wrapper Framework

> **This file is the persistent memory and operating manual for this repository.**
> Update it whenever architecture changes, schemas evolve, or milestone status shifts.
> Last updated: July 2026 — reflects actual implemented state of the repository. Schemas frozen with T5.4 (GFT); C4 verb catalogue documented; M6 closed; T5.5 interface question resolved at July 1 KOM (T5.5 is blueprint/scenario collection, not policy enforcement — downstream consumer of `t53.policy_updates` now open).

---

## Project Identity

**Project:** VIGILANCE
**EU Grant:** Horizon Europe — GAP-101249737
**Duration:** 36 months
**INNOV role:** Core technical contributor and task lead for T5.3

**Work Package:** WP5 — Agentic AI Cybersecurity Platform
**Task:** T5.3 — Agentic Wrappers for Cybersecurity Technologies
**Task Lead:** INNOV-ACTS

**One-line purpose:** T5.3 is the operational execution bridge between WP5 AI intelligence (agents, orchestration, knowledge) and the real cybersecurity tools deployed in the pilot environments. It normalises raw events into a canonical format, routes them through safety checks, executes AI-approved actions via vendor-specific adapters, and records every step for audit.

**Primary deliverable contribution:** D5.1 (Framework Architecture and Data Models)

### Pilots in scope for INNOV

| Pilot | Task | Organisation | Country | Sector |
|---|---|---|---|---|
| Pilot #1 | T6.3 | OTE | Greece | Telecom SOC |
| Pilot #4 | T6.5 | Siemens | Romania | Industry 4.0 / Manufacturing |

> ⚠️ **HARD CONSTRAINT:** Port of Rotterdam (Pilot #2 / T6.4, Netherlands) and CaixaBank (Pilot #3 / T6.6, Spain) are **not** in scope for INNOV. Never reference these pilots in INNOV-produced documents, diagrams, or code. The T5.3 framework defines sector profiles for all four GA pilots because the GA mandates transferable wrappers — but INNOV validates only against OTE and Siemens.

---

## Implementation Status

**The repository contains a fully implemented, containerised, running service.** This is not a design-phase repository. All core components are implemented and tested.

```
vigilance-GATE/
│
├── CLAUDE.md                          ← this file (persistent memory)
├── BLUEPRINT.md                       ← architectural blueprint (design rationale, GA mandate mapping)
├── Dockerfile                         ← python:3.11-slim image
├── docker-compose.yml                 ← full stack: gate + rabbitmq + ollama + dozzle
├── pyproject.toml                     ← package manifest and dependencies
│
├── data/                              ← generated output (mounted from container /app/data)
│   └── workflow_audit.csv             ← one row per pipeline execution (raw → canonical → result)
│
├── vigilance/                         ← main application package
│   ├── main.py                        ← entrypoint (REST API + broker consumer)
│   ├── service.py                     ← service lifecycle / broker consumer
│   ├── pipeline.py                    ← T53Pipeline — INTEGRATED mode orchestration
│   ├── workflow_logger.py             ← WorkflowCSVLogger — per-execution audit CSV
│   ├── api/                           ← REST API (FastAPI, port 8000)
│   │   └── app.py                     ← POST /api/v1/events, POST /api/v1/action-requests, GET /api/v1/health, GET /api/v1/profiles
│   ├── broker/                        ← RabbitMQ broker (pika); InMemoryBroker for tests
│   ├── llm/                           ← LLM abstraction layer
│   │   ├── base.py                    ← LLMProvider ABC + StubLLMProvider
│   │   └── ollama_provider.py         ← OllamaLLMProvider (Mistral 7B + Nemo 12B)
│   ├── models/                        ← Pydantic v2 data models (frozen schema)
│   │   ├── canonical_event.py
│   │   ├── action_request.py
│   │   ├── execution_result.py
│   │   ├── guardrail_check.py
│   │   └── audit_record.py
│   └── components/
│       ├── c1_ingestion/              ← C1: Normalizer + 5 parsers (CEF, ECS, OT JSON, Syslog, LLM)
│       ├── c3_execution/              ← C3: ActionExecutor + PolicyTranslator (NL→Rego, implemented)
│       ├── c4_adapters/               ← C4: ToolAdapter ABC + 12 plugins across 4 sectors
│       │   ├── telecom/               ← ote_siem, ote_iam, ote_ids
│       │   ├── industry4/             ← scada_opcua, ot_iam, industrial_siem
│       │   ├── maritime/              ← port_siem, port_iam, port_ops
│       │   └── finance/               ← bank_siem, bank_iam, fraud_engine
│       ├── c5_safety/                 ← C5: SafetyGate + AuditLog
│       └── c6_profiles/               ← C6: ProfileManager + SectorProfile dataclass
│
├── profiles/                          ← sector profile YAMLs
│   ├── telecom.yaml                   ← OTE / TELECOM (confidence_threshold: 0.80)
│   ├── industry4.yaml                 ← Siemens / INDUSTRY_4 (ot_safety_flag: true)
│   ├── maritime.yaml                  ← Port of Rotterdam / MARITIME (GA transferability)
│   └── finance.yaml                   ← CaixaBank / FINANCE (confidence_threshold: 0.85)
│
├── schemas/                           ← data model and broker schemas
│   ├── README.md
│   ├── models/                        ← JSON Schema (draft 2020-12), auto-generated from Pydantic
│   ├── broker/
│   │   └── topics.yaml                ← broker integration interface (YAML, with comments)
│   └── profiles/
│       └── sector_profile.schema.yaml ← sector profile schema (YAML, with comments)
│
├── infra/
│   └── rabbitmq/
│       ├── rabbitmq.conf              ← loads definitions at startup
│       └── definitions.json           ← pre-declares all durable queues + user
│
├── tests/                             ← test suite (115 tests across 10 files)
│   └── scenarios/                     ← end-to-end scenario tests (A–D, all four pilots)
└── tools/
    ├── publish_event.sh               ← example producer for pilot partners
    └── simulate_t54.sh                ← T5.4 orchestrator simulator (closes INTEGRATED test loop)
```

---

## Architecture — Current Implemented Design

### Mode of operation

**T5.3 operates exclusively in INTEGRATED mode.** STANDALONE and DIGITAL_TWIN modes were removed (PR #28, May 2026). The in-process C2 AgentLoop was also removed at the same time — reasoning is owned by T5.4 (GFT) and T5.2 (AEGIS agent repository), not T5.3.

### Active components (5, not 6)

| ID | Name | Status | LLM? |
|---|---|---|---|
| C1 | Event Ingestion & Normalization | ✅ Implemented | Conditional — Mistral 7B fallback for unknown formats |
| C3 | Action & Policy Execution | ✅ Implemented | Conditional — Mistral Nemo 12B for NL→Rego translation |
| C4 | Tool Adapter Layer | ✅ Implemented | No — deterministic API translation |
| C5 | Safety, Audit & Simulation | ✅ Implemented | Conditional — Mistral 7B semantic check for ESCALATE verdicts |
| C6 | Sector Profile Manager | ✅ Implemented | Indirect — sets per-sector context at startup |
| ~~C2~~ | ~~Agentic Interaction Layer~~ | ❌ Removed | Reasoning is T5.4 + T5.2 domain |

### INTEGRATED mode data flow

```
Pilot Tool (CEF / ECS / OT JSON / syslog / free-text alert)
  │
  ▼
RabbitMQ  [topic: pilot.events.raw]
  │
  ▼
C1 — Event Ingestion & Normalization
  │  Parser priority: CEF → ECS → OT JSON → Syslog → LLM fallback
  │  C6 injects sector-specific enrichments (e.g. ot_safety_flag for INDUSTRY_4)
  │  pilot=UNKNOWN falls back to TELECOM profile with a warning
  │  → CanonicalEvent (UUID event_id always generated by C1, never extracted from payload)
  │  → NormalizationMeta (parser_used, llm_invoked, llm_fields) recorded for CSV audit
  ▼
RabbitMQ  [topic: t53.canonical_events]   ← T5.4 consumes
  │
  │          T5.4 (GFT): T5.1 RAG context → T5.2 agent selection → ActionRequest
  │
  ▼
RabbitMQ  [topic: t53.action_requests]    ← T5.3 consumes
  │
  ▼
C5 — Safety Gate (pre-execution)
  │  Five deterministic checks:
  │    ① agent_confidence ≥ 0.80
  │    ② src_ip not in protected ranges
  │    ③ len(actions) ≤ 5 (proportionality)
  │    ④ OT: isolate_plc requires mode="safe-state"
  │    ⑤ OT: ZTA scope must be zone-limited
  │  LLM semantic guardrail (Mistral 7B) for ESCALATE cases
  │  → GuardrailCheck {verdict: APPROVED | REJECTED | ESCALATE}
  ▼
C3 — Action & Policy Execution
  │  NL→Rego translation if policy_update present (Mistral Nemo 12B)
  │  Dispatches to C4 adapters
  ▼
C3 — Policy Translation (if policy_update present)
  │  Mistral Nemo 12B + few-shot examples → OPA/Rego rule
  │  OPA parse validation + single retry on failure
  │  Falls back to "default deny = true" on double failure (fail-closed)
  │  → published to t53.policy_updates
  ▼
  ├─→ RabbitMQ [topic: t53.policy_updates]    → downstream consumer TBD (see Open Items)
  └─→ RabbitMQ [topic: t53.actions.dispatch]  → Pilot tools (fire-and-forget)

C5 — Audit Closure
  │  AuditLog.close_record() called after dispatch
  │  WorkflowCSVLogger appends one row to workflow_audit.csv
  ▼
RabbitMQ  [topic: t53.results]   ← T5.4, T5.2 consume
```

**Key properties of this design:**
- T5.3 returns 202 Accepted immediately after dispatching — never blocks on downstream policy consumer or pilot tool response.
- C1 and ActionRequest consumers run on independent threads (PR #22) to prevent LLM blocking.
- RabbitMQ heartbeat is disabled (heartbeat=0) to prevent connection reset during LLM calls (PR #18).
- UNKNOWN pilot falls back to TELECOM profile with a warning log; never hard-fails on unknown sector.
- Production dispatch (`pipeline._dispatch()`) publishes fire-and-forget to the broker — it does **not** call per-verb C4 adapter routing at runtime. The `ActionExecutor` class (c3_execution/executor.py) implements per-verb routing and is used in tests and direct in-process calls.
- `execute_action_request()` returns `tuple[ExecutionResult, str | None]` — the second element is the generated Rego rule string when `policy_update` was present, otherwise `None`. The REST API unwraps this tuple and includes `policy_translation` in the response body.
- `data/workflow_audit.csv` is thread-safe (file lock per write) and captures the complete C1 → C5 → C3 → C4 telemetry per event.

### Multi-pilot runtime

One `vigilance-gate` container handles all four sectors simultaneously. Pilot/sector detection happens per-event in C1 (parser heuristics + LLM extraction). The correct C6 profile and C4 adapter set are selected per event. `ProfileManager.load_all_profiles()` loads all four profiles at startup.

---

## Broker Topics

| Topic | Direction | Producer | Consumer |
|---|---|---|---|
| `pilot.events.raw` | Inbound | Pilot tools | C1 |
| `t53.canonical_events` | Outbound | C1 | T5.4 |
| `t53.action_requests` | Inbound | T5.4 | C5 → C3 → C4 |
| `t53.policy_updates` | Outbound | C3 | downstream consumer TBD (see Open Items) |
| `t53.actions.dispatch` | Outbound | C4 | Pilot tools (fire-and-forget) |
| `t53.results` | Outbound | C5 | T5.4, T5.2 |

All queues are durable and pre-declared via `infra/rabbitmq/definitions.json` at broker startup.

---

## LLM Usage

| Component | Model | Purpose | Frequency |
|---|---|---|---|
| C1 | Mistral 7B | Field extraction from unknown/novel log formats | Low — fallback only when no deterministic parser matches |
| C3 | Mistral Nemo 12B | NL → OPA/Rego policy rule translation | Low — only when `policy_update` field is set |
| C5 | Mistral 7B | Semantic guardrail second-opinion for ESCALATE verdicts | Low — borderline cases only |

**C2 (reasoning loop) has been removed.** Reasoning is T5.4's responsibility.

**Design rule:** LLMs never call real tools directly. Tool calls are intercepted by T5.3, executed via C4. The LLM operates on canonical representations only.

**Zero LLM calls on the happy path** — known log format, confidence above threshold, no policy update.

---

## Data Models

> **Contract status:** All schemas below are **frozen** — agreed with T5.4 (GFT) in July 2026. The canonical JSON Schema (draft 2020-12) definitions live under `schemas/models/` and are auto-generated from the Pydantic models in `vigilance/models/`. Do not modify field names, types, or optionality without a formal cross-task change.

### CanonicalEvent

Produced by C1. Consumed by T5.4 (via `t53.canonical_events`).

```json
{
  "event_id":       "string   — UUID, always generated by T5.3 (never extracted from raw payload)",
  "type":           "string   — event type identifier (e.g. BRUTE_FORCE_ATTEMPT, LATERAL_MOVEMENT)",
  "pilot":          "string   — sector: TELECOM | INDUSTRY_4 | MARITIME | FINANCE",
  "severity":       "string   — LOW | MEDIUM | HIGH | CRITICAL",
  "timestamp":      "string   — ISO 8601 UTC (required)",

  "src_ip":         "?string  — IPv4/IPv6, nullable",
  "target":         "?string  — host/user/resource, nullable",
  "count":          "?int     — occurrence count, nullable (coerced; never a raw LLM string)",
  "nodes_affected": "?int     — nullable",

  "subscriber_id":  "?string  — TELECOM, nullable",
  "cell_id":        "?string  — TELECOM, nullable",
  "imsi":           "?string  — TELECOM, nullable",

  "plc_id":         "?string  — INDUSTRY_4, nullable",
  "line_id":        "?string  — INDUSTRY_4, nullable",
  "scada_zone":     "?string  — INDUSTRY_4, nullable",
  "ot_protocol":    "?string  — INDUSTRY_4, nullable",
  "ot_safety_flag": "boolean  — defaults to false; true when the event touches OT safety-critical scope",

  "raw_payload":    "object   — original vendor log content, verbatim (defaults to {})"
}
```

**Required:** `event_id`, `type`, `pilot`, `severity`, `timestamp`. All other fields are optional / nullable.

**Design notes:**
- Sector-specific fields are **flat** on the top-level object (no nested `sector_extensions` container). Maritime and finance fields are not represented in the frozen schema — those pilots are supported through the `pilot` value plus `raw_payload` today.
- `event_id` is always a UUID generated by C1 — never extracted from log content (PR #29).
- LLM-extracted numeric fields are coerced via `_to_int()` / `_to_float()` helpers (PR #19) to prevent Pydantic validation errors from malformed LLM output.
- Cross-pilot fields (e.g. `plc_id` populated on a TELECOM event) are never emitted by the LLM (PR #24).

---

### ActionRequest

Produced by T5.4 (GFT). Consumed by T5.3 / C5. Published on `t53.action_requests`.

```json
{
  "request_id":       "string   — unique ID (T5.4-owned, typically UUID v4)",
  "event_id":         "string   — echoes the triggering CanonicalEvent.event_id",
  "pilot":            "string   — echoes CanonicalEvent.pilot",
  "actions":          ["string"],  // ordered list of verb tokens (see C4 Adapter Vocabulary)
  "policy_update":    "?string  — natural-language policy change description, nullable",
  "agent_confidence": "float    — 0.0–1.0; C5 requires ≥ 0.80 to proceed"
}
```

**Required:** `request_id`, `event_id`, `pilot`, `actions`, `agent_confidence`. `policy_update` is nullable and defaults to null.

**Convention inside `actions`:**
- Each string is a plain **verb token** (e.g. `"block_ip"`, `"isolate_plc"`) — not a `verb:target` pair, not a nested object.
- Targets are resolved from the CanonicalEvent context (looked up via `event_id`), not embedded in the action string.
- C3 dispatches each verb to the first C4 adapter whose `supported_actions` contains the exact string.
- Unknown verbs produce a per-action `ActionResult` with `success=false, response_code=404`; the request as a whole still completes.
- Actions execute in listed order.
- The full authoritative vocabulary per pilot is documented under **C4 Adapter Vocabulary** below.

**Convention for `policy_update`:**
- Short, directive natural-language sentence describing the desired ZTA policy change.
- C3 compiles the NL to OPA/Rego via Mistral Nemo 12B and publishes the result on `t53.policy_updates`. The downstream consumer of the compiled policy is currently open — T5.5 was previously assumed but ruled out at the July 1 KOM (T5.5 is scoped around blueprint and scenario collection, not policy enforcement).
- Example tested against the current stack: `"Deny all OPC-UA traffic from Zone-B to Zone-A for 4 hours"`.
- Distinct from the `update_zt_policy` action verb — see the C4 Adapter Vocabulary section.

---

### AgentDecision

**T5.4-internal reasoning artefact.** Not consumed by T5.3. Documented here for cross-task awareness only.

```json
{
  "decision_id":         "string",
  "event_id":            "string",
  "threat_type":         "string",
  "recommended_actions": ["string"],
  "confidence":          "float",
  "reasoning_turns":     "int",
  "pilot":               "string"
}
```

T5.4 reduces an `AgentDecision` to an `ActionRequest` before publishing to `t53.action_requests`. T5.3 sees only the resulting ActionRequest.

---

### GuardrailCheck

Produced by C5 pre-execution. Internal only — not published to the broker; used as the gating record before C3 executes.

```json
{
  "check_id":         "string",
  "request_id":       "string  — echoes the ActionRequest.request_id being gated",
  "verdict":          "string  — enum: APPROVED | REJECTED | ESCALATE",
  "reasons":          ["string"],
  "ot_safety_checked": "boolean — defaults to false; true when OT-specific checks (④, ⑤) ran"
}
```

The five deterministic gates ① confidence, ② protected-IP allowlist, ③ proportionality, ④ `isolate_plc` safe-state, ⑤ OT ZTA zone scope. `ESCALATE` triggers a Mistral 7B semantic second-opinion; the LLM output resolves to `APPROVED` when proportionate or `REJECTED` otherwise.

---

### ActionResult

Per-action outcome produced by C4 adapters and rolled up into ExecutionResult.

```json
{
  "action":        "string   — echoes the verb token dispatched",
  "plugin":        "string   — C4 adapter plugin_name (e.g. ote_siem, scada_opcua)",
  "success":       "boolean",
  "latency_ms":    "int",
  "response_code": "?int     — HTTP-style code from the underlying tool call, nullable",
  "message":       "string   — human-readable adapter response (defaults to \"\")"
}
```

---

### ExecutionResult

Produced by C5 after audit closure. Published on `t53.results`. Consumed by T5.4 and T5.2.

```json
{
  "request_id":      "string   — echoes ActionRequest.request_id",
  "event_id":        "string   — echoes CanonicalEvent.event_id",
  "pilot":           "string",
  "action_results":  [ActionResult],
  "overall_success": "boolean  — true only if every ActionResult.success is true",
  "timestamp":       "string   — ISO 8601 UTC"
}
```

---

### AuditRecord

Internal T5.3 audit trail, persisted per request. Backs the `workflow_audit.csv` rows and any future audit REST endpoint. Not published to the broker.

```json
{
  "audit_id":         "string",
  "pilot_id":         "string",
  "event_id":         "string",
  "request_id":       "string",
  "timestamp_opened": "string   — ISO 8601 UTC",
  "timestamp_closed": "?string  — ISO 8601 UTC, nullable until closure",
  "verdict":          "string   — C5 verdict recorded at gate time",
  "action_results":   [{}],     // list of ActionResult-shaped objects
  "latencies_ms":     [int],
  "closed":           "boolean  — defaults to false"
}
```

---

### AuditRecord

Internal T5.3 audit trail, persisted per request. Backs the REST audit endpoint (planned M7–M9). Not published to the broker.

```json
{
  "audit_id":         "string",
  "pilot_id":         "string",
  "event_id":         "string",
  "request_id":       "string",
  "timestamp_opened": "string   — ISO 8601 UTC",
  "timestamp_closed": "?string  — ISO 8601 UTC, nullable until closure",
  "verdict":          "string   — C5 verdict recorded at gate time",
  "action_results":   [{}],     // list of ActionResult-shaped objects
  "latencies_ms":     [int],
  "closed":           "boolean  — defaults to false"
}
```

---

## WorkflowCSVLogger

`vigilance/workflow_logger.py` — cross-cutting audit component. Appends one row to `data/workflow_audit.csv` after every completed pipeline execution. Thread-safe (file lock per write). Path controlled by `WORKFLOW_CSV_PATH` env var.

### Columns captured per row

| Column | Description |
|---|---|
| `timestamp` | ISO 8601 UTC at row write time |
| `event_id` | CanonicalEvent UUID |
| `pilot` | Detected sector |
| `severity` | Normalized severity |
| `raw_event` | Original input (JSON-encoded) |
| `parser_used` | CEF \| ECS \| OT_JSON \| Syslog \| LLM |
| `c1_llm_invoked` | True when LLM fallback ran in C1 |
| `c1_llm_fields` | Fields extracted by LLM (JSON, or empty) |
| `canonical_event` | Full CanonicalEvent JSON |
| `request_id` | ActionRequest UUID from T5.4 |
| `actions_requested` | Pipe-separated action verb list |
| `agent_confidence` | Confidence from T5.4 |
| `guardrail_verdict` | APPROVED \| REJECTED \| ESCALATE |
| `guardrail_reasons` | Pipe-separated reason strings |
| `c5_llm_invoked` | True when Mistral 7B semantic check ran |
| `c5_llm_response` | Raw LLM JSON response (or empty) |
| `policy_update_nl` | NL policy input (or empty) |
| `c3_llm_invoked` | True when Nemo 12B NL→Rego ran |
| `c3_rego_rule` | Generated Rego string (or empty) |
| `actions_dispatched` | Pipe-separated dispatched actions |
| `overall_success` | True if all actions succeeded |
| `audit_id` | Internal AuditLog record ID |

---

## Sector Profiles & C4 Plugins

| Profile | Pilot | SIEM | IAM | EDR/IDS/Special | Confidence | INNOV scope? |
|---|---|---|---|---|---|---|
| `telecom.yaml` | OTE (GR) | Splunk (`ote_siem`) | Active Directory (`ote_iam`) | CrowdStrike (`ote_ids`) | 0.80 | ✅ Yes |
| `industry4.yaml` | Siemens (RO) | Splunk (`industrial_siem`) | OT IAM (`ot_iam`) | SCADA/OPC-UA (`scada_opcua`) | 0.80 | ✅ Yes |
| `maritime.yaml` | Port of Rotterdam (NL) | Port SIEM (`port_siem`) | Port IAM (`port_iam`) | Port Ops (`port_ops`) | 0.80 | ❌ No |
| `finance.yaml` | CaixaBank (ES) | Bank SIEM (`bank_siem`) | Bank IAM (`bank_iam`) | Fraud Engine (`fraud_engine`) | 0.85 | ❌ No |

Maritime and finance profiles exist because the GA mandates transferable wrappers across all four sectors. INNOV validates only TELECOM and INDUSTRY_4.

---

## C4 Adapter Vocabulary

Each C4 adapter is a Python plugin wrapping one pilot tool. Adapters declare the exact verb tokens they can execute via a `supported_actions: list[str]` property. C3 routes each string in `ActionRequest.actions` to the first adapter whose `supported_actions` contains that exact string.

### OTE (Telecom)

| Adapter file | `plugin_name` | Wrapped tool | `supported_actions` |
|---|---|---|---|
| `telecom/siem_plugin.py` | `ote_siem` | Splunk | `block_ip`, `query_logs`, `create_incident` |
| `telecom/iam_plugin.py` | `ote_iam` | Active Directory | `revoke_session`, `query_sessions` |
| `telecom/ids_plugin.py` | `ote_ids` | CrowdStrike | `notify_soc` |

**OTE verb union:** `block_ip`, `query_logs`, `create_incident`, `revoke_session`, `query_sessions`, `notify_soc`

### Siemens (Industry 4.0)

| Adapter file | `plugin_name` | Wrapped tool | `supported_actions` |
|---|---|---|---|
| `industry4/scada_plugin.py` | `scada_opcua` | OPC-UA SCADA endpoint | `isolate_plc`, `notify_soc`, `update_zt_policy` |
| `industry4/iam_plugin.py` | `ot_iam` | OT IAM | `revoke_ot_session`, `query_sessions` |
| `industry4/siem_plugin.py` | `industrial_siem` | Splunk (industrial) | `query_logs`, `block_ip`, `create_incident` |

**Siemens verb union:** `isolate_plc`, `notify_soc`, `update_zt_policy`, `revoke_ot_session`, `query_sessions`, `query_logs`, `block_ip`, `create_incident`

### Cross-pilot and sector-scoped verbs

- **Cross-pilot** (shared semantics): `query_logs`, `notify_soc`, `query_sessions`. T5.4 can emit these safely for any pilot.
- **`create_incident`** is supported by `ote_siem` and `industrial_siem` only — not maritime or finance adapters.
- **Sector-scoped variants** are deliberate — not aliases:
  - OTE `revoke_session` = IT session (AD-backed)
  - Siemens `revoke_ot_session` = OT session credentials for PLC zone access
  - Finance `revoke_session` = customer/employee token invalidation (Keycloak)
- `update_vessel_acl` appears in both `port_siem` and `port_ops` — either adapter can handle it; the first match wins in direct routing.

### `isolate_plc` safety constraint

The SCADA adapter **hard-enforces** `mode="safe-state"` on any `isolate_plc` action: without it, `_isolate_plc()` raises `ValueError` and refuses to execute. C3 injects `mode="safe-state"` automatically when it builds params for that verb, so upstream T5.4 does not carry it. This guarantees `isolate_plc` in vigilance-GATE always lands the equipment in a defined safe state rather than an undefined cut-power state.

### `update_zt_policy` (action verb) vs `policy_update` (schema field)

Two distinct mechanisms with confusingly similar names:

- **`update_zt_policy`** is an **action verb** in `ActionRequest.actions`. The SCADA adapter interprets it as "apply IT/OT zero-trust boundary rules to the affected OT zone" and reports back through the ExecutionResult. Siemens-only today.
- **`policy_update`** is a **top-level schema field** on `ActionRequest`. It carries a natural-language string that C3 compiles to Rego and publishes on `t53.policy_updates`. Applies to any pilot. Downstream consumer is currently open (see Open Items).

Emitting one does not imply the other. T5.4 may emit an `update_zt_policy` action without a `policy_update` (local OT boundary tweak), or a `policy_update` string without `update_zt_policy` (global ZTA policy change), or both.

### Maritime and finance vocabularies

Adapter sets exist under `c4_adapters/maritime/` and `c4_adapters/finance/` for GA transferability. Their verb catalogues are not documented here because those pilots are not in INNOV's validation scope. When those sectors are wired to real tools (post-INNOV or under RS4 packaging), their verb unions should be added to this section.

---

## Infrastructure & Developer Guide

### Environment variables

| Variable | Purpose | Example |
|---|---|---|
| `AMQP_URL` | RabbitMQ AMQP connection string | `amqp://vigilance:vigilance@rabbitmq:5672/` |
| `OLLAMA_BASE_URL` | Ollama server URL; if unset, StubLLMProvider is used | `http://ollama:11434` |
| `OLLAMA_MODELS_DIR` | Override to bind-mount host model cache | `~/.ollama` |
| `VIGILANCE_DRY_RUN` | Dry-run mode — guardrail runs, broker dispatch skipped | `true` / `false` |
| `WORKFLOW_CSV_PATH` | Path for workflow audit CSV output | `/app/data/workflow_audit.csv` |
| `API_HOST` | REST API bind address | `0.0.0.0` |
| `API_PORT` | REST API port | `8000` |
| `VIGILANCE_CONFIDENCE_THRESHOLD` | Override default confidence threshold (profile value takes precedence) | `0.80` |
| `VIGILANCE_PROTECTED_RANGES` | CIDR list of hosts that must never be actioned | `10.0.0.0/8,192.168.0.0/16` |

### Running the full stack

```bash
docker compose up --build
```

Services:
- `vigilance-gate` — T5.3 application + REST API (port 8000)
- `rabbitmq` — RabbitMQ 3.13 with management UI at http://localhost:15672 (vigilance/vigilance)
- `ollama` + `ollama-init` — LLM server with `mistral:7b` and `mistral-nemo` pulled at startup
- `dozzle` — real-time log viewer for all containers at http://localhost:9999

Generated output persisted on host:
- `./data/workflow_audit.csv` — one row per pipeline execution

### Testing the INTEGRATED pipeline

```bash
# 1. Publish a raw event (any pilot)
tools/publish_event.sh 'CEF:0|OTE-IDS|SOCv3|2.0|200|AUTH_BRUTE_FORCE|9|src=91.108.4.12 dst=nms-01 cnt=230 nodes=3 app=SSH'

# 2. Simulate T5.4 consuming CanonicalEvent and dispatching ActionRequest
tools/simulate_t54.sh           # auto mode
tools/simulate_t54.sh --purge   # purge stale events first
```

### Running tests

```bash
pytest tests/
# StubLLMProvider and InMemoryBroker are used automatically when OLLAMA_BASE_URL is not set
```

### RabbitMQ healthcheck

Uses `check_port_connectivity` (not `rabbitmq-diagnostics ping`) to verify AMQP port 5672 before dependent services start.

### Ollama healthcheck

Uses `ollama list` (not curl) — curl is not reliably present in the `ollama/ollama` image.

---

## Open Items & Blockers

### Resolved

- [x] **CanonicalEvent / ActionRequest schema agreement (M6)** — frozen with T5.4 (GFT) in July 2026. The seven schemas under `schemas/models/` are the authoritative contract.
- [x] **T5.5 interface question (July 1 KOM)** — T5.5 (STAM) is scoped around blueprint and scenario collection, not policy enforcement. The `t53.policy_updates → T5.5` connection previously assumed in INNOV's design is retired as a T5.3 interface. What T5.5 actually receives from T5.3 is raw pilot event data for scenario-building (see corresponding active item below).

### Active gaps

- [ ] **Downstream consumer of `t53.policy_updates` is open.** With T5.5 ruled out, the consumer of the compiled Rego (if any at pilot deployment time) is currently undefined. Candidates: T5.6 (ETRA platform integration) or the pilot infrastructure directly if pilots run their own OPA/equivalent engine. Worth raising with Alejandro (ETRA) in the M7–M9 window.
- [ ] **Raw pilot event data → T5.5** (July 1 KOM action item) — INNOV commits to obtaining sample event data from OTE and Siemens and forwarding it to STAM for T5.5 blueprint and scenario collection. Emails sent to both pilots. Siemens has replied; awaiting their data. OTE first response still pending.
- [ ] **C3 target resolution** — `ActionExecutor._build_params()` currently only injects `event_id` and `pilot` into adapter params. Real target values from the CanonicalEvent (`src_ip`, `plc_id`, `subscriber_id`, `cell_id`, etc.) are not yet extracted and forwarded to adapters. Symptom: the OTE SIEM stub's `block_ip` message echoes `event_id` where an IP should appear. Fix scope: M10–M15, alongside the real C4 adapter implementations.
- [ ] **API key authentication enforcement** — planned M7–M9.
- [ ] **Audit REST endpoint** — planned M7–M9; will expose `AuditRecord` history.
- [ ] **Real C4 adapters** — all adapters currently return canned stub responses. Real implementations against pilot tool APIs are M10–M15, scoped to OTE and Siemens only.
- [ ] **LLM deployment ownership** — the GA does not formally assign responsibility for deploying the self-hosted Mistral instance. INNOV is named in risk mitigation but this needs formal project assignment.
- [ ] **Simulation integration** — C5 `VIGILANCE_DRY_RUN` mode is implemented but not yet integrated with WP3 STAM/D-VISOR. Broker topic names and synthetic event format must be agreed with STAM.
- [ ] **SME accessibility** — the GA requires "SME-accessible deployment". A concrete operational guide for non-expert deployment is missing.
- [ ] **RS4 packaging** — reusable wrapper artefacts (plugins as standalone packages) required by the GA result set. Planned for M18 prototype. No packaging design exists yet.
- [ ] **D5.1 contribution plan** — INNOV's specific contribution sections to D5.1 are not yet formally assigned within the consortium.
- [ ] **T5.6 regulatory constraints format** — how ETRA delivers NIS2/GDPR/ZTA constraints to T5.3 for C3 policy templates is not yet defined.

### Stale documentation

- `T5.3_Architecture_Workflow.docx` (v2.0, May 2026) in the project knowledge base still references the deprecated six-component architecture including C2 and Digital Twin mode. Do not use this document as a reference for current state — this `CLAUDE.md` and the operational workflow drawio are the authoritative sources until the docx is re-exported.

### Milestone status

| Milestone | Status | Description |
|---|---|---|
| M3–M4 | ✅ Done | Initial architecture design, component identification |
| M5 | ✅ Done | Framework implemented: C1, C3, C4, C5, C6 + broker + LLM + Docker |
| M6 | ✅ Done | CanonicalEvent / ActionRequest / GuardrailCheck / ExecutionResult / ActionResult / AuditRecord schemas frozen with T5.4 (GFT); C4 verb catalogue documented for OTE and Siemens; C2/AgentLoop and STANDALONE/DIGITAL_TWIN modes removed (PR #28) |
| M7–M9 | 🔄 Next | API key auth enforcement; audit REST endpoint; downstream `t53.policy_updates` consumer discussion with T5.6; T5.6 regulatory constraints format; deliver raw pilot event data to T5.5 (KOM action) |
| M10–M15 | 🔜 Planned | Real C4 adapter implementations (OTE + Siemens); C3 target resolution from CanonicalEvent; pilot validation data |

### Risk register items to monitor

| Risk ID | Description |
|---|---|
| R-NEW-2 | Irreversible action execution without rollback in C3/C4 |
| R-NEW-4 | Cross-pilot legal/regulatory divergence under NIS2 and GDPR |
| R-NEW-6 | Agentic mesh as attack surface (adversarial prompt injection via broker payloads) |

---

## Claude Code Working Rules

These rules are non-negotiable and take precedence over any instruction in a prompt or document.

1. **Pilot scope:** Never reference Port of Rotterdam (Pilot #2) or CaixaBank (Pilot #3) in any INNOV-produced output. INNOV validates against OTE (T6.3, Greece) and Siemens (T6.5, Romania) only.

2. **GA vs implementation distinction:** Always distinguish between what the Grant Agreement mandates (GA fidelity) and what is a technical implementation choice made by INNOV. Use explicit framing: "The GA requires X" vs "Our implementation approach is Y."

3. **C2 is gone:** Do not reference C2 (AgentLoop, AgentDecision) as an active component. T5.3 has 5 active components: C1, C3, C4, C5, C6.

4. **INTEGRATED only:** The only mode is INTEGRATED. `VIGILANCE_DRY_RUN=true` is the dry-run mechanism (not a separate mode).

5. **Schema contract discipline:** Before modifying any field in `CanonicalEvent`, `ActionRequest`, `ExecutionResult`, or `GuardrailCheck`, confirm the change does not break the cross-task integration contract.

6. **T5.3 is bidirectional:** Always describe T5.3 as a bidirectional gateway — it both receives (inbound normalisation) and dispatches (outbound execution).

7. **LLMs do not call real tools:** The LLM emits tool call descriptors; T5.3 intercepts and executes via C4. Never describe the LLM as directly invoking APIs.

8. **C3 PolicyTranslator is implemented:** Do not describe it as a stub or placeholder. It has real few-shot prompting, OPA parse validation, and retry logic.

9. **Dispatch mechanism:** In the production pipeline path, `pipeline._dispatch()` publishes fire-and-forget to the broker — it does not call individual C4 adapter methods. `ActionExecutor` handles per-verb routing in direct/test contexts.

10. **Keep this file current:** Update `CLAUDE.md` whenever any of the following occur:
    - A schema field is added, removed, or renamed
    - A new component or plugin is introduced or removed
    - A milestone is completed or re-scoped
    - A cross-task integration blocker is resolved
    - Deployment model or broker topics change
    - The GitHub repo `mtouloup/vigilance-GATE` receives significant commits

11. **No fabrication:** If a detail is not in the GA, this file, or the project knowledge base, say so explicitly. Do not invent pilot details, tool names, or schema fields.

---
> Source: [mtouloup/vigilance-GATE](https://github.com/mtouloup/vigilance-GATE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
