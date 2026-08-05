## groundmesh-on-call

> GroundMesh On-Call is a Hermes-powered failed-deployment investigation and recovery agency.

# GroundMesh On-Call Project Instructions

## Product

GroundMesh On-Call is a Hermes-powered failed-deployment investigation and recovery agency.

Tagline: Verified context. Safe recovery.

The demo must prove:

```text
HTTP 500
-> source-backed investigation
-> bounded patch
-> preview verification
-> HTTP 200
-> GitHub pull request
-> unsafe alternative blocked
```

## MVP Architecture

The system contains:

1. Incident Commander
2. Context Investigator
3. Remediation Engineer
4. Verification and Release Agent
5. Deterministic Action Safety Gate

GroundMesh produces one structured Incident Context Pack that is passed explicitly to each specialist agent.

## Grounded Sources

Only use these incident sources for the MVP:

- runtime logs
- latest Git diff
- deployment configuration
- one runbook

Every factual claim must include:

- claim
- source
- source type
- observed timestamp when applicable
- authority score
- freshness score

Never invent evidence. When evidence is insufficient, return `INSUFFICIENT_EVIDENCE`.

## Required Workflow

Always follow this order:

1. Reproduce the failure
2. Collect incident sources
3. Build the Context Pack
4. Detect conflicts and stale information
5. Identify the authoritative value
6. Propose the smallest possible patch
7. Run the deterministic safety gate
8. Run tests
9. Start the preview environment
10. Send real HTTP requests to preview
11. Create a branch and pull request
12. Return `READY_FOR_APPROVAL`

Do not skip directly to patching.

## Remediation Restrictions

- Modify no more than 3 files
- Prefer exactly 1 changed file
- Never commit directly to main
- Never merge a pull request
- Never modify real production secrets
- Never execute destructive database operations
- Never update unrelated dependencies
- Never refactor unrelated code
- Allow only one automatic repair attempt
- Require human approval before production deployment

## Automatically Block

Return `BLOCKED_BY_POLICY` when:

- tests fail
- preview remains unhealthy
- more than 3 files are changed
- destructive SQL is introduced
- authentication or billing files are modified outside scope
- the patch contradicts a higher-authority source
- evidence does not support the proposed action
- secrets or production credentials would be changed

## Demo Commands

Run tests:

```bash
pytest -q
```

Start the broken service:

```bash
uvicorn app.main:app --host 127.0.0.1 --port 8000
```

Start the dashboard:

```bash
streamlit run app/ui/dashboard.py
```

## Completion Criteria

The incident is complete only when:

- the original endpoint returns HTTP 500
- the root cause has source-backed evidence
- the patch passes the safety gate
- all tests pass
- preview returns HTTP 200
- synthetic requests pass
- a pull request is created
- production deployment still requires human approval

---
> Source: [shivanshgupta365/GroundMesh-On-Call](https://github.com/shivanshgupta365/GroundMesh-On-Call) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
