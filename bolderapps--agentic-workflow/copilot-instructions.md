## 06-observability

> Observability conventions. Applied to every feature that has runtime behavior.


# Observability rules

Every task that introduces runtime behavior (API handler, background job,
client mutation, service) must ship with observability built in. Observability
is part of Definition of Done, not a separate later task.

## 1) Structured logging

- Every service and API handler logs at entry and exit (success + error path).
- Logs are structured (JSON) with at minimum:
  - `level` (`debug` | `info` | `warn` | `error`)
  - `event` (short, snake_case, stable name — e.g. `login_attempt`,
    `booking_created`, `payment_failed`)
  - `actor_id` when available (do not log raw credentials or tokens)
  - `request_id` / `trace_id` when available
  - `duration_ms` for any boundary-crossing call
- Do not log PII fields (email/phone/address) in full. Hash or truncate
  (`email_hash`, `email_domain_only`) if the signal is needed.
- `console.log` / `print()` are not acceptable outside scripts. Use the
  project logger.

## 2) Error reporting

- Every `catch` path either logs at `error` level or rethrows — never both
  silently swallows and returns a success shape.
- Uncaught errors surface to the project's error reporter (Sentry, Rollbar,
  Datadog, etc.) when one is configured.
- Error responses carry a `code` the UI can switch on. The human-readable
  `message` is separate from the machine `code`.

## 3) Metrics

For any task that introduces a user-visible flow or a cross-service call,
add at least one counter and one duration metric:

- `event_count{event="<event_name>", outcome="success|failure"}`
- `event_duration_ms{event="<event_name>"}`

Use whatever library the project declares (OpenTelemetry, StatsD, Prometheus
client). If no metrics library is configured, the first task that needs one
adds it and records the choice in `memory/decisions.md`.

## 4) Tracing

For any request that crosses a process or network boundary:

- Propagate `trace_id` / `request_id` across the boundary (header or context).
- Child spans for external calls (DB, third-party API, queue) when the
  project uses OpenTelemetry or equivalent.
- No task marks `done` with a new cross-service call that drops trace context.

## 5) Client-side telemetry (UI tasks)

For user-facing flows:

- Emit an analytics event at the decision points defined in the feature's
  `User flow`:
  - flow entry
  - each meaningful user action
  - flow success / flow failure / flow abandonment
- Event names follow `object_action` convention (`login_submitted`,
  `booking_confirmed`, `search_empty_result_shown`).
- Do not emit events with free-form user content as property values.

## 6) What QA checks

QA rejects a task if any of the following is true for affected files:

- A new code path has no log line at error boundary.
- A new API route has no entry/exit log or duration metric.
- A new user-facing action has no analytics event when the feature's
  `User flow` lists it as a decision point.
- Any log line prints a raw token, password, full email, or PII without
  hashing/truncation.
- A new cross-service call drops `trace_id` propagation.

## 7) What is out of scope here

This rule does not require dashboards, alerts, or SLO definitions. Those live
in ops work. This rule only covers the instrumentation that agents must emit
so the resulting system is observable when ops work happens.

---
> Source: [BolderApps/agentic-workflow](https://github.com/BolderApps/agentic-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
