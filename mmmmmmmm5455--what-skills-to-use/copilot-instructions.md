## what-skills-to-use

> Automated skill gap analysis, fusion assessment, discovery, and installation. Scans local skills for capability coverage, merges/extend existing skills before installing new ones, searches skillhub/clawhub/web only when fusion is infeasible, and integrates into any PRD or workflow.


# what-skills-to-use — 智能技能匹配與自動安裝

Automated pipeline: **Scan → Analyze → Fuse → Search → Install → Log → Continue**.

---

## 1. Trigger Conditions

### 1.0 Language Detection (run first)

Before any analysis, detect the user's input language and respond in the same language:

```
Input script detection:
  - Contains CJK characters ([一-鿿぀-ゟ゠-ヿ]) → respond in Chinese
  - ASCII-only OR contains Latin accented chars → respond in English
  - Mixed: prioritize the majority script

Response language:
  - ZH: All section headers, table labels, recommendations in Chinese
  - EN: All section headers, table labels, recommendations in English
```

### 1.1 Auto-trigger (implicit)

When user says any of the following AND the task implies tool/skill dependency:

**English patterns (any of these substrings in user message):**
- Task intent: "I want to", "I need a", "I need to", "build me a", "build a", "create a", "set up a", "set up an", "implement", "help me set up", "help me build", "can you help me", "could you help me", "would you"
- Skill inquiry: "what skills", "any skill for", "is there a skill", "find me a skill", "what skill covers", "skills for", "how to build", "do you have a skill", "do I have a skill", "is there a tool"
- Install intent: "install skill", "install a skill", "look for a skill", "need a skill for", "missing a skill"

**Chinese patterns (any of these substrings in user message):**
- Task intent: "我要", "做一個", "實現", "幫我搭建", "我需要一個", "幫我做", "搭建一個", "建立一個"
- Skill inquiry: "有什麼技能", "哪些技能", "檢查技能", "技能庫", "有沒有技能"
- Install intent: "安裝技能", "找技能", "裝一個技能"

### 1.2 Quick mode (lightweight — no full pipeline)

Triggered by:
- `/has-skill <name>` — single lookup: "is X installed?"
- `/has-skill <capability>` — "can I do X with existing skills?"
- User: "do I have X skill?" / "can I do X?" / "我有沒有 X 技能" / "能不能做 X"

Quick mode behavior:
1. Check capability_index.json for keyword match (skip full scan)
2. If index stale (>7 days) or absent, do a fast targeted grep in `.claude/skills/*/SKILL.md`
3. Return: YES (skill names) / PARTIAL (needs fusion) / NO (would need install)
4. End. Do NOT proceed to Phases 3-5 unless user says "go ahead" or "install it"

### 1.3 Manual trigger (explicit)

- `/tools` — run full skill audit (Phases 1-2, output coverage report)
- `/skills-check` — run gap analysis only (Phase 1-2, no install)
- `/skills-check --full` — run all 5 phases, including fusion assessment
- "檢查技能庫" / "what skills do I have" → Phase 1 only
- "檢查技能庫能不能做 X" / "what skills do I have for X" → Phases 1-2

### 1.4 Programmatic trigger

Other skills or workflows (e.g. gsd-workflows plan-phase) can invoke this skill by emitting:
```
<invoke skill="what-skills-to-use" args="--query=<task description> --auto-approve=false" />
```

---

## 2. Phase 1: Local Skill Scan & Capability Indexing

### 2.1 Scan sources (in order)

| Priority | Source | Path / Command | Incremental? |
|----------|--------|---------------|-------------|
| 1 | Active skills | `ls -d .claude/skills/*/` → read each `SKILL.md` | Yes — skip if mtime < last index time |
| 2 | Skills archive | `ls .claude/skills-archive/` (if present) | Yes — archive rarely changes |
| 3 | External catalog | `.claude/repos/Product-Manager-Skills/catalog/skills-index.yaml` | Yes |
| 4 | Plugin skills | `.claude/skills/*/skills/*/SKILL.md` (nested, e.g. gstack-*, ccgs-*, pua) | Yes |
| 5 | Installed MCP servers | Check `claude mcp list` or `.claude/mcp.json` | No — always fresh |

**Incremental scan logic:**
1. Read `capability_index.json` → check `generated` timestamp
2. If index exists and age < 7 days: only re-scan directories where `ls -lt` shows changes newer than index
3. If index missing or age >= 7 days: full re-scan
4. Always do a quick existence check on previously-indexed skills; remove any that are gone

