## external-adaptation-safety

> Safely adapt external repos, packages, and upstream examples into this codebase without bypassing tenant, security, or architecture rules


# External Adaptation Safety

When adapting code, patterns, or dependencies from an external repository:

1. Classify the source first:
   - dependency
   - pattern-only
   - reference-only
   - product-copilot inspiration
2. Do not paste large upstream files verbatim when they bypass this repo's conventions.
3. Preserve repo safety boundaries:
   - `business_id` tenant scope
   - Form Requests for non-trivial validation
   - permission checks before mutations
   - CSRF/auth boundaries for web actions
   - Utils as the main business-logic layer
4. Keep UI work inside Metronic 8.3.3 patterns; do not import a second design system.
5. Prefer module-local landing points over global root pollution when the feature is bounded to one domain.
6. Do not add a new dependency without naming:
   - compatibility
   - license
   - maintenance signal
   - security impact
   - rollback path
7. For browser-agent or assistant ideas, require human approval boundaries and route the evaluation through `ai/product-copilot-patterns.md`.

If any of the above are unclear, stop and ask before changing architecture or dependency surface.

---
> Source: [kunwaso/upos612-metro](https://github.com/kunwaso/upos612-metro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
