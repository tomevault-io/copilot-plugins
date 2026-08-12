## readme-first-spec-writing

> README-first specification rules for live coding interviews. Focus on scope, API contracts, edge cases, and optional conceptual Mermaid diagrams. No code.


# README-First Spec Writing Rules — General

## Purpose
This document aligns requirements before coding:
- scope and assumptions
- endpoints and behavior (high-level)
- edge cases and failure modes
- clear non-goals

Time cap: **≤ 10 minutes** total.

## Absolute Rules
- ❌ no code snippets
- ❌ no pseudo-code
- ❌ no framework/library details
- ❌ no long prose
- ✅ prefer bullets and short sections

## Preferred Structure (Keep it short)

### 1) Purpose (2–3 sentences)
- what this service does
- when the flow ends / success definition (if applicable)

### 2) API Overview
List each endpoint:
- method + path
- one-line behavior

### 3) High-Level Data Model
- core entities
- key fields and meanings
- allowed states (if any)

### 4) Edge Cases & Failure Scenarios (Must have)
Bullets that will directly become tests:
- validation failures
- not found
- conflicts / invalid state transitions
- duplicates / retries (if relevant)
- concurrency assumptions (if stateful)

### 5) Non-Goals
Explicitly list what is intentionally skipped.

## Mermaid Diagrams (Optional, Preferred When Helpful)
Use Mermaid only if it materially improves clarity.
Diagrams must be conceptual, not implementation-level.

Allowed:
- sequence diagram (request flow)
- state diagram (state transitions)
- simple component interaction (Client → API → Service → Storage)

Not allowed:
- class diagrams
- method-level detail
- framework-specific components

Use **one** diagram unless the problem truly needs two.

## Exit Criteria
README is “done” when:
- endpoints are listed
- key entities/states are clear
- top edge cases are captured
- non-goals are explicit
- diagrams (if used) are minimal and conceptual

---
> Source: [cagridursun/ai-peer-live-coding-agent](https://github.com/cagridursun/ai-peer-live-coding-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