### 2.2 Extraction per skill

For each `SKILL.md` found, extract:
```yaml
name: <directory name or frontmatter name>
description: <frontmatter description or first paragraph>
trigger_keywords: <from frontmatter or "觸發方式" section>
capabilities: <inferred from section headers, code blocks, CLI commands>
inputs: <what does it consume>
outputs: <what does it produce>
extensibility: <HIGH|MEDIUM|LOW — can it be extended to cover adjacent tasks?>
```

### 2.3 Build capability keyword index

Aggregate all extracted `capabilities` into a flat keyword-to-skill mapping:

```
keyword                → [skill_name, ...]
"git commit"           → ["caveman-commit", "guard", "using-git-worktrees"]
"code review"          → ["caveman-review", "gstack-review", "ccgs-code-review", "code-audit"]
"PRD"                  → ["prd-writer", "gsd-workflows", "pm-prd-development"]
"Three.js"             → ["ccgs-prototype", "ccgs-art-bible"]
"Obsidian"             → ["obsidian-process", "obsidian-markdown", "vault-search", "obsidian-cli", "obsidian-bases", "project-archive"]
"testing"              → ["skill-tester", "gstack-qa", "ccgs-qa-plan"]
"workflow"             → ["gsd-workflows", "workflow-orchestrator", "skill-router"]
"browser"              → ["agent-browser", "browser-automation", "scrapling-skill"]
"skill management"     → ["skill-manager", "skill-discovery", "find-skills", "what-skills-to-use"]
...
```

Save index to `.claude/skills/what-skills-to-use/capability_index.json` for fast lookup.

### 2.4 Output artifact

After scan, output a structured summary:

```
## Active Skills: <N>
## Archived Skills: <M>
## External Catalog Entries: <P>
## Capability Keywords: <K>

Top-level capability clusters:
  - git/*        (12 skills)
  - review/*     (8 skills)
  - design/*     (7 skills)
  - testing/*    (6 skills)
  - workflow/*   (5 skills)
  - obsidian/*   (4 skills)
  - ...
```

### 2.5 Error handling during scan

| Failure | Fallback |
|---------|----------|
| `.claude/skills/` does not exist | Report "No skills directory found — fresh install?" and skip to Phase 4 (search-only) |
| `SKILL.md` is empty or unreadable | Skip that skill, flag with `[UNREADABLE]` in output |
| `capability_index.json` is corrupt (invalid JSON) | Delete it, do full re-scan |
| External catalog path missing | Skip source 3, continue with 1-2-4-5 |
| `claude mcp list` command not available | Skip source 5, note in output |
| 0 skills found across all sources | Report "No skills installed. Use /tools to search for skills." and offer Phase 4 directly |
| Archive directory (>500 skills) scan would take >30s | Sample top 100 most recently modified; note "archive sampled" in output |

---

## 3. Phase 2: Requirement Decomposition & Gap Analysis

### 3.1 Decompose user request into atomic capabilities

Given the user's task description, break it down:

```
User: "我要搭建一個能自動歸檔筆記的 Obsidian 系統"
User (EN): "I want to build an Obsidian system that auto-archives notes"

Atomic capabilities needed:
  1. [vault-read]       Read markdown files from Obsidian vault
  2. [vault-write]      Write/modify markdown files in vault
  3. [file-watch]       Watch for file changes (new/modified notes)
  4. [auto-classify]    Classify notes by content (keyword / frontmatter rules)
  5. [move-rename]      Move/rename files based on classification
  6. [yaml-frontmatter] Parse and update YAML frontmatter
  7. [scheduled-task]   Run periodically (cron / Task Scheduler)
  8. [logging]          Log archiving actions
```

### 3.2 Match against capability index

For each atomic capability, check keyword index:

| Capability | Match Status | Matching Skills |
|-----------|-------------|-----------------|
| vault-read | FULL | vault-search (semantic search, sqlite-vec index) ✓ |
| vault-write | FULL | obsidian-markdown (frontmatter, wikilinks) + project-archive ✓ |
| file-watch | PARTIAL | auto-recovery (has cron polling pattern, not file-change) |
| auto-classify | PARTIAL | vault-search (has index + content access, no routing rules) |
| move-rename | FULL | project-archive (moves files to projects/<name>/) ✓ |
| yaml-frontmatter | FULL | obsidian-markdown/references/PROPERTIES.md ✓ |
| scheduled-task | PARTIAL | auto-recovery (cron infra), ov-cron (scheduling patterns) |
| logging | FULL | project-archive (structured logging) + auto-recovery pattern ✓ |

