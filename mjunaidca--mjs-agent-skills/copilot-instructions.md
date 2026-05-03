## mjs-agent-skills

> Use when [specific trigger condition].

# Claude Code Rules

> Operational directives for AI agents working in mjs-skills

---

## What This Repo Is

A personal Agent Skills library. Skills are frozen decisions—not tools, not utilities, not helpers. Each skill encodes judgment about what matters, what fails, and what works.

---

## Before You Do Anything

1. **Read the constitution**: `.specify/memory/constitution.md`
2. **Check existing skills**: `ls .claude/skills/`
3. **Understand the PRD**: `research/h3-skills-master-prd.md` for skill specifications
4. **Reference the spec**: `agentskills-standard/docs/specification.mdx` for Agent Skills format

---

## Task Context

**Your Surface:** You operate on a project level, providing guidance to users and executing development tasks via a defined set of tools.

**Your Success is Measured By:**
- All outputs strictly follow the user intent
- Skills comply with constitution principles and Agent Skills spec
- Prompt History Records (PHRs) created automatically and accurately
- ADR suggestions made intelligently for significant decisions
- All changes are small, testable, and reference code precisely

---

## Core Guarantees (Product Promise)

- Record every user input verbatim in a PHR after every user message
- PHR routing (all under `history/prompts/`):
  - Constitution → `history/prompts/constitution/`
  - Feature-specific → `history/prompts/<feature-name>/`
  - General → `history/prompts/general/`
- ADR suggestions: when an architecturally significant decision is detected, suggest: "📋 Architectural decision detected: <brief>. Document? Run `/sp.adr <title>`." Never auto-create ADRs; require user consent.

---

## Skill Development Rules

### Authoritative Standard

Follow the [Agent Skills Specification](agentskills-standard/docs/specification.mdx). When in doubt, the spec takes precedence. The constitution at `.specify/memory/constitution.md` adds project-specific constraints.

### Required Structure

```
.claude/skills/[skill-name]/
├── SKILL.md           # YAML frontmatter + instructions
└── scripts/
    └── verify.py      # Exit 0 (success) or 1 (failure)
```

### SKILL.md Template

```yaml
---
name: [gerund-form-name]
description: |
  [What it does in one sentence].
  Use when [specific trigger condition].
---

## Quick Start
[Immediate action for the 80% case]

## Instructions
1. [Step with command]
2. [Step with command]
3. `python scripts/verify.py`

## If Verification Fails
1. Run diagnostic: `[specific command]`
2. Check: `[what to look for]`
3. **Stop and report** — do not proceed with downstream steps
```

### verify.py Template

```python
#!/usr/bin/env python3
"""Verify [what this checks]."""
import subprocess
import sys

def main():
    result = subprocess.run([...], capture_output=True)

    if result.returncode == 0:
        print("✓ [Specific success message]")  # Keep under 100 chars
        sys.exit(0)
    else:
        print("✗ [Actionable error]. Run: [diagnostic command]")
        sys.exit(1)

if __name__ == "__main__":
    main()
```

### Rules You Must Follow

#### Naming
- Use gerund form: `deploying-*`, `scaffolding-*`, `building-*`
- Max 64 characters, lowercase + hyphens only
- Exception: Pattern names can override if sharper (e.g., `kafka-exactly-once`)
- Test: Would someone search for this exact name?
- **BANNED**: `utils`, `helpers`, `setup`, `manage`, `handle`

#### Description
- MUST include "Use when [trigger]"
- Max 1024 characters (per Agent Skills spec)
- Use third person ("Deploys..." not "I deploy...")
- If collision possible, add "NOT when [exclusion]"
- **BANNED words**: "manage", "handle", "setup", "utilities", "helpers"

#### Token Discipline (Progressive Disclosure)

| Layer | Content | Budget | When Loaded |
|-------|---------|--------|-------------|
| 1. Metadata | name + description | ~100 tokens | Always (startup) |
| 2. Instructions | SKILL.md body | <5000 tokens | When skill activates |
| 3. Resources | scripts/, references/ | As needed | On demand only |

- SKILL.md: <500 lines, <5000 tokens
- verify.py output: <100 characters
- References: one level deep only

#### MCP Output Discipline
- NEVER inject raw MCP output into context
- ALWAYS filter/summarize before context entry
- Raw payloads stay in subprocess or temp files
- **BANNED patterns**: `print(full_response)`, `context += mcp_output`, `return transcript`

#### Verification
- Every skill MUST have verify.py
- On failure: exit 1 + actionable error + diagnostic command
- On success: exit 0 + minimal confirmation

#### Failure Escalation
- If verify.py fails twice: STOP
- Surface the diagnostic command
- Request human intervention
- Do NOT proceed with downstream steps

### Before Shipping Any Skill

```bash
# 1. Validate with skills-ref
skills-ref validate ./.claude/skills/[skill-name]

# 2. Structure check
ls .claude/skills/[skill-name]/SKILL.md
ls .claude/skills/[skill-name]/scripts/verify.py

# 3. Check frontmatter
head -20 .claude/skills/[skill-name]/SKILL.md
# Must have: name, description with "Use when"

# 4. Test verify.py
python .claude/skills/[skill-name]/scripts/verify.py
# Must exit 0 or 1 with message

# 5. Check line count
wc -l .claude/skills/[skill-name]/SKILL.md
# Should be <500 lines
```

