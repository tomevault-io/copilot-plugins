## ocsf

> Elixir library implementing the Open Cybersecurity Schema Framework

# OCSF — Claude Code Instructions

## Project overview

Elixir library implementing the Open Cybersecurity Schema Framework
(OCSF 1.8). Persistence-agnostic core with optional `ocsf_ecto`
(Postgres) and `ocsf_clickhouse` (ClickHouse) companion libs.

## Before starting any task

1. Read the planning docs in `../.agents/`:
   - `PLAN.md` — architecture, decisions, milestones
   - `SPEC.md` — concrete APIs, struct shapes, DDL, validation
   - `QA.md` — QA strategy and per-milestone gates
   - `EVENT_CODE_FORMAT_CONFIG_SPEC.md` — event code generation
2. Check the current milestone status in `PLAN.md` §Milestones.
3. Read `DOC_GUIDELINES.md` and `TESTING_GUIDELINES.md` before
   writing any code.

## After every code change

Always run this sequence before committing:

```bash
mix format
mix test --cover --raise    # 0 failures, 0 warnings, coverage >= 90%
mix audit                   # credo, dialyzer, doctor, sobelow, deps.audit
```

Fix all issues before committing. Never commit with warnings,
failing tests, or audit failures.

## Git conventions

- Start every commit message with an emoji.
- Never add `Co-Authored-By: Claude` lines.
- Use `git add -A` before committing.
- Write concise commit messages explaining the "why".

## Code conventions

- Follow `DOC_GUIDELINES.md` for all documentation:
  - Every public module has `@moduledoc`.
  - Every public function has `@doc` with `## Examples` (doctests
    where applicable) and `@spec`.
  - OCSF schema links on every module that maps to an OCSF concept.
  - No duplicate examples between `@moduledoc` and `@doc`.

- Follow `TESTING_GUIDELINES.md` for all tests:
  - `async: true` unless touching shared state (telemetry, app config).
  - Module-based telemetry handlers, never anonymous functions.
  - Non-empty list check: `[_ | _] = list`, not `!= []` or `length > 0`.
  - Pattern-match on tagged tuples in assertions.

## OCSF field naming

- `_id` suffix for enum values: `activity_id`, `severity_id`,
  `status_id`, `auth_protocol_id`.
- `_uid` suffix for unique identifiers: `category_uid`, `class_uid`,
  `type_uid`, `metadata.uid`, `user.uid`, `correlation_uid`.
- `__` (double underscore) is the flat-projection segment separator
  everywhere: table names, column names, index names.
- Default table: `ocsf_event__logs` (prefix `ocsf_event__`, base `logs`).

## Key architectural decisions

- `jason` and `uuid_v7` and `telemetry` are runtime deps.
- `ex_json_schema`, `stream_data`, `benchee` are test/dev only.
- `status_detail` is a free-form `String.t()`, not an integer enum.
- `type_uid = class_uid * 100 + activity_id` (OCSF required).
- `metadata.product` is required (OCSF required).
- `user` is required for Authentication class (3002).
- `metadata.event_code` is derived from OCSF fields via
  `OCSF.EventCodeFormat`, not a parallel taxonomy.
- Tagged tuples only — no bang (`!`) variants in v0.

## What NOT to do

- Don't add features beyond the current milestone scope.
- Don't skip `mix audit` — it catches real issues.
- Don't use anonymous functions in `:telemetry.attach/4`.
- Don't commit `erl_crash.dump`, `_build/`, `deps/`, `cover/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/elixir-ocsf) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
