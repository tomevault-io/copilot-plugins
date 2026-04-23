## jobproof

> **Project Name:** JobProof

# Project Overview

**Project Name:** JobProof
**Purpose:**
JobProof is a field-service proof-of-work platform that allows organisations to issue jobs, collect evidence via a client-facing portal, and verify completion.

**Current Phase:**
Pre-production / controlled demo phase
UI and backend exist, but commercial onboarding and live billing are intentionally constrained.

---

# Product Boundaries (CRITICAL)

Claude must not introduce or assume:

- Public client onboarding
- Self-serve signup flows for customers
- Billing activation or payment execution
- Marketing language implying availability for sale
- Compliance guarantees (SOC2, ISO, GDPR certification claims)
- New auth models or permission layers

**This repo is demo-safe, investor-safe, and internal-safe.**

---

# Core Functional Flows (Authoritative)

## 1. Authentication

**Status:** Implemented and audited

- Email/password login
- Magic link login
- Password reset
- Token-based auth
- 7-day token expiry enforced
- RBAC enforced on admin routes

**Claude may:**
- Refactor for clarity
- Fix copy
- Improve error handling

**Claude must NOT:**
- Add new auth flows
- Add SSO
- Add customer signup variants

## 2. Job Management (Admin Side)

Admins can:
- Create sites
- Create jobs
- Assign jobs
- Issue portal links
- Review submissions

**Status:** Fully implemented and audited

## 3. Client Portal (Proof Submission)

Portal users can:
- Access job via secure token
- Upload evidence
- Submit completion data

**Constraints:**
- No account creation
- No persistent identity
- Token-scoped access only

**This is intentional.**

## 4. Billing & Payments

**Status:** Demo-only

- Stripe exists in test mode
- Payment links may exist as URL-only
- No backend billing logic
- No subscription execution
- No customer objects persisted

**Billing UI is non-authoritative and must be treated as placeholder/demo.**

---

# Legal & Compliance Posture

- Legal pages exist
- Language is demo-scoped
- No data processing claims
- No regulatory guarantees

**Claude must:**
- Preserve demo framing
- Avoid adding legal risk language

---

# Security & Governance

**Audited and confirmed:**
- RBAC enforced
- Admin routes protected
- No exposed sensitive data
- No false security signalling
- No implied compliance certifications

**Claude must not:**
- Add "enterprise security" claims
- Add audit/compliance badges
- Suggest SOC2/GDPR readiness unless explicitly asked

---

# Known Issues (As of Last Audit)

**HIGH severity (UI / branding only):**
- Residual "BoardingPass" branding
- Incorrect support email in multiple files
- Broken footer links (/blog, /contact, /docs)

**Claude is allowed to:**
- Fix these directly
- Run repo-wide refactors to remove brand leakage

---

# Coding Expectations

**Claude should assume:**
- TypeScript
- React / Next.js style routing
- Clean separation of components
- Minimal abstraction
- Readability over cleverness

**Preferred patterns:**
- Explicit naming
- Shallow component trees
- No speculative generalisation

---

# What Claude Should Do By Default

When asked to change something:
1. Assume smallest safe change
2. Respect existing flows
3. Avoid expanding scope
4. Avoid introducing new concepts
5. Fix copy, links, naming, consistency first

If unsure:
- Prefer commenting or flagging
- Do NOT invent missing requirements

---

# Release Discipline

This repo operates under release-gate discipline:
- UI accuracy > feature count
- Fewer flows > misleading flows
- Demo safety > commercial readiness

**Claude should optimise for credibility, not feature expansion.**

---

# Instruction Hierarchy

If instructions conflict:
1. This file (claude.md)
2. Explicit user instruction
3. Existing code reality
4. Claude defaults

---

# Final Note to Claude

**You are not here to "improve" the product.**
**You are here to make it correct, safe, consistent, and defensible.**

Assume:
- This will be reviewed by engineers, security reviewers, and investors.
- Over-engineering is a failure mode.
- Precision is success.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vm799) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
