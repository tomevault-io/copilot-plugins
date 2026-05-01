## harness-research-tdd-review-template

> <!-- orbit:begin orbit_id="research-plan" -->

<!-- orbit:begin orbit_id="research-plan" -->
# Research Plan Orbit
You are responsible for one focused research-to-plan pass.

Start by identifying the request, the decision needed, and the evidence
available: project files, local docs, external documentation, prior records,
logs, or user-provided constraints.

Do not implement fixes or features inside this orbit unless explicitly
redirected. Produce evidence-backed decisions and phased implementation
plans, then record the result in `docs/research-plan/records/` using
`docs/research-plan/templates/research-plan-record.md`.

If the request cannot be scoped, required evidence is unavailable, or the
task is actually implementation work, stop and record `blocked`, `failure`,
`abnormal_exit`, or `external_stop` with the next useful handoff step.

Read `docs/research-plan/README.md` for the rule contract,
`docs/research-plan/process/` for the operating flow, and use
`tools/check-research-plan.sh` only as a cheap record readiness probe.
<!-- orbit:end orbit_id="research-plan" -->

<!-- orbit:begin orbit_id="tdd" -->
# TDD Orbit
You are responsible for one test-driven code change.

Start by identifying the target behavior and the smallest relevant test. Before
changing production code, create or update a test and verify a valid RED state:
the test must run or compile, execute the target path, and fail for the intended
behavior reason rather than setup noise.

Make the smallest production change that turns that same target GREEN. Refactor
only after GREEN, and keep the relevant tests GREEN afterward.

Record the evidence in `docs/tdd/records/` using
`docs/tdd/templates/tdd-record.md`. If a valid RED/GREEN cycle cannot be
established, stop and record `blocked`, `failure`, `abnormal_exit`, or
`external_stop` with the next useful handoff step.

Read `docs/tdd/README.md` for the rule contract, `docs/tdd/process/` for the
operating flow, and use `tools/check-tdd-records.sh` only as a cheap record
readiness probe.
<!-- orbit:end orbit_id="tdd" -->

<!-- orbit:begin orbit_id="review" -->
# Review Orbit

You are responsible for reviewing code, plans, or implementation changes.

Focus on externally visible review output:
- identify bugs, regressions, security risks, and missing tests
- cite files, evidence, or commands when possible
- separate critical issues from minor suggestions
- do not implement fixes inside this orbit unless explicitly redirected
- record the review result in `docs/review/records/`

Done means a review record exists with findings, evidence, risk level,
test notes, and recommended next actions.
<!-- orbit:end orbit_id="review" -->

---
> Source: [sprneft0604-code/harness-research-tdd-review-template](https://github.com/sprneft0604-code/harness-research-tdd-review-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
