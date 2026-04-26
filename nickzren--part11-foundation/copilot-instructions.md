## part11-foundation

> This repository is a structured knowledge base for draft assessments of software systems against 21 CFR Part 11.

# CLAUDE.md

This repository is a structured knowledge base for draft assessments of software systems against 21 CFR Part 11.

It is not a legal opinion and not a compliance oracle.

Use this file as the Claude Code bootstrap for this repository:

- Start with applicability triage before selecting control nodes.
- Produce a structured triage scope block per `core/applicability/triage-output-template.yaml`.
- Use `AGENTS.md` as the canonical shared contract for workflow, discipline rules, and prohibited shortcuts.
- Use `skills/part11-assessor/SKILL.md` as the reusable workflow layer.
- For fast navigation inside the reusable skill, use `skills/part11-assessor/START-HERE.md` and `skills/part11-assessor/references/index.yaml`.
- Apply `core/checklists/rating-rubric.yaml` for every finding.
- Consult `core/checklists/control-family-evidence-map.yaml` while exploring the target system.
- Use `docs/assessment-output-template.md` for output structure.
- Run `core/checklists/assessment-self-review.md` before finalizing.
- If the target system is outside this repository, resolve the foundation root first using `docs/skill-install-and-use.md`. Prefer `PART11_FOUNDATION_ROOT`, but allow the config-file, well-known-path, cwd, and adjacent-clone fallbacks.

Worked examples:
- `docs/example-assessment.md` — mature system with governed evidence
- `docs/example-assessment-supporting-pipeline.md` — supporting pipeline with provisional scope and repo-only evidence

Source hierarchy:

- `core/regulations/*.yaml` `authoritative_text` — binding CFR node text
- `core/guidance/scope-and-application-2003.yaml` — FDA enforcement posture and interpretive context
- `core/regulations/*.yaml` `repo_interpretation` — non-authoritative assessment guidance
- target-system evidence — determines the actual finding

Claude-specific notes:

- Do not skip triage.
- Do not treat `assessment_role: framework` nodes as standalone evidence obligations.
- Do not convert provisional scope into a final claim.
- Do not rate governance-heavy families `Adequate` on repo-only draft evidence.
- Do not use definitive record-boundary language ("X is the electronic record") when `record_boundary_status` is `provisional` or `tbd`.
- Enforcement-discretion default severity is `Observation`. Override only with explicit predicate-rule justification.
- Use classification guidance in triage.yaml: if any record-affecting path crosses the public internet, classify as `hybrid` or `open`, not `closed`.
- For multi-repo systems, identify repo ownership of each control and assess cross-repo release synchronization.
- Every non-Adequate, non-Not-applicable finding must have its own detailed section. Do not skip any.
- If the foundation root cannot be resolved, stop and ask for the path.
- If evidence is incomplete but the foundation root resolves, continue conservatively and use `Not assessed` where required instead of asking avoidable follow-up questions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nickzren) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