### 3.3 Classification rules

- **FULL**: An existing skill directly provides this capability. Mark as COVERED.
- **PARTIAL**: An existing skill provides a subset or adjacent capability. Flag for FUSION assessment (Phase 3).
- **NONE**: No existing skill provides this. Flag for SEARCH (Phase 4).

---

## 4. Phase 3: Fusion Feasibility Assessment (THE KEY DIFFERENTIATOR)

**This phase runs BEFORE any external search. Do not skip.**

### 4.1 For each PARTIAL-match skill, assess:

```
Skill: <name>
Current capability: <what it does>
Missing for this task: <what's needed>
Extensibility score: HIGH | MEDIUM | LOW

Fusion feasibility questions:
  Q1. Can this skill's scope be expanded by adding a sub-function?
      → Check if SKILL.md has an "Advanced" or "Extending" section.
  Q2. Can this skill's configuration be adjusted to cover the new case?
      → Check for config files, templates, JSON schemas.
  Q3. Does this skill have a modular structure that allows plugins?
      → Check for /skills/<name>/modules/ or extensions/ directories.
  Q4. Is the missing capability adjacent enough that a few lines of bash/python
      would bridge the gap?
      → Estimate lines of code needed.

If (Q1 OR Q2 OR Q3) AND estimated effort < 50 lines:
    → FUSION RECOMMENDED. Propose a concrete modification plan.
Else if Q4 AND estimated effort < 30 lines:
    → FUSION RECOMMENDED. Propose a thin adapter script.
Else:
    → FUSION NOT RECOMMENDED. Proceed to Phase 4 (external search).
```

### 4.2 Fusion decision table

| Missing Capability | Best Fusion Target | Effort | Action |
|-------------------|-------------------|--------|--------|
| file-watch | auto-recovery | 25 lines | Extend with generic `watch_directory()` using existing cron polling pattern |
| auto-classify | vault-search | 35 lines | Add `classify.py` using existing sqlite-vec index + frontmatter rules |
| scheduled-task | auto-recovery or Task Scheduler | 15 lines | Extract cron template; or use Windows Task Scheduler XML export |

### 4.3 Present fusion plan to user

Before modifying any file, present:

```
## Fusion Plan

I can cover 2 of your 3 missing capabilities by extending existing skills,
avoiding the need to install 2 new skills:

**Fusion 1: Extend `auto-recovery` with generic file-watch**
  - Add `watch_directory(path, callback)` function (~15 lines)
  - Existing cron infra reuses the scheduling logic directly
  - File: .claude/skills/auto-recovery/SKILL.md (+1 section)
  - File: .claude/skills/auto-recovery/file-watcher.sh (new, ~20 lines)

**Fusion 2: Extend `vault-search` with auto-classify**
  - Add `scripts/classify.py` (~35 lines) using existing frontmatter queries + keyword rules
  - Uses sqlite-vec index already built by vault-search for content awareness
  - File: .claude/skills/vault-search/scripts/classify.py (new)
  - File: .claude/skills/vault-search/SKILL.md (+1 section, ~15 lines)

**Still need to install:**
  - None — all capabilities now covered.

Proceed with fusion? [Y/n]
```

### 4.4 Execute fusion (with user approval)

If approved:
1. Read the target skill's full SKILL.md and associated files
2. Draft the minimal changes (follow caveman principle: minimal, no abstraction bloat)
3. Show the diff to user
4. Apply changes
5. Re-run capability check to verify coverage
6. Log fusion to `.claude/logs/skill-install-log.md`

---

## 5. Phase 4: External Search & Install (only if fusion infeasible)

### 5.1 Search order

Only for capabilities still marked NONE after Phase 3:

```
1. skillhub search <query>          # Chinese-optimized, preferred for CN users
2. clawhub search <query>           # Fallback
3. WebSearch: "<capability> Claude Code skill github"
4. GitHub: gh search repos "claude skill <capability>"
5. npm: npm search "claude-code-<capability>"
6. Anthropic skills registry (if accessible)
```

### 5.2 Selection criteria (sorted by priority)

