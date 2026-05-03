## claude-code-agents

> You're the orchestrator. Spawn subagents via `Task()` for focused work.

# CLAUDE.md

You're the orchestrator. Spawn subagents via `Task()` for focused work.

> **Plugin mode:** When installed as a plugin, agents and skills are available natively.
> Use `/full-audit`, `/pre-commit`, `/pre-deploy`, `/new-feature`, `/bug-fix`, `/release-prep` skills.

## Agents (24 Total)

### Audit Agents (11) - NON-OVERLAPPING SCOPE
| Type | Prompt | Does |
|------|--------|------|
| `bug-auditor` | `agents/bug-auditor.md` | Runtime bugs (NOT security) |
| `code-auditor` | `agents/code-auditor.md` | Code quality (NOT security/bugs) |
| `security-auditor` | `agents/security-auditor.md` | ALL security (single authority) |
| `doc-auditor` | `agents/doc-auditor.md` | Documentation |
| `infra-auditor` | `agents/infra-auditor.md` | Config/infra |
| `ui-auditor` | `agents/ui-auditor.md` | UI/UX & a11y |
| `db-auditor` | `agents/db-auditor.md` | Database & queries |
| `perf-auditor` | `agents/perf-auditor.md` | Performance |
| `dep-auditor` | `agents/dep-auditor.md` | Dependencies |
| `seo-auditor` | `agents/seo-auditor.md` | SEO & meta tags |
| `api-tester` | `agents/api-tester.md` | API validation |

### Fix/Implement Agents (4)
| Type | Prompt | Does |
|------|--------|------|
| `fix-planner` | `agents/fix-planner.md` | Prioritize fixes |
| `code-fixer` | `agents/code-fixer.md` | Implement fixes |
| `test-runner` | `agents/test-runner.md` | Run tests |
| `test-writer` | `agents/test-writer.md` | Generate tests |

### Browser QA Agents (4)
| Type | Prompt | Does |
|------|--------|------|
| `browser-qa-agent` | `agents/browser-qa-agent.md` | Chrome UI testing |
| `fullstack-qa-orchestrator` | `agents/fullstack-qa-orchestrator.md` | Find-fix-verify loop |
| `console-monitor` | `agents/console-monitor.md` | Real-time console |
| `visual-diff` | `agents/visual-diff.md` | Screenshot comparison |

### Deploy Agents (2)
| Type | Prompt | Does |
|------|--------|------|
| `deploy-checker` | `agents/deploy-checker.md` | Pre-deploy validation |
| `env-validator` | `agents/env-validator.md` | Environment config |

### Utility Agents (2)
| Type | Prompt | Does |
|------|--------|------|
| `pr-writer` | `agents/pr-writer.md` | PR descriptions |
| `seed-generator` | `agents/seed-generator.md` | Test data |

### Supervisors (1)
| Type | Prompt | Does |
|------|--------|------|
| `architect-reviewer` | `agents/architect-reviewer.md` | Final approval |

## Spawning

**Single:**
```
Task(subagent_type="bug-auditor", prompt="...", description="Security")
```

**Parallel (same response):**
```
Task(subagent_type="bug-auditor", prompt="...", description="Security")
Task(subagent_type="code-auditor", prompt="...", description="Code")
Task(subagent_type="security-auditor", prompt="...", description="OWASP")
Task(subagent_type="db-auditor", prompt="...", description="Database")
```

**Sequential:** Wait for result, then next Task.

## Workflows / Skills

Available as slash commands when installed as a plugin, or see `workflows/` for details.

| Workflow | Purpose | Agents |
|----------|---------|--------|
| `pre-commit` | Before committing | code-auditor, test-runner |
| `pre-deploy` | Before deployment | deploy-checker, env-validator, dep-auditor |
| `full-audit` | Complete audit | All 11 auditors → fix-planner |
| `new-feature` | Build feature | test-writer, code-fixer, test-runner, browser-qa |
| `bug-fix` | Fix bugs | test-writer, code-fixer, test-runner |
| `release-prep` | Release prep | full-audit, fixes, deploy-checker, pr-writer |

**Quick commands:**
- **Full audit:** Spawn all auditors parallel → fix-planner
- **Fix cycle:** fix-planner → code-fixer → test-runner → architect-reviewer
- **Quick check:** Just bug-auditor
- **Browser QA:** browser-qa-agent → fix-planner → code-fixer → verify

## Outputs

All go to `.claude/audits/`:

| File | Source |
|------|--------|
| `AUDIT_SECURITY.md` | security-auditor (SINGLE security authority) |
| `AUDIT_BUGS.md` | bug-auditor (runtime bugs only) |
| `AUDIT_CODE.md` | code-auditor (quality only) |
| `AUDIT_DOCS.md` | doc-auditor |
| `AUDIT_INFRA.md` | infra-auditor |
| `AUDIT_UI_UX.md` | ui-auditor |
| `AUDIT_DB.md` | db-auditor |
| `AUDIT_PERF.md` | perf-auditor |
| `AUDIT_DEPS.md` | dep-auditor |
| `AUDIT_SEO.md` | seo-auditor |
| `API_TEST_REPORT.md` | api-tester |
| `DEPLOY_CHECK.md` | deploy-checker |
| `ENV_REPORT.md` | env-validator |
| `FIXES.md` | fix-planner (with deduplication) |
| `TEST_REPORT.md` | test-runner |
| `AUDIT_BROWSER_QA.md` | browser-qa-agent |
| `QA_SESSION_LOG.md` | fullstack-qa-orchestrator |
| `QA_COMPLETE.md` | fullstack-qa-orchestrator |
| `EXECUTION_LOG.md` | All agents (status tracking) |

## Agent Status Protocol

Every agent output MUST start with a status block:
```yaml
---
agent: [agent-name]
status: COMPLETE | PARTIAL | SKIPPED | ERROR
timestamp: [ISO timestamp]
duration: [seconds]
findings: [count]
errors: []
skipped_checks: []
---
```

---

## Project

<!-- Edit this -->

**Stack:**
**Critical paths:**
**Don't touch:** `.env`

---
> Source: [undeadlist/claude-code-agents](https://github.com/undeadlist/claude-code-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
