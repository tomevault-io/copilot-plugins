## sub2api-tokenlab-task7-mn2-axwai

> This repository serves two related but distinct tracks:

# AGENTS.md - Sub2API / TokenLab Repo Guidance

## Scope
This repository serves two related but distinct tracks:
- upstream `sub2api`
- private production branch / deployment context for **TokenLab**

When working here, assume changes may affect real production systems.

## First read
Before making meaningful changes, read:
- `docs/TOKENLAB_OPS_DEV_PLAYBOOK.md`
- other relevant docs under `docs/`

Do not rely only on chat memory when the repo docs already capture the decision.

## Documentation discipline
For substantial development or operations work in this repo:
- prefer writing durable notes into `docs/`
- do not leave important deployment / architecture / pricing / rollout knowledge only in chat
- if you learn a new persistent gotcha, update or add a doc in `docs/`

Examples of knowledge that should be preserved in `docs/`:
- deployment topology
- failover / rollout procedure
- pricing workflow and scripts
- domain / Cloudflare routing rules
- OAuth account onboarding workflow
- production-safe operational rules

## Operational safety
- Verify live state before claiming something is current
- Prefer read-modify-write for admin config and pricing changes
- Do not assume test nodes, backup nodes, or legacy containers are safe to reuse without checking their current role
- For production updates, prefer existing rollout scripts and documented sequencing over ad-hoc manual steps

## Development / operations split
If the human explicitly says this session should focus on **operations**, avoid mixing in broad feature work unless asked.
If the human explicitly says feature development is happening elsewhere, prioritize:
- reviewing current repo state
- documenting known decisions
- preparing safe operational steps

## Current high-value doc
- `docs/TOKENLAB_OPS_DEV_PLAYBOOK.md`
  - primary summary of this repo's accumulated TokenLab/Sub2API development and operations knowledge

---
> Source: [Ciward/sub2api-tokenlab-task7-mn2-axwai](https://github.com/Ciward/sub2api-tokenlab-task7-mn2-axwai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
