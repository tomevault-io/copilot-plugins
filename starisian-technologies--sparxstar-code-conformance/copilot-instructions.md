## sparxstar-code-conformance

> Starisian Technologies --- Copilot Instructions

Starisian Technologies --- Copilot Instructions
=============================================

This repository defines coding standards. Treat this repo as the source of policy, not product implementation.

Reference repositories (read via MCP)
-------------------------------------

Before reviewing any PR in this repo, read these repos: two others via MCP, plus this one for
context. This section is covered by the OQ-007 carve-out (see `QUESTIONS.md`): these instructions
are operational, not a coding standard, and are unusable without the exact repo identifiers to
fetch via MCP.

-   **ADR Registry:** `Starisian-Technologies/sparxstar-architecture-governance-registry`.
-   **Product Specs:** `Starisian-Technologies/sparxstar-product-specification-registry`.
-   **This repo:** `Starisian-Technologies/sparxstar-code-conformance` --- you are here. Standards, configs, workflows, matrices.

See `docs/platform-setup/STARISIAN-GOVERNANCE-PLATFORM-SETUP.md` for the pinned `uses:` examples
for the first two repos — the names above are the ones to use for MCP fetches directly.

If an MCP fetch for either of the first two repos returns 404: report the failure and the exact
identifier you used, and stop there. Do not guess whether the cause is a wrong name or a missing
installation/token grant — that determination is a platform admin's, not yours.

Mission
-------

-   Keep standards enforceable, testable, and technology-agnostic by default.
-   Encode stack-specific rules only where implementation constraints demand it.
-   Preserve consistency between `docs/`, root reference files, and agent guidance.

Authoritative sources (within this repo)
----------------------------------------

1.  `docs/standards-catalog.md` --- master catalog, read this first
2.  `docs/standards-handbook.md` --- global principles
3.  `docs/*-standard.md` --- per-language standards
4.  `docs/enforcement-matrix.md` and `CI-Enforcement-Matrix.md` --- enforcement mapping

Legacy material at the root (`ENGINEERING-STANDARDS.md`) is reference only; not final authority unless merged into the canonical docs.

Review checklist
----------------

On every PR, check:

1.  **ADR traceability.** Does every standard cite its source ADR/INV? A standard with no source has no authority --- flag it.
2.  **ADR compliance.** Read the ADR registry via MCP. Does the standard contradict any decision or invariant? Flag with the number.
3.  **OQ discipline.** Does the standard assume an OPEN OQ is resolved? Flag with the OQ number.
4.  **Spec consistency.** Read the product specs via MCP. Does the standard conflict with how a product is specced? Flag both.
5.  **Matrix honesty.** Is a row marked ENFORCED without a workflow? Flag --- status should be SPECIFIED.
6.  **Trademark discipline.** Any product name, repo name, service name, or trademark? Flag --- this is the org-wide repo, zero product names. **Exception: see OQ-007 in `QUESTIONS.md`** for the carve-out's exact scope and the three locations it applies to --- do not flag repo names inside those.
7.  **Governance snapshots.** Is the PR editing a file under `.github/instructions/governance/`? Flag --- auto-synced, read-only.

Trademark discipline
--------------------

Zero product names. Refer to capabilities by generic role: "the authority layer", "the auth SDK", "the audio capture SDK", "the runtime layer". If a rule only makes sense with a product name, the rule belongs in that product's repo. **Exception: see OQ-007 in `QUESTIONS.md`** for the carve-out's exact scope.

Non-negotiable engineering rules
--------------------------------

-   If a rule cannot be enforced or verified, it is incomplete.
-   Sanitize → Validate → Escape (in that order).
-   No silent failure.
-   Bounded execution and explicit limits are mandatory.
-   Fail-closed for authority, trust, and safety decisions.
-   Platform abstractions required; no hardwired provider-specific behavior.

Stack coverage
--------------

All maintenance must keep standards current for: PHP, WordPress, JavaScript, React, Node, CSS, SQL, PostgreSQL, Neo4j, XML, JSON, Laravel, Vite.

Security guardrails
-------------------

-   Never commit credentials, tokens, or secrets.
-   Require parameterized database access.
-   Keep CI stages explicit and ordered.

What you must NOT do
--------------------

-   You are a reviewer, not the authority. Flag and explain. The owner decides.
-   Do not suggest edits to governance snapshot files.
-   Do not add product names in suggested changes. *Exception: see OQ-007 in `QUESTIONS.md`.*

---
> Source: [Starisian-Technologies/sparxstar-code-conformance](https://github.com/Starisian-Technologies/sparxstar-code-conformance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
