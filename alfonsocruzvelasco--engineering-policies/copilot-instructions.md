## engineering-policies

> - This repository is an authoritative policy corpus.

# AGENTS.md

## Project Architecture Overview
- This repository is an authoritative policy corpus.
- Source files live in `rules/` and are consumed as governance documents for other repositories.
- `rules/templates/` contains reusable templates; `rules/references/` contains supporting references.
- `rules/system/` contains scripts and system-level supporting docs.

## Directory Map
- `rules/`: authoritative policy documents.
- `rules/templates/`: starter templates (`readme`, `agents`, `prompt`, `mcp`, etc.).
- `rules/references/`: supporting references and research notes.
- `rules/system/`: scripts and infrastructure docs used by policies.
- `rules/todo/`: deferred specs, not yet authoritative.
- `docs/`: operational guides (navigation, adoption, maintenance); not normative policy text.

## Build and Test Commands
- Install hooks: `pre-commit install`
- Full checks: `pre-commit run --all-files`
- Policy guardrail check only: `rules/system/scripts/ai-prohibited-tools-check.sh --strict`

## Coding Standards
- Preserve existing document style and section hierarchy.
- Keep policy updates explicit, dated, and consistent with linked cross-references.
- Favor minimal, auditable edits over broad rewrites.
- Use lowercase-with-hyphens for new filenames unless tool standards require otherwise.

## Dependency Management
- Do not add dependencies unless explicitly required.
- If dependencies are needed, follow `rules/dependency-install-policy.md` and `rules/security-policy.md`:
  - pinned versions,
  - lockfile discipline,
  - scan/audit before merge.

## Security Constraints
- Never commit secrets, credentials, tokens, or private keys.
- Treat all AI output as untrusted until reviewed.
- Use only approved AI tooling per `rules/approved-ai-tools.md`.
- Maintain policy enforcement hooks in `.pre-commit-config.yaml`.

## Prohibited Patterns
- Do not weaken or bypass mandatory controls in policy files.
- Do not introduce contradictory guidance across policy documents.
- Do not remove cross-references without replacing them with valid targets.
- Do not add production credentials, datasets, or heavy artifacts to this repository.

## Deployment Assumptions
- This repository ships policy documents, not runtime services.
- Validation is documentation integrity plus pre-commit enforcement.
- Changes remain local unless explicitly pushed by the repository owner.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alfonsocruzvelasco) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