| # | Criterion | Weight |
|---|----------|--------|
| 1 | Matches the EXACT missing capability | Required |
| 2 | Open source (MIT, Apache-2.0, or similar) | Required |
| 3 | Compatible with Claude Code (SKILL.md format or MCP server) | Required |
| 4 | Maintained (commit in last 90 days) | High |
| 5 | Installable via `npx skills add`, `git clone`, or `npm install` | High |
| 6 | Minimal dependencies (prefer stdlib-only) | Medium |
| 7 | Caveman-compatible (no framework bloat) | Medium |

### 5.3 Present options to user

If multiple matches:
```
## Search Results for "<capability>"

| # | Name | Source | Stars | Last Commit | Install | Notes |
|---|------|--------|-------|-------------|---------|-------|
| 1 | skill-name-a | skillhub | 120 | 2026-04-15 | `skillhub install ...` | Best match, active |
| 2 | skill-name-b | clawhub | 45 | 2026-03-20 | `clawhub install ...` | Partial match |
| 3 | github.com/x/y | GitHub | 230 | 2026-04-28 | `git clone ...` | Full match, needs manual setup |

Recommendation: #1 (reason). Which to install?
```

If no results:
```
## No Existing Skill Found for "<capability>"

Would you like me to create a custom caveman-style skill for this?
Estimated effort: <N> lines, <language>. I can draft it now.
```

### 5.4 Install selected skill

```bash
# From skillhub
skillhub install <skill-name>

# From clawhub
clawhub install <skill-name>

# From git
cd .claude/skills && git clone <repo-url> <skill-name>

# From npm
npm install -g <package-name>
```

After install:
1. Verify `SKILL.md` exists at target path
2. Re-run Phase 1 capability scan
3. Confirm the missing capability is now COVERED
4. Add to capability index
5. Log to `.claude/logs/skill-install-log.md`

---

## 6. Phase 5: Integration & Continuation

### 6.1 Update registry

After fusion or install, update:
- `.claude/skills/what-skills-to-use/capability_index.json` (keyword index)
- `.claude/logs/skill-install-log.md` (audit trail)

### 6.2 Return to original workflow

```
## Skill Resolution Complete

To fulfill your request "<original task>", I have:
  - Used existing: <list of skills already covering the need>
  - Fused/Extended: <list of skills modified>
  - Installed: <list of new skills installed>

All required capabilities are now available. Continuing with the original task...
```

### 6.3 Integration with gsd-workflows

When invoked from a gsd-workflow (plan-phase, new-project, new-milestone):
- This skill runs as a pre-step before the plan is finalized
- Missing skills are resolved before the plan asks the user to proceed
- The plan's "Dependencies" section is auto-populated with skill requirements

---

## 7. Logging & Safety

### 7.1 Log format (`.claude/logs/skill-install-log.md`)

```markdown
## <timestamp> — <action_type>

- **Trigger**: "<user request or workflow step>"
- **Action**: FUSION | INSTALL | SKIP
- **Skill(s) affected**: <name(s)>
- **Capability gap**: <description>
- **Resolution**: <what was done>
- **Files changed**: <list>
- **Verification**: <PASS | FAIL — re-scan result>
```

### 7.2 Safety rules

1. **Never install without presenting the plan first** (unless `--auto-approve=true` is set explicitly by the user)
2. **Never modify a skill without showing the diff** — user must approve before fusion is applied
3. **Never delete or overwrite existing skill files** during fusion — only add new sections or new companion files
4. **Never execute `curl | bash` patterns** — all installs use `git clone`, `skillhub install`, or `npm install`
5. **Never bypass the fusion phase** — always check if existing skills can be extended before searching externally
6. **If `skillhub` or `clawhub` is unavailable**, fall back gracefully and report which sources were tried

### 7.3 Auto-approve mode

User can set in request: `--auto-approve=true` or "自動安裝" / "不用問我直接裝"

In auto-approve mode:
- FUSION still requires confirmation (modifies existing code)
- INSTALL of well-known skills (skillhub/clawhub) can proceed without confirmation
- INSTALL from git/npm still requires confirmation (external source risk)

### 7.4 Obsidian vault logging

After any FUSION or INSTALL action, optionally write a structured note to the Obsidian vault:

**Vault path:** `C:/Users/qwqwh/ObsidianVault` (configurable via `OBSIDIAN_VAULT_PATH` env var)

**Note location:** `{vault}/_system/skill-events/{skill-name}-{date}-{action}.md`

