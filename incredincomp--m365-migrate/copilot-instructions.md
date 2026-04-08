## m365-migrate

> This repository contains an operator-first Microsoft 365 tenant-to-tenant migration runbook and automation toolkit. It is built for low-cost execution, native Microsoft tooling first, and strict safety controls that block destructive actions until objective validation evidence is recorded.

# AGENTS.md

## Repo Purpose
This repository contains an operator-first Microsoft 365 tenant-to-tenant migration runbook and automation toolkit. It is built for low-cost execution, native Microsoft tooling first, and strict safety controls that block destructive actions until objective validation evidence is recorded.
The repository now includes a stateful autonomous-assist orchestrator for phase execution, evidence assertions, gate checks, and operator summaries.

## Migration Philosophy
- Native-first: use Exchange Online cross-tenant mailbox migration and Microsoft-native SharePoint/OneDrive migration capabilities where practical.
- Lowest-cost reasonable path: use PowerShell automation, exports, and checklists before considering paid tooling.
- Validate before destroy: no source teardown actions are allowed before migration usability is proven.

## Non-Negotiable Safety Rule
The source domain must not be removed and the source tenant must not be decommissioned until per-user validation, dependency scans, mail-flow validation, admin sign-in validation, rollback readiness, and required evidence artifacts have all passed and been recorded.

## Forbidden Actions
- Removing the source custom domain before dependency scans and validation gates pass.
- Deleting or disabling source admin access before decommission gates pass.
- Cancelling source licensing early if it impairs validation, rollback, or coexistence access.
- Deleting migration batches before migration verification is complete and recorded.
- Performing destructive actions without required evidence files under `evidence/`.
- Releasing source-domain objects before shared mailbox/resource mailbox/group/contact/address checks pass.

## Required Technical Safeguards
- Preserve and validate aliases/proxy addresses and `x500`/`LegacyExchangeDN` handling.
- Validate target object population before migration starts.
- Validate migrated users can sign in, send/receive mail, access historical mailbox content, and access calendars.
- Validate OneDrive/SharePoint access if workload is in scope.
- Scan and remediate shared mailboxes, resource mailboxes, distribution lists, M365 groups, contacts, and admin sign-ins tied to source domain.
- Enforce hold period before final decommission unless explicitly waived with recorded sign-off.
- Keep migration batches until verification is complete.

## Execution Model
- Multi-task work is allowed across phases, but destructive tasks are blocked behind validation gates.
- Checklist status alone is insufficient. Required CSV/JSON evidence files must exist, be populated, and be referenced in `IMPLEMENTATION_PROGRESS.md`.
- If evidence is missing, empty, stale, or inconsistent, gate state is `FAIL/HOLD`.
- Operator performs final cutover and decommission steps manually using checklist and sign-off blocks.
- State-driven execution is mandatory:
  - `agent/runbook-state.json` and `state/current-phase.json` are the source of truth.
  - Agent phases run via `scripts/agent-entry.ps1` and `scripts/invoke-agent-phase.ps1`.
  - Every phase run writes `logs/operator-summary.md`.
- Approval-gated phases:
  - `CutoverAssist` requires populated `state/approvals.json` (`cutoverAssist`).
  - `DecommissionAssist` requires populated `state/approvals.json` (`decommissionAssist`).
  - Required approval fields: `approvedBy`, `approvedAtUtc`, and `changeTicket` or `justification`.
- Prior-gate enforcement:
  - Later phases must satisfy task prerequisite gates before running.
  - Gate assertions also verify prerequisite gate status from `agent/runbook-state.json`.
- Evidence quality enforcement:
  - Missing, empty, stale, malformed, or schema-invalid evidence causes `HOLD`.
  - Freshness windows are enforced by evidence scope (24h discovery/prereqs, 12h pilot/bulk, 2h cutover/decommission).
- DryRun enforcement:
  - Orchestrator DryRun is non-mutating for tenant objects.
  - Manual/destructive boundaries remain blocked regardless of DryRun state.

## Manual-Only Boundaries
- DNS cutover execution is manual-only.
- Source custom domain removal is manual-only.
- Final source tenant decommission is manual-only.
- Agent may prepare, validate, and summarize these steps but must not execute them automatically.

## Operator Expectations
- Validate exact tenant prerequisites and permissions in Microsoft documentation before execution.
- Review script placeholders marked `VERIFY IN TENANT / DOC` and set environment-specific values in config files.
- Record all gate decisions and reviewer notes in `IMPLEMENTATION_PROGRESS.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/incredincomp)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/incredincomp)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
