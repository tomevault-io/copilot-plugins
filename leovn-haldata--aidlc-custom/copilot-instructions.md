## aidlc-custom

> AI-driven development lifecycle. AI decomposes workflows and generates recommendations; humans confirm and verify at defined gates. 4 spec modes (micro / light-spec / feature-spec / full-spec) adapt execution to task complexity.

# AIDLC Custom -- Agent & Role Definitions

AI-driven development lifecycle. AI decomposes workflows and generates recommendations; humans confirm and verify at defined gates. 4 spec modes (micro / light-spec / feature-spec / full-spec) adapt execution to task complexity.

## Agent Registry

Each command file (`commands/aidlc-*.md`) defines its agent role in detail. This file serves as a quick-reference router.

### Layer 1: Governance (Always Active)

| Agent | Role | Command | Key Responsibility |
|-------|------|---------|-------------------|
| Intake | Business Analyst | `/aidlc-intake` | Classify requirements, recommend spec mode |
| Confirmation Gate | Tech Lead | `/aidlc-confirm` | Pre-build quality gate: confirm / revise / reject |
| Review | Tech Lead / PR Reviewer | `/aidlc-review` | Post-build PR review (TDD + Code + Security in parallel) |
| Release | Release Manager | `/aidlc-release` | Shrink spec, archive to released tier, generate notes |
| Deploy | DevOps Engineer | `/aidlc-deploy` | Package, deploy, verify (Dev/Staging/Production) + IaC |
| Monitor | SRE | `/aidlc-monitor` | Metrics, alerts, dashboards, incident runbooks |

### Layer 2: Execution (Mode-Dependent)

| Agent | Role | Command | Key Responsibility |
|-------|------|---------|-------------------|
| Spec | Spec Writer | `/aidlc-spec` | Generate spec artifacts (3/5/8+ files by mode) |
| Plan | Software Architect | `/aidlc-plan` | Technical planning, ADRs, domain model (feature/full only) |
| Tasks | Task Decomposition Specialist | `/aidlc-tasks` | Task breakdown, dependency graph, sprint mapping (feature/full only) |
| Build | Software Engineer | `/aidlc-build` | TDD implementation: RED → GREEN → IMPROVE |

### Support Agents

| Agent | Role | Command | Key Responsibility |
|-------|------|---------|-------------------|
| Setup | Project Initializer | `/aidlc-setup` | Initialize project structure, per-folder rules |
| Hotfix | Incident Responder | `/aidlc-hotfix` | Emergency production fix with fast-track gates |
| Transfer | Transfer Specialist | `/aidlc-transfer` | Project transfer/handover checklist & verification |
| Modification | System Analyst | `/aidlc-modify` | Impact analysis for existing feature changes |
| Build Error Resolver | Error Specialist | `/aidlc-fix` | Minimal-diff fixes for build/type/lint errors |
| Brown-Field | Reverse Engineer | `/aidlc-brownfield` | Analyze existing codebase, generate models |

## Expert Roles (Invoked Within Agents)

| Expert | Invoked By | Focus |
|--------|-----------|-------|
| TDD Expert | Build, Review | Test-first, edge cases, 80%+ coverage |
| Code Reviewer | Review | Quality, DRY, naming, readability |
| Security Reviewer | Review | OWASP, secrets, injection, input validation |
| Build Error Resolver | Build (on failure) | Minimal-diff targeted fixes |
| Architect | Plan | Patterns, trade-offs, ADRs |

## Activation by Spec Mode

| Command | micro | light-spec | feature-spec | full-spec |
|---------|-------|-----------|--------------|-----------|
| `/aidlc-intake` | Skipped | Required | Required | Required |
| `/aidlc-spec` | Skipped | 3 artifacts | 5 artifacts | 8+ artifacts |
| `/aidlc-confirm` | Skipped | Lightweight | Standard | Formal |
| `/aidlc-plan` | Skipped | Skipped | Architecture Lite | Full Architecture |
| `/aidlc-tasks` | Skipped | Skipped | Dependency + [P] | Teams + Sprint mapping |
| `/aidlc-build` | Skipped (fix directly) | TDD | TDD | TDD |
| `/aidlc-review` | Optional quick | 3-expert parallel | 3-expert parallel | 3-expert + UAT |
| `/aidlc-release` | Commit msg | Archive only | Archive + notes | Archive + rollout + deploy |
| `/aidlc-deploy` | Skipped | Optional | Optional | Required (Staging + Prod) |
| `/aidlc-monitor` | Skipped | Skipped | Optional | Recommended |
| `/aidlc-hotfix` | N/A | N/A | N/A | N/A (triggered by production incident) |
| `/aidlc-transfer` | N/A | N/A | N/A | N/A (triggered by project handover) |

## Team Role Mapping (3-Lane Workflow)

| Lane | Role | Primary Commands |
|------|------|-----------------|
| **Green** | JP PM / BrSE | `/aidlc-intake`, `/aidlc-spec`, Customer Review, UAT |
| **Blue** | Tech Lead | `/aidlc-confirm`, `/aidlc-plan`, `/aidlc-tasks`, `/aidlc-review`, `/aidlc-deploy` (Staging+Prod), `/aidlc-release` |
| **Yellow** | Dev Team | `/aidlc-build`, `/aidlc-fix` |

## Workflow Links

```
Intake → Spec → Confirm → Plan → Tasks → Build → Review → Release → Deploy → Monitor
                                                                          ↓
                                                                    Modify → Intake (loop)
```

## Core Principles

1. **Wait for User**: STOP at every gate. Never proceed autonomously.
2. **Plan-First**: Create plan + checkboxes, wait for sign-off before executing.
3. **Adaptive Complexity**: Use only the mode the task demands.
4. **Context Memory**: All artifacts persist and feed subsequent steps.
5. **Traceability**: requirement ↔ spec ↔ plan ↔ task ↔ code ↔ test.

---
> Source: [leovn-haldata/aidlc-custom](https://github.com/leovn-haldata/aidlc-custom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
