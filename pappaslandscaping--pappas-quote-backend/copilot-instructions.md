## pappas-quote-backend

> - Copilot is the live source of truth for all YardDesk operational data.

# AGENTS.md

## Source Of Truth

- Copilot is the live source of truth for all YardDesk operational data.
- YardDesk should read live from Copilot for scheduling, dispatch, route printing, communications context, and related operational workflows.
- Import-based or CSV-based flows are legacy/fallback only and should not be treated as the primary architecture unless explicitly approved for emergency use.

## Schedule And Dispatch

- Schedule is the canonical YardDesk office view of live Copilot jobs for a date.
- Dispatch is a derived operational view of that same live Copilot-backed job set for routing, map display, crew grouping, and print sheets.
- Schedule and Dispatch must not behave like separate systems with separate truth.

## Local Data Rules

- YardDesk may store overlays and operational metadata on top of Copilot-backed jobs.
- Allowed local overlays include office notes, review state, hold flags, address corrections, tags, customer/property linkage, print notes, map overrides, and route-order overrides.
- YardDesk must not create a competing local source of truth for Copilot-owned job identity, date, customer, service, status, crew, address, stop order, or price.

## Execution And Completion

- Copilot is the source of truth for live execution and completion state.
- YardDesk should mirror Copilot execution state for office visibility and workflow support.
- Do not build a competing primary completion/status workflow in YardDesk unless explicitly approved as fallback-only.

## Communications

- YardDesk is the office-side operational system for texts, emails, reporting, and internal coordination.
- Communications tied to jobs should use the live Copilot-backed job model, not imported shadow rows.

## Billing Safety

- Do not change billing or invoice behavior casually when changing scheduling, dispatch, or execution flows.
- Billing-trigger changes must be explicit, reviewed separately, and designed to avoid duplicate or conflicting invoice behavior.

## Implementation Preference

- Prefer additive migrations and compatibility layers over risky cutovers.
- Fix data fidelity before UI polish.
- Do not build on top of a view that is still known to be incorrect.

---
> Source: [pappaslandscaping/pappas-quote-backend](https://github.com/pappaslandscaping/pappas-quote-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
