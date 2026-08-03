## asvs-generalized

> A documentation-only repository: an unofficial third-party methodology applying OWASP ASVS to non-web software. No source code, no build, no tests — every change is to a Markdown file.

# asvs-generalized

A documentation-only repository: an unofficial third-party methodology applying OWASP ASVS to non-web software. No source code, no build, no tests — every change is to a Markdown file.

## What lives here

- [`README.md`](README.md) — public-facing landing page; cross-links the rest.
- [`ASVS_METHODOLOGY.md`](ASVS_METHODOLOGY.md) — how an audit is conducted (classification, severity, citation, structure).
- [`APPLICABILITY_MAP.md`](APPLICABILITY_MAP.md) — default applicability of each ASVS chapter for non-web software.
- [`AUDIT_TEMPLATE.md`](AUDIT_TEMPLATE.md) — blank audit-instance scaffold. **This is a template meant to be copied into other projects; preserve its placeholder-with-HTML-comments structure when editing.**
- [`THREAT_MODELING.md`](THREAT_MODELING.md) — companion describing the threat-model shape the methodology expects to cite.
- [`LICENSE`](LICENSE) — CC-BY 4.0.

## Cross-document consistency

The four substantive docs share a vocabulary that must stay synchronized:

- Verdicts: Pass / Partial / N/A / Finding / Documented gap.
- Severities: blocker / warning / nit.
- Positive-finding ID pattern: `V{n}-P{m}`.
- Default in-scope chapters: V1, V4, V5, V6, V7, V10, V12, V14. Default N/A: V2, V3, V8, V9, V11, V13.

When changing one document, sweep the others for matching terms before committing. The audit-template chapter list, the applicability-map applicable-chapters list, and the methodology's classification scheme must agree.

## Substantive details that are easy to miss

- **Vigil is the canonical worked example.** References to https://github.com/Logopher/Vigil (its `THREAT_MODEL.md` and `ASVS_AUDIT.md`) are real demonstrations, not placeholders. If a new example is needed, take it from Vigil — do not invent hypothetical projects.
- **OWASP non-affiliation is load-bearing.** The "unofficial third-party methodology … not affiliated with or endorsed by the OWASP Foundation" disclaimer in `README.md` is required for the CC-BY reuse posture; do not soften, remove, or rephrase without explicit instruction.

## Voice

Declarative and concrete. Prefer specific examples to categorical claims — `THREAT_MODELING.md` codifies this rule and the rest of the repo follows it. Avoid marketing language.

## Commits

Global rules apply. Scopes for this project (one per commit):

- `methodology` — `ASVS_METHODOLOGY.md`
- `applicability` — `APPLICABILITY_MAP.md`
- `template` — `AUDIT_TEMPLATE.md`
- `threat-model` — `THREAT_MODELING.md`
- `readme` — `README.md`
- `repo` — repository-wide changes (LICENSE, multi-file rewordings, structural moves)

Types in practice: `docs:` (content), `chore:` (LICENSE, housekeeping), `refactor:` (cross-document restructuring with no content delta).

## Reviewer agents

No code to review; the `code-reviewer` gate from the global profile is not meaningful here and may be skipped. For non-trivial cross-document edits, invoking `architect` is still useful — it forces a written plan listing which files need to change together.

---
> Source: [Logopher/asvs-generalized](https://github.com/Logopher/asvs-generalized) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