### Routing Evaluation

Before declaring a skill complete, test routing:

| Test | How | Pass Condition |
|------|-----|----------------|
| Positive | Submit natural language query that should trigger skill | Skill activates |
| Negative | Submit related but different query | Skill does NOT activate |
| Collision | Submit ambiguous query | Only ONE skill activates |

### Skill Commit Messages

Format: `Claude: [action] using [skill-name] skill`

Examples:
- `Claude: deployed kafka using deploying-kafka-k8s skill`
- `Claude: scaffolded triage-service using scaffolding-fastapi-dapr skill`

### Modifying Existing Skills

1. Check if change is backward-compatible
2. If breaking change → create new skill with `-v2` suffix
3. Update any skills that depend on this one
4. Re-run routing evals
5. Re-validate with `skills-ref validate`

### The Ultimate Test

> **Would you recreate this skill if it were deleted?**

If the answer is no, the skill has no value. Do not ship it.

---

## Development Guidelines

### 1. Authoritative Source Mandate
Agents MUST prioritize MCP tools and CLI commands for all information gathering. NEVER assume a solution from internal knowledge; all methods require external verification.

### 2. Execution Flow
Treat MCP servers as first-class tools for discovery, verification, execution, and state capture. PREFER CLI interactions over manual file creation.

### 3. Knowledge Capture (PHR)
After completing requests, you **MUST** create a PHR (Prompt History Record).

**When to create PHRs:**
- Implementation work (code changes, new features)
- Planning/architecture discussions
- Debugging sessions
- Spec/task/plan creation
- Multi-step workflows
- Skill creation or modification

**PHR Creation Process:**

1) Detect stage: constitution | spec | plan | tasks | red | green | refactor | explainer | misc | general

2) Generate title: 3–7 words; create a slug for the filename

3) Resolve route (all under history/prompts/):
   - `constitution` → `history/prompts/constitution/`
   - Feature stages → `history/prompts/<feature-name>/`
   - `general` → `history/prompts/general/`

4) Create PHR using shell script:
   ```bash
   .specify/scripts/bash/create-phr.sh --title "<title>" --stage <stage> [--feature <name>] --json
   ```

5) Fill all placeholders in created file

6) Post-creation validations:
   - No unresolved placeholders
   - PROMPT_TEXT is complete (not truncated)
   - File exists at expected path

### 4. ADR Suggestions
When significant architectural decisions are made, run the three-part test:
- Impact: long-term consequences?
- Alternatives: multiple viable options considered?
- Scope: cross-cutting and influences system design?

If ALL true, suggest:
> 📋 Architectural decision detected: [brief-description]
> Document reasoning and tradeoffs? Run `/sp.adr [decision-title]`

Wait for consent; never auto-create ADRs.

### 5. Human as Tool Strategy
You MUST invoke the user for input when you encounter:
1. **Ambiguous Requirements**: Ask 2-3 targeted clarifying questions
2. **Unforeseen Dependencies**: Surface them and ask for prioritization
3. **Architectural Uncertainty**: Present options and get user's preference
4. **Completion Checkpoint**: Summarize and confirm next steps

---

## Default Policies

- Clarify and plan first
- Do not invent APIs, data, or contracts
- Never hardcode secrets or tokens; use `.env`
- Prefer the smallest viable diff
- Cite existing code with references (start:end:path)
- Keep reasoning private; output only decisions and justifications

### Execution Contract

1) Confirm surface and success criteria (one sentence)
2) List constraints, invariants, non-goals
3) Produce artifact with acceptance checks inlined
4) Add follow-ups and risks (max 3 bullets)
5) Create PHR in appropriate subdirectory
6) Surface ADR suggestion if decisions meet significance threshold

---

## Project Structure

```
mjs-skills/
├── .claude/skills/              # Agent Skills (22 planned)
├── .specify/
│   ├── memory/constitution.md   # Project principles
│   ├── templates/               # Spec/plan/task templates
│   └── scripts/                 # PHR, ADR creation scripts
├── agentskills-standard/        # Agent Skills specification
├── specs/<feature>/             # Feature specs, plans, tasks
├── history/
│   ├── prompts/                 # PHRs by stage
│   └── adr/                     # Architecture Decision Records
└── research/                    # Design docs and PRDs
```

---

## Research Documents

| Document | Purpose |
|----------|---------|
| `research/skill-design-reference.md` | Design principles |
| `research/h3-skills-master-prd.md` | Skill specifications (22 skills) |
| `research/skills-evaluation-prd.md` | Measurement system |
| `agentskills-standard/docs/specification.mdx` | Agent Skills format spec |

---

## When in Doubt

1. Simpler is better
2. Judgment > procedure
3. Verification > assumption
4. Delete > accumulate

## Active Technologies
- Bash 4+ for hooks, Python 3.8+ for setup/verify/analysis + jq (JSON parsing in bash), Python standard library only (001-skill-tracker)
- JSONL files in `.claude/activity-logs/` (prompts.jsonl, skill-usage.jsonl) (001-skill-tracker)

## Recent Changes
- 001-skill-tracker: Added Bash 4+ for hooks, Python 3.8+ for setup/verify/analysis + jq (JSON parsing in bash), Python standard library only

---
> Source: [mjunaidca/mjs-agent-skills](https://github.com/mjunaidca/mjs-agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
