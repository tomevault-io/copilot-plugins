## repo-gov-live-1775550547047

> @.devops/generators/claude/root-template.md

# Repository Governance Guidance

## Canonical Memory Imports

@.devops/generators/claude/root-template.md
@.devops/generators/claude/import-map.yaml

## Canonical Root

- `.devops/` is the canonical governance truth root directory.
- Files under `.devops/` are authoritative because they live under that root, not because generated projections repeat their content.

## Canonical Governance Inputs

- Treat `.devops/index/file-index.json` as the canonical file graph snapshot.
- Treat `.devops/policies/ownership-map.yaml`, `.devops/policies/gate-rules.yaml`, and `.devops/graph/manifests/capability-map.yaml` as canonical governance data.
- Treat `AGENTS.md`, `CLAUDE.md`, and `.github/*` as generated projections. Regenerate them instead of editing by hand.

## Execution Rules

1. Configure provider credentials before analysis.
2. Use linked git worktrees for any write-capable repo flow.
3. Preview and verify generated projections before publish.
4. When present, consult `.devops/suites/*/PLAYBOOK.md` before changing governed behavior.
5. Anthropic / Claude API access is optional for now; keep parity at the projection and memory-template layer until credentials are available.

## Scoped Memory Strategy

- Nested `CLAUDE.md` files are generated only when a subtree carries canonical owner and suite playbook context.
- Use `.devops/generators/claude/subtree-template.md` plus `.devops/generators/claude/import-map.yaml` to audit or regenerate scoped memories from canonical data.
- Scoped Claude memories must stay owner-aligned and should import canonical owner contracts and suite playbooks instead of retyping them.

## Capability Owners

- owner-backend-api: backend.api -> 1 files | github @nyartsgnaw
- owner-domain-logic: domain.logic -> 1 files | github @nyartsgnaw
- owner-external-integration: external.integration -> 1 files | github @nyartsgnaw
- owner-infra-workflow: infra.workflow -> 1 files | github @nyartsgnaw
- owner-llm-modeling: llm.modeling -> 2 files | github @nyartsgnaw

## Required Checks

- check-backend-api
- check-domain-logic
- check-external-integration
- check-infra-workflow
- check-llm-modeling

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nyartsgnaw)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nyartsgnaw)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
