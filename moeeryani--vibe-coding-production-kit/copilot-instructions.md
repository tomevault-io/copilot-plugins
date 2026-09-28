## vibe-coding-production-kit

> This file defines repository-wide operating rules for AI coding agents. Project-specific rules should override generic guidance only when explicitly documented.

# AGENTS.md

This file defines repository-wide operating rules for AI coding agents. Project-specific rules should override generic guidance only when explicitly documented.

## 1. Read before changing code

Before implementing a task, read:

1. this file;
2. the relevant product requirement and acceptance criteria;
3. affected architecture/domain/data documents;
4. relevant ADRs;
5. existing code and tests in the affected area.

Do not modify files during the planning phase unless the user explicitly requests implementation immediately.

## 2. Planning contract

Before implementation, provide a concise plan containing:

- requirement restatement;
- assumptions that materially affect implementation;
- affected modules/files;
- proposed approach;
- data/API/migration impact;
- security and privacy impact;
- edge cases and failure modes;
- tests to add or update;
- architecture conflicts, if any.

If the requested change violates an ADR, architecture boundary, security rule, or acceptance criterion, surface the conflict instead of silently working around it.

## 3. Scope discipline

- Implement only the requested task.
- Do not mix feature work with unrelated refactors.
- Do not rename/reformat unrelated files.
- Do not add a dependency unless it is necessary and justified.
- Prefer the smallest coherent diff that fully satisfies acceptance criteria.
- If a safe implementation requires a broader change, explain why and isolate it when practical.

## 4. Architecture

Adapt these rules to the project's architecture document:

- Keep business rules out of transport/UI/framework glue.
- Respect module boundaries and dependency direction.
- Access another module through its public contract, not its internals.
- Avoid circular dependencies.
- Prefer explicit domain types for meaningful concepts over raw strings/numbers.
- Important architectural decisions require an ADR.
- New abstractions require demonstrated value; avoid speculative generalization.

## 5. Data and migrations

- All persistent schema changes require versioned migrations.
- Never edit an already-released migration to change production history.
- Consider forward/backward compatibility for rolling deployments.
- Identify data backfills separately from schema changes when risk warrants it.
- Document rollback or recovery strategy for destructive or high-risk changes.
- Preserve invariants at appropriate layers; do not rely solely on UI validation.

## 6. APIs and integrations

- Validate untrusted input at trust boundaries.
- Keep API contracts explicit and versioned when appropriate.
- Do not make silent breaking changes.
- Use idempotency for retryable write operations when the domain requires it.
- Define timeouts, retries, and failure handling for external calls.
- Never expose internal errors, stack traces, secrets, or sensitive implementation details to clients.

## 7. Security

- Authentication and authorization are distinct; implement both where required.
- Authorization must be enforced server-side and be resource/action specific.
- Never log secrets, credentials, session tokens, private keys, or passwords.
- Minimize PII collection and logging.
- Use secure secret storage; never commit real secrets.
- Treat file uploads, URLs, templates, queries, redirects, and serialized data as untrusted input.
- Consider injection, XSS, CSRF, SSRF, broken access control, tenant isolation, rate limiting, replay, and abuse where relevant.
- Security-sensitive changes require explicit negative-path tests.

## 8. Error handling

- Do not silently swallow failures.
- Prefer typed/structured errors where the language supports them.
- Separate user-facing errors from diagnostic details.
- Make retryability explicit where useful.
- Preserve causal context when wrapping errors.

## 9. Observability

For important production flows, consider:

- structured logs;
- correlation/request identifiers;
- metrics;
- traces;
- domain/audit events;
- actionable alerts.

Observability must not leak secrets or unnecessary PII.

## 10. Testing

Choose tests by risk and contract, not by coverage percentage alone.

Expected layers may include:

- unit tests for isolated business rules;
- integration tests for databases/queues/external adapters;
- contract tests for service/API boundaries;
- E2E tests for critical user journeys;
- regression tests for fixed defects.

Tests should verify behavior, include meaningful negative paths, and avoid coupling to irrelevant implementation details.

## 11. Required verification commands

Replace placeholders below with the real project commands before relying on this file:

```text
INSTALL_COMMAND=<define>
FORMAT_CHECK_COMMAND=<define>
LINT_COMMAND=<define>
TYPECHECK_COMMAND=<define or n/a>
UNIT_TEST_COMMAND=<define>
INTEGRATION_TEST_COMMAND=<define>
BUILD_COMMAND=<define>
E2E_COMMAND=<define or n/a>
```

Before declaring a task complete, run every relevant configured command. Never claim a command passed if it was not executed successfully.

## 12. Self-review before handoff

Review the final diff for:

- correctness against acceptance criteria;
- architecture violations;
- authorization/security issues;
- data integrity and migration hazards;
- race conditions/concurrency problems;
- missing validation/error handling;
- duplicated or dead code;
- unnecessary complexity;
- backward compatibility;
- missing tests;
- stale docs;
- accidental unrelated changes.

## 13. Definition of Done

A task is not done until relevant items in `docs/delivery/DEFINITION-OF-DONE.md` are satisfied.

At minimum:

- acceptance criteria are satisfied;
- tests are added/updated and pass;
- required validation passes;
- security impact is addressed;
- documentation/contracts are updated;
- no unrelated changes are included.

## 14. Communication

When reporting completion, include:

1. what changed;
2. why;
3. tests/verification actually run;
4. migrations or operational impact;
5. remaining risks/known limitations.

Never hide uncertainty. Never report inferred success as verified success.

---
> Source: [Moeeryani/Vibe-Coding-Production-Kit](https://github.com/Moeeryani/Vibe-Coding-Production-Kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
