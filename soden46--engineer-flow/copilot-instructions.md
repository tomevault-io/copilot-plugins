## engineer-flow

> Use `skills/engineer-flow/SKILL.md` as the canonical entrypoint for engineering tasks.

# Engineer Flow Agent Instructions

Use `skills/engineer-flow/SKILL.md` as the canonical entrypoint for engineering tasks.

## Runtime Model

Engineer Flow consists of:

- one public `engineer-flow` Agent Skill
- 16 generalized internal engineering capabilities under `skills/engineer-flow/core/`
- conditional persistent project memory under `skills/engineer-flow/infrastructure/memory-management/`
- compatible user-installed external Agent Skills
- sparse specialist routing with at most 2 development specialists
- mandatory post-development security verification

Framework-specific and technology-specific expertise should come from project evidence, native stack mechanisms, or relevant external Agent Skills.

Do not add built-in framework adapters.

## Routing Rules

- Start from the engineering concern, not the framework.
- Select at most 2 development specialists:
  - primary
  - optional support
- Prefer generalized internal capabilities for generic engineering tasks.
- Activate external skills only when task or project evidence specifically supports them.
- Do not activate unrelated external skills because of generic keyword overlap.
- Current project code and configuration take precedence over assumptions.
- Prefer the smallest correct change.
- Preserve existing project conventions where reasonable.
- Avoid unnecessary repository-wide exploration.

## Memory Contract

Memory is conditional infrastructure and never consumes a primary or support specialist slot.

Use memory preflight only when prior project/session/workflow context could materially affect correctness.

Prefer a host-provided MCP memory tool when available. Otherwise use:

`skills/engineer-flow/infrastructure/memory-management/scripts/memory.mjs`

Current code and configuration override stale memory.

Checkpoint only durable reusable project knowledge and never store secrets, credentials, `.env` values, raw tokens, or personal data.
## Security Contract

After development, run the mandatory security verification stage.

Security capability:

`skills/engineer-flow/core/security/SKILL.md`

Final result must be exactly one of:

`SECURITY REVIEW: PASS`

`SECURITY REVIEW: NEEDS_FIX`

Security verification does not consume one of the two development specialist slots.

If actionable findings remain, fix them and re-test before completion.

## Runtime Files

Resolver:

`skills/engineer-flow/scripts/engineer-flow.mjs`

Security gate:

`skills/engineer-flow/scripts/security-gate.mjs`

Security gate installer:

`skills/engineer-flow/scripts/install-security-gate.ps1`

Validation:

`skills/engineer-flow/scripts/validate.mjs`

## Validation

Run:

```bash
npm run validate
npm run self-test
npm run inventory
npx --yes skills add . --list
```

Expected invariants:

```text
INTERNAL_SKILLS=16
MAX_SPECIALISTS=2
POST_DEVELOPMENT_SECURITY=ENABLED
```

The public Agent Skills install surface should expose only:

```text
engineer-flow
```

Do not commit, tag, release, or publish unless explicitly requested.

---
> Source: [soden46/engineer-flow](https://github.com/soden46/engineer-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
