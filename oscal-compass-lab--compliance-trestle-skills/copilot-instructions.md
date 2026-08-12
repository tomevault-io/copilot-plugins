## oscal-json-edit

> Prefer Trestle markdown roundtrip over direct OSCAL JSON edits


Direct edits to OSCAL JSON or YAML in catalogs, profiles, component definitions, or SSP directories bypass the Trestle markdown authoring roundtrip.
They can cause validation or assembly issues.

Prefer:

1. `trestle author *-generate` to make markdown
2. Edit markdown with source references
3. `trestle author *-assemble` to regenerate OSCAL
4. `trestle validate -a` or `/workspace-validate`

If a direct JSON or YAML edit is intentional, continue with care and validate afterward.
Assessment plans, assessment results, and POA&M models use the JSON split and merge workflow.
See `commands/workflow/assessment-roundtrip.md` and `commands/workflow/poam-roundtrip.md`.

---
> Source: [oscal-compass-lab/compliance-trestle-skills](https://github.com/oscal-compass-lab/compliance-trestle-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
