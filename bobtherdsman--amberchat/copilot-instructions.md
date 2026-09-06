## amberchat

> AmberChat is a portable, zero-install conversational operator experience built over Project Amber.

# AGENTS.md — AmberChat

## Project purpose

AmberChat is a portable, zero-install conversational operator experience built over Project Amber.

- AmberChat owns the local UI, deterministic workflow, consent, progress, and explanations.
- Project Amber remains the authoritative collector, connector, privacy, policy, tokenization, and artifact core.
- Project Amber is located at `..\amber`.

## Authoritative documentation

Before making changes, review the relevant sections of:

1. `BUILD_PLAN.md`
2. `docs/ARCHITECTURE.md`
3. `docs/REQUIREMENTS_TRACEABILITY.md`
4. `docs/OPERATOR_ACCEPTANCE_MATRIX.md`
5. Relevant schemas and tests

Do not introduce undocumented product scope or architecture changes without explicit approval.

## Module workflow

Work through modules in `BUILD_PLAN.md` in dependency order.

A module is complete only when:

- every documented requirement is implemented;
- traceability contains no pending requirement for that module;
- focused positive and negative tests pass;
- applicable operator behavior is tested through the compiled native host in Microsoft Edge;
- the complete AmberChat and Project Amber suites pass;
- npm and NuGet vulnerability audits pass;
- the module-specific Release gate emits:

  `MODULE_GATE_PASSED module=N configuration=Release`

Never mark a module complete based only on source inspection, fixture existence, direct API tests, or a generic build pass.

Run the completion gate with:

```powershell
.\scripts\Verify-Module.ps1 -Module N -Configuration Release
```

## Testing expectations

Use proportionate evidence at every applicable layer:

1. schema and canonical contract;
2. component/service behavior;
3. negative and fail-closed behavior;
4. native-host API integration;
5. real Microsoft Edge operator journey;
6. Project Amber collection and privacy behavior;
7. full Release build and dependency audits.

Operator-facing controls must be exercised through the actual browser UI. Source-pattern assertions are supplemental only.

## Architecture boundaries

- Preserve deterministic Doctor → Scan → artifact review → explicit Prepare Export sequencing.
- The model may propose intent or explanations but must not control execution policy.
- Never bypass readiness, review, entitlement, privacy, or export gates.
- Credentials must remain approved references; never place passwords, tokens, or connection strings in chat, configuration, logs, tests, or artifacts.
- Keep scan, export preparation, and submission separate.
- Do not introduce automatic upload, telemetry, remediation, arbitrary queries, or target-side agents.
- Keep customer-readable identifiers local-sensitive unless explicitly tokenized or allowlisted.

## Connector boundaries

Modules 7–10 are synthetic connector contract modules.

Do not claim live-target readiness from synthetic evidence.

Live VMware, SQL Server, and Oracle implementation and qualification—including authentication, TLS, least privilege, exact-version behavior, cancellation, timeout, no-mutation proof, and customer-like testing—belong to Module 13.

Oracle AWR always requires explicit Diagnostics Pack entitlement and must remain aggregate-only. Never include SQL text/IDs, ASH session detail, object statistics, bind data, plan hashes, raw wait events, or raw AWR reports.

## Change discipline

- Preserve unrelated user changes.
- Prefer the smallest change that satisfies documented requirements.
- Update contracts, fixtures, tests, and documentation together.
- Unknown fields and unsafe states must fail closed.
- Do not weaken tests merely to obtain a passing result.
- Fix the implementation or correct an inaccurate documented expectation with explicit justification.
- Keep generated artifacts and temporary test output out of source control.

## Completion records

After a module passes its gate:

- update its status and acceptance evidence in `BUILD_PLAN.md`;
- update `docs/REQUIREMENTS_TRACEABILITY.md`;
- update `docs/OPERATOR_ACCEPTANCE_MATRIX.md` when operator behavior applies;
- record actual test counts and the exact gate result;
- state clearly what remains deferred;
- update project memory so the next session starts from the verified state.

## Current state

Modules 1–12 are complete.

- Last verified AmberChat Release suite: 65 tests.
- Project Amber: 31 Pester tests and 8 standalone suites.
- Module 12, Tiny LLM, passed direct and native model evaluation, compiled-host lifecycle, native Edge, full-suite, and dependency-audit evidence on 2026-08-22.
- Live connector qualification remains deferred to Module 13.

---
> Source: [bobtherdsman/AmberChat](https://github.com/bobtherdsman/AmberChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
