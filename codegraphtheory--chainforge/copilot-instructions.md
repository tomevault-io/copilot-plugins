## chainforge

> This repository is a Hermes Agent profile distribution for ChainForge, a production blockchain engineering agent. It is designed to be maintained by AI coding agents.

# ChainForge: Agent Instructions

This repository is a Hermes Agent profile distribution for ChainForge, a production blockchain engineering agent. It is designed to be maintained by AI coding agents.

## Hard rules

1. Never commit secrets. `.env` is forbidden. `.env.EXAMPLE` is allowed.
2. Preserve `distribution.yaml` at the repository root.
3. Keep the profile installable with `hermes profile install <source>`.
4. Run `python3 scripts/validate_profile.py .` after substantive edits.
5. Keep instructions concrete and operational.
6. Do not claim tools, audits, partnerships, deployed contracts, community channels, or integrations that are not verified.
7. Do not weaken ChainForge refusals around rug pulls, hidden controls, exploit activity, or deceptive tokenomics.

## Blockchain profile maintenance

When editing ChainForge:

1. Preserve the security-first identity in `SOUL.md`.
2. Keep EVM, Solana, Move, CosmWasm, tokenomics, governance, and audit-readiness scope accurate.
3. Update `skills/chainforge-blockchain-engineering/SKILL.md` when adding reusable blockchain workflow guidance.
4. Keep `config.yaml` and `distribution.yaml` aligned with files that actually exist.
5. Run validation and at least one install smoke test before pushing.

## Interactive profile creation

If the user asks this profile to create another Hermes profile:

1. Ask only for missing essentials: name, mission, target user, required integrations, and risk level.
2. Write a params YAML file using `templates/profile.params.yaml` as the schema reference.
3. Run `python3 scripts/generate_profile.py --params <params.yaml> --output <target-dir>`.
4. Run `python3 <target-dir>/scripts/validate_profile.py <target-dir>`.
5. Report the target path and validation output.

## Handoff format

When finishing, report files changed, commands run, validation output, and remaining manual steps.

---
> Source: [codegraphtheory/chainforge](https://github.com/codegraphtheory/chainforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
