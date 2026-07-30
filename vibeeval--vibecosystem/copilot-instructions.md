## vibecosystem

> - `agent-assignment-matrix.md` → Hangi task hangi agent'a gider

# Agent Orchestration

## Ilgili Kurallar
- `agent-assignment-matrix.md` → Hangi task hangi agent'a gider
- `qa-loop.md` → Dev-QA dongusu, retry logic, escalation
- `skills/handoff-templates/SKILL.md` → Agent arasi mesaj sablonlari

## Available Agents

Located in `~/.claude/agents/`:

| Agent | Purpose | When to Use |
|-------|---------|-------------|
| planner | Implementation planning | Complex features, refactoring |
| architect | System design | Architectural decisions |
| tdd-guide | Test-driven development | New features, bug fixes |
| code-reviewer | Code review | After writing code |
| security-reviewer | Security analysis | Before commits |
| build-error-resolver | Fix build errors | When build fails |
| e2e-runner | E2E testing | Critical user flows |
| refactor-cleaner | Dead code cleanup | Code maintenance |
| doc-updater | Documentation | Updating docs |
| self-learner | Hatalardan ogrenim | Her hata sonrasi |
| verifier | Son quality gate | "Bitti" demeden once |
| janitor | Tech debt & code hygiene | Codebase temizlik, dead code, file size |
| migrator | Dependency upgrade & migration | CVE scan, breaking change, rollback planning |
| compass | Session context recovery | Nerede kaldik, decision log, thread tracking |
| shipper | Release & deploy lifecycle | Pre-deploy checklist, changelog, smoke test |
| catalyst | Scaffold & boilerplate | Pattern scan, tutarli kod uretimi |
| coroner | Post-mortem & pattern propagation | Bug fix sonrasi ayni hatayi baska yerde bul |
| mocksmith | Test data & fixture | Type'dan mock data, edge case, fixture library |
| replay | Bug reproduction & flaky test | %100 reproduce adimlari, flaky test analizi |

## Immediate Agent Usage

No user prompt needed:
1. Complex feature requests - Use **planner** agent
2. Code just written/modified - Use **code-reviewer** agent
3. Bug fix or new feature - Use **tdd-guide** agent
4. Architectural decision - Use **architect** agent
5. Hata yapildiginda - Use **self-learner** agent
6. Is tamamlandiginda - Use **verifier** agent
7. Tech debt/cleanup - Use **janitor** agent
8. Dependency upgrade - Use **migrator** agent
9. Session baslangici/context - Use **compass** agent
10. Release/deploy - Use **shipper** agent
11. Bug fix sonrasi propagation - Use **coroner** agent
12. Test data lazim - Use **mocksmith** agent
13. Bug reproduce edilemiyor - Use **replay** agent

## Dev-QA Loop (ZORUNLU)

Her task implement edildikten sonra:
1. Developer agent implement eder
2. @code-reviewer + @verifier QA yapar
3. PASS → sonraki task | FAIL → developer'a feedback, retry (max 3)
4. 3x FAIL → escalation (reassign, parcala, ertele)
Detay: `qa-loop.md`

## Parallel Task Execution

ALWAYS use parallel Task execution for independent operations.
Bagimsiz task'lari farkli agent'lara AYNI ANDA ver.

## Multi-Perspective Analysis

For complex problems, use split role sub-agents:
- Factual reviewer
- Senior engineer
- Security expert
- Consistency reviewer
- Redundancy checker

---
> Source: [vibeeval/vibecosystem](https://github.com/vibeeval/vibecosystem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
