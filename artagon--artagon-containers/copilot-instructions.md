## artagon-containers

> <!-- OPENSPEC:START -->

<!-- OPENSPEC:START -->
# OpenSpec Instructions

These instructions are for AI assistants working in this project.

Always open `@/openspec/AGENTS.md` when the request:
- Mentions planning or proposals (words like proposal, spec, change, plan)
- Introduces new capabilities, breaking changes, architecture shifts, or big performance/security work
- Sounds ambiguous and you need the authoritative spec before coding

Use `@/openspec/AGENTS.md` to learn:
- How to create and apply change proposals
- Spec format and conventions
- Project structure and guidelines

Keep this managed block so 'openspec update' can refresh the instructions.

<!-- OPENSPEC:END -->

# AI Agent Instructions

This repository provides hardened, production-ready OCI container images for JVM workloads.

## Quick Reference

- OpenSpec workflow: `openspec/AGENTS.md`
- Project conventions: `openspec/project.md`
- Active changes: `openspec/changes/`
- Contributing guide: `openspec/contributing.md`

## Key Directories

| Directory | Purpose |
|-----------|---------|
| `images/` | Container image Dockerfiles (Chainguard, Distroless, UBI9) |
| `scripts/` | Build and automation scripts (resolve_jdk.sh, health-check.sh) |
| `security/` | Seccomp and AppArmor profiles |
| `policy/` | Security (SECURITY.md) and supply chain (SUPPLY-CHAIN.md) policies |
| `openspec/` | Spec-driven workflow (specs, changes, guidance) |
| `sbom/` | Software Bill of Materials storage |

## Container Build Patterns

- **Multi-stage builds** with digest-pinned base images
- **Non-root user** execution (UID 65532)
- **Security hardening**: no-new-privileges, read-only rootfs support
- **Multi-architecture**: linux/amd64 and linux/arm64

## Implementation Guidelines

### Core Rules

- Specs live in GitHub Issues (label: `spec`) and `openspec/changes/`
- Built behavior lives in `openspec/specs/`
- Link every implementation PR to its parent spec issue

### Before You Code

1. Run `openspec list` to identify the active change
2. Read `openspec/changes/<id>/proposal.md` and `tasks.md`
3. Read spec deltas in `openspec/changes/<id>/specs/` and base spec in `openspec/specs/`
4. Follow conventions in `openspec/project.md`

### Implementation Expectations

- Keep changes scoped to the spec and proposal
- Update `openspec/changes/<id>/tasks.md` as items are completed
- Maintain digest-pinned base images
- Preserve non-root user conventions
- Follow Dockerfile patterns in existing images

### Good vs Bad Patterns

**Good:**
- PR references spec issue, updates tasks.md, implements all acceptance criteria
- Uses existing build patterns and security conventions
- Maintains consistency across image variants

**Bad:**
- Adds features not mentioned in the spec or proposal
- Changes security profiles without updating specs
- Introduces hardcoded values instead of using environment variables

## For Reviews

See `docs/ai/review-instructions.md` for code review guidance including:
- Spec compliance review steps
- Compliance rating scale
- Structured review format

---
> Source: [artagon/artagon-containers](https://github.com/artagon/artagon-containers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
