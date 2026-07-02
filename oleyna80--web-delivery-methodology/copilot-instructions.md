## web-delivery-methodology

> Maintain a portable, tool-agnostic business process for delivering websites and

# Repository Instructions

## Purpose

Maintain a portable, tool-agnostic business process for delivering websites and
web applications with human and agent participants.

## Required workflow

For non-trivial work use:

`Plan -> Spec -> Implementation Planning / Build -> Review -> Verification`

`Implementation Planning / Build` maps to the canonical lifecycle stages that
turn approved specifications into bounded Work Blocks and implementation
results. It is not a separate lifecycle stage.

Before editing:

1. Read `PROJECT_MAP.md`, the relevant core documents, and the active Work Block.
2. Check `git status`.
3. State stage, objective, role, expected result, approved write-set, and out of
   scope.
4. Stop when the requested write-set is unclear or must expand.

Only one Coder may write during a Build stage. Reviewer and Verifier are
read-only. The Orchestrator is the only writer of the active Work Block audit
trail. Do not commit or push without explicit Owner approval.

## Methodology boundaries

- Keep `docs/00_core/` independent of specific AI products and vendors.
- Put task-size variations in profiles, not duplicate lifecycles.
- Put product-specific behavior in adapters, not core contracts.
- Give every stage explicit inputs, outputs, acceptance criteria, approver,
  checks, and failure routes.
- Give every canonical artifact one owner and one source-of-truth path.
- Treat chat transcripts and machine-local runtime logs as evidence, not project
  authority.
- Do not add application code, provider credentials, secrets, deployment
  automation, or dependencies to this repository without a separately approved
  scope.

## Required closeout

Report:

- stage, objective, role, and expected result;
- scope and out of scope;
- actions taken and files changed;
- checks and verdicts;
- residual risks and next action.

---
> Source: [oleyna80/web-delivery-methodology](https://github.com/oleyna80/web-delivery-methodology) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-02 -->
