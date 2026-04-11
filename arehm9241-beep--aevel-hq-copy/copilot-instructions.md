## aevel-hq-copy

> This document defines data schemas, architectural invariants, behavioral rules, and maintenance log. No production logic may contradict it without updating this file first.

# PROJECT CONSTITUTION (gemini.md) — LAW

This document defines data schemas, architectural invariants, behavioral rules, and maintenance log. No production logic may contradict it without updating this file first.

---

## 1. JSON Data Schemas

### 1.1 Raw Input (Source of Truth)

Data as received from `DATA_SOURCE_PATH` or `DATA_SOURCE_URL`. Supported shape for ingestion:

```json
{
  "schema_version": "1.0",
  "records": [
    {
      "id": "string",
      "timestamp": "ISO8601",
      "source": "string",
      "metrics": {
        "visits": "number",
        "conversions": "number",
        "revenue": "number"
      },
      "meta": {}
    }
  ],
  "metadata": {
    "generated_at": "ISO8601",
    "source_label": "string"
  }
}
```

- `records` may be empty. `metrics` fields are optional; missing values treated as 0 in analytics.
- Alternative: CSV with headers; tool normalizes to same logical structure (id, timestamp, source, visits, conversions, revenue).

### 1.2 Cleaned Data (Tool Output — .tmp/cleaned_data.json)

Output of `clean_data` tool:

```json
{
  "schema_version": "1.0",
  "cleaned_at": "ISO8601",
  "record_count": "number",
  "records": [
    {
      "id": "string",
      "timestamp": "ISO8601",
      "source": "string",
      "visits": "number",
      "conversions": "number",
      "revenue": "number"
    }
  ],
  "validation_errors_count": "number"
}
```

- Flattened metrics; invalid rows dropped; counts deterministic.

### 1.3 Analytics Result (Tool Output — .tmp/analytics_result.json)

Output of `analyze` tool:

```json
{
  "schema_version": "1.0",
  "computed_at": "ISO8601",
  "period_start": "ISO8601",
  "period_end": "ISO8601",
  "totals": {
    "visits": "number",
    "conversions": "number",
    "revenue": "number"
  },
  "by_source": [
    {
      "source": "string",
      "visits": "number",
      "conversions": "number",
      "revenue": "number"
    }
  ],
  "summary": "string"
}
```

- All numeric fields from deterministic aggregation only. `summary` is optional one-line from tool (no LLM).

### 1.4 Report Payload (Tool Output — .tmp/report_output.json)

Output of `generate_report` tool (input: analytics result + optional router-formatted request):

```json
{
  "schema_version": "1.0",
  "generated_at": "ISO8601",
  "title": "string",
  "period": "string",
  "metrics": {
    "visits": "number",
    "conversions": "number",
    "revenue": "number"
  },
  "by_source": [],
  "narrative": "string",
  "format": "json"
}
```

- `narrative` may be plain text for email/Sheets; LLM may format narrative from this payload only (no new metrics).

### 1.5 Delivery Payload (Sent to Webhook or Written to .tmp/)

Body sent to `DELIVERY_WEBHOOK_URL` or written as `report_summary.txt`:

- Same structure as report payload, or minimal subset: `title`, `period`, `metrics`, `narrative`.
- Content-Type: application/json for webhook.

### 1.6 Router Request (Input to Navigation Layer)

```json
{
  "action": "ingest|clean|analyze|report|deliver|health|full_pipeline",
  "payload": {},
  "options": {}
}
```

### 1.7 Router Response (Output of Navigation Layer)

```json
{
  "route": "string",
  "tool": "string",
  "formatted_payload": {},
  "message": "string"
}
```

- Router MUST NOT add or change numeric fields; only route and format for display/forwarding.

---

## 2. Architectural Invariants

1. **Layer 1 (architecture/)**: SOPs define purpose, inputs, outputs, edge cases. SOPs are updated BEFORE code when behavior changes.
2. **Layer 2 (navigation/)**: Uses Gemini Free only. Routes requests to tool names; formats payloads for display or downstream. NEVER performs calculations or business logic; NEVER modifies schemas.
3. **Layer 3 (tools/)**: Python only. Deterministic. Atomic. Testable. Read/write intermediates in `.tmp/`. All config via environment variables.
4. **Data flow**: ingest → clean → analyze → report → send_payload. Each step reads from .tmp/ or env and writes to .tmp/ or external endpoint.
5. **No paid APIs**: Only Gemini Free and free-tier or local integrations.

---

## 3. Behavioral Rules

1. **Calculations**: Only in tools. No arithmetic or aggregation in router/LLM.
2. **Schema authority**: gemini.md is the single source of truth for input/output shapes. LLM cannot change schemas.
3. **Fail-fast**: If required env is missing or a required integration is unreachable, health check fails and pipeline does not run.
4. **Idempotency**: Same inputs to a tool produce same outputs.
5. **Self-healing**: On tool failure, log error, patch if possible, update SOP, document in progress.md.

---

## 4. Maintenance Log

| Date | Change | Author |
|------|--------|--------|
| 2025-02-02 | Initial constitution and schemas | System |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arehm9241-beep)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/arehm9241-beep)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
