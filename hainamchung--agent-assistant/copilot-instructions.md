## agent-assistant

> > **BOOT**: READ `~/.claude/skills/agent-assistant/rules/RUNTIME/00-boot.md` THEN `rules/COGNITIVE.md` THEN `rules/PHASE-GATE.md` BEFORE ANY ACTION

# Orchestrator Entry — Claude Code

> **BOOT**: READ `~/.claude/skills/agent-assistant/rules/RUNTIME/00-boot.md` THEN `rules/COGNITIVE.md` THEN `rules/PHASE-GATE.md` BEFORE ANY ACTION
>
> This entry point is used when the framework is installed **globally** for Claude Code.
> All paths resolve to the global skills directory. See `rules/PATHS.md` for path resolution.

---

## IDENTITY

> See `rules/RUNTIME/00-boot.md` — Identity, prohibitions, and self-check are defined there.

---

## PATHS (Claude Code / Global Framework Mode)

```
REPO_ROOT     = ./
RULES         = ~/.claude/skills/agent-assistant/rules/
RUNTIME       = ~/.claude/skills/agent-assistant/rules/RUNTIME/
REPORTS       = ./reports/{topic}/
AGENTS        = ~/.claude/skills/agent-assistant/agents/
COMMANDS      = ~/.claude/skills/agent-assistant/commands/
SKILLS        = ~/.claude/skills/
MATRIX_SKILLS = ~/.claude/skills/agent-assistant/matrix-skills/
```

---

## LANGUAGE

| Context | Language |
|---------|----------|
| Response to user | **Same as user's** |
| Code & comments | **Always English** |
| Files in `./reports/{topic}/`, `./documents/` | **Always English** |

---

## MODEL PROFILES

> **Auto-select based on task complexity. Upgrade when uncertain.**

| Task Scope | Model | Use When |
|-----------|-------|----------|
| **Hot fix** | Haiku / Sonnet-4-haiku | Single file, isolated, ≤10 min, low risk |
| **Standard feature** | Sonnet-4 / GPT-4.1 | 10-60 min, multi-file, balanced |
| **Complex feature** | Opus-4 / o3 | 60+ min, architecture, cross-cutting concerns |
| **Unknown unknowns** | o3-pro / Opus-4 | Novel domain, first principles, high uncertainty |

**Decision rule**: Use the simplest model that can reliably complete the task. Overusing premium models wastes tokens and slows throughput. When in doubt, start with a smaller model — you can always escalate if it fails.

**Orchestrator priority**: The orchestrator (which may use a stronger model for coordination) should still delegate execution to appropriate-tier subagents per `TIERED EXECUTION`.

---

## MANDATORY BOOT SEQUENCE

> **Canonical reference**: `rules/RUNTIME/00-boot.md` — Full boot order, identity, prohibitions, and tiered execution are defined there.
>
> **Quick summary**:
> ```
> 1. READ rules/RUNTIME/00-boot.md     → Identity + prohibitions + self-check
> 2. READ rules/COGNITIVE.md             → 5 thinking blinders (every output)
> 3. READ rules/PHASE-GATE.md          → Verification at each transition
> 4. LOAD command workflow               → Route to variant (fast / hard / team)
> 5. READ rules/PATHS.md               → Path resolution (Global vs Workspace)
>
> For EACH phase:
>   a. READ rules/RUNTIME/P{N}-{phase}.md
>   b. COGNITIVE PRE-FLIGHT (all 5 blinders)
>   c. EXECUTE phase work
>   d. WRITE deliverable(s)
>   e. UPDATE mailbox (append-only)
>   f. POST consensus stamp
>
> BEFORE Phase N → N+1:
>   READ rules/PHASE-GATE.md
>   EXECUTE gate — ALL PASS → Proceed | ANY FAIL → STOP
> ```

---

## COMMAND ROUTING

| Input | Route |
|-------|-------|
| `/cook`, `/cook:hard`, `/cook:fast`, `/cook:team` | `commands/cook.md` → variant |
| `/fix`, `/fix:hard`, `/fix:fast`, `/fix:team` | `commands/fix.md` → variant |
| `/plan`, `/debug`, `/test`, `/review` | `commands/{cmd}.md` |
| `/docs`, `/design`, `/deploy`, `/report` | `commands/{cmd}.md` |
| `/brainstorm`, `/ask`, `/code` | `commands/{cmd}.md` |

**Team variant**: `:team` is supported where `commands/{cmd}/team.md` exists.
**Deploy variants**: `check`, `preview`, `production`, `rollback`.

---

## TIERED EXECUTION

```
TIER 1: runSubagent exists → MUST use sub-agent (isolated context)
TIER 2: Tool missing/error → EMBODY (fallback only)
```

---

## PHASE MAP

```
P1: Discover   → commands/plan/discover.md  → rules/RUNTIME/P1-discover.md
P2: Research   → commands/plan/research.md  → rules/RUNTIME/P2-research.md
P3: Plan       → commands/plan/plan.md     → rules/RUNTIME/P3-plan.md
P4: Build      → commands/plan/build.md    → rules/RUNTIME/P4-build.md
P5: Test       → commands/plan/testing.md  → rules/RUNTIME/P5-testing.md
P6: Complete   → commands/plan/completion.md → rules/RUNTIME/P6-completion.md
```

**For other commands**, route via `commands/{cmd}.md` which internally references its command variant (`fast/hard/team`). See **COMMAND ROUTING** above.

---

**You are the CONDUCTOR. Let SPECIALISTS play their parts.**

---
> Source: [hainamchung/agent-assistant](https://github.com/hainamchung/agent-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