**Note template:**
```markdown
---
type: skill-event
skill: <skill-name>
action: FUSION | INSTALL | UPGRADE
version: <old> → <new>
date: <ISO8601>
tags:
  - skill-management
  - what-skills-to-use
---

# <skill-name> — <action>

## Trigger
<original user request>

## Changes
<bullet list of what changed>

## Files Modified
<file paths>

## Capability Impact
<before/after coverage comparison>
```

To enable: set `WHAT_SKILLS_LOG_TO_OBSIDIAN=true` or pass `--log-to-obsidian`.
Default: off (to avoid vault writes unless user explicitly wants them).

---

## 8. Usage Examples

### Example 1: Simple skill check

```
User: /skills-check
→ Run Phase 1 + Phase 2 only. Output capability coverage report. No install.
```

### Example 2: New project with auto-resolve

```
User: 我要搭建一個能自動歸檔筆記的 Obsidian 系統
→ Full pipeline: Scan → Decompose → Match → Fuse → (Search if needed) → Log → Continue
```

### Example 3: Workflow integration

```
gsd-workflows plan-phase invokes:
  what-skills-to-use --query="<PRD description>" --mode=pre-flight
→ Returns capability coverage to the plan. Plan includes "Skills to install: [...]"
```

### Example 4: Gap-only mode

```
User: 檢查我的技能庫能不能做瀏覽器自動化測試
→ Run Phase 1 + Phase 2 scoped to "browser automation testing".
   Report coverage % and list gaps.
```

---

## 9. Self-Maintenance

### 9.1 Capability index refresh

The `capability_index.json` should be refreshed:
- After any skill installation or removal
- At session start if older than 7 days
- On manual `/tools` invocation

### 9.2 Stale skill detection

During Phase 1 scan, flag skills where:
- `SKILL.md` references files that no longer exist
- Dependencies are not installed (check `command -v <tool>`)
- Last modified > 180 days AND never used in logs

### 9.3 Conflict detection

When fusion is proposed, check:
- Does the proposed change overlap with another skill's scope?
- Would the fusion create two skills that do the same thing?
- If yes: recommend merging the two skills instead of extending one.

### 9.4 Self-audit checklist (run on every edit to this SKILL.md)

After any modification to `what-skills-to-use` itself:

| # | Check | Command / Method |
|---|-------|-----------------|
| 1 | All example skill references exist | `ls .claude/skills/<name>/SKILL.md` for each skill in examples |
| 2 | All paths are valid for current OS | `ls` each file path referenced in code blocks |
| 3 | Trigger keywords have EN + ZH coverage | Count Chinese vs English entries in frontmatter |
| 4 | No dead links in Appendix A | Verify each skill in capability_index.json example exists |
| 5 | Version matches changelog | Compare `version:` field with latest changelog entry |
| 6 | Phases 1-5 are sequential and complete | Read §2-§6 headers: Scan → Decompose → Fuse → Search → Integrate |
| 7 | All code blocks are syntactically valid | Bash: `bash -n`, Python: `python -c "compile(...)"`, JSON: `jq .` |

---

## Appendix A: Capability Index Schema

```json
{
  "generated": "2026-05-01T00:00:00Z",
  "skill_count": 99,
  "archive_count": 553,
  "keywords": {
    "obsidian": {
      "skills": ["obsidian-process", "obsidian-markdown", "vault-search", "obsidian-cli", "obsidian-bases", "project-archive"],
      "capabilities": ["read", "write", "frontmatter", "wikilinks", "search", "index", "classify", "archive", "process-mgmt"],
      "coverage": "HIGH"
    },
    "git": {
      "skills": ["caveman-commit", "guard", "using-git-worktrees", "ship"],
      "capabilities": ["commit", "review", "branch", "push", "hook", "worktree"],
      "coverage": "HIGH"
    }
  }
}
```

## Appendix B: Fusion Feasibility Quick Reference

| Signal | Fusion Likely | Fusion Unlikely |
|--------|-------------|-----------------|
| Skill has modular structure | ✓ | |
| Skill uses config files / templates | ✓ | |
| Missing capability is a thin wrapper | ✓ | |
| Skill is a single-file script | ✓ | |
| Skill has hardcoded domain logic | | ✗ |
| Missing capability is fundamentally different domain | | ✗ |
| Skill is proprietary / closed-source | | ✗ |
| Estimated effort > 100 lines | | ✗ |

---
> Source: [mmmmmmmm5455/what-skills-to-use](https://github.com/mmmmmmmm5455/what-skills-to-use) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
