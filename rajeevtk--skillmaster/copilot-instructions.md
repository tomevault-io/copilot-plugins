## skillmaster

> Skill Master uses the **Claude Agent SDK** to orchestrate an agentic pipeline that generates Claude Agent Skills from unstructured inputs. This document explains the agent architecture, skill generation flow, and how to work with the skills in this project.

# Agents & Skills Reference

## Overview

Skill Master uses the **Claude Agent SDK** to orchestrate an agentic pipeline that generates Claude Agent Skills from unstructured inputs. This document explains the agent architecture, skill generation flow, and how to work with the skills in this project.

## Agent Architecture

### Skill Generation Pipeline

The pipeline runs as a multi-step agentic loop using `claude-code-sdk`:

```
Input → Processor → Analyzer → Generator → Validator → GCS Storage
```

1. **Processor** — Extracts structured text from the input source (plain text, voice transcript, Google Slides, interview notes)
2. **Analyzer** — Uses Claude to classify the GTM use case, extract requirements, identify personas, workflows, and tools
3. **Generator** — Uses Claude with the skill-creator pattern to draft a SKILL.md following best practices
4. **Validator** — Checks the generated skill against structural and quality rules
5. **Storage** — Uploads the validated skill to GCS (bucket resolved from env var or NeonDB)

### Using the Claude Agent SDK

```python
from claude_code_sdk import query, ClaudeCodeOptions

# Run a Claude agent with custom system prompt and tools
result = await query(
    prompt="Analyze this GTM workflow and generate a skill...",
    options=ClaudeCodeOptions(
        system_prompt="You are a skill authoring expert...",
        max_turns=10,
    )
)
```

The SDK handles the agentic loop, tool execution, and context management. Our orchestrator wraps this with domain-specific logic for GTM skill generation.

## Skills in This Project

### Skill Master's Own Skill

Located at `skills/skill-master/SKILL.md`, this is a meta-skill: it defines how Skill Master itself generates skills. It can be loaded into Claude to enable skill creation directly in conversation.

**Key files:**
- `skills/skill-master/SKILL.md` — Main skill definition
- `skills/skill-master/GTM_TEMPLATES.md` — GTM-specific templates and patterns
- `skills/skill-master/VALIDATION.md` — Validation rules for generated skills

### Generated Skills

Skills produced by the service follow this structure:

```
{skill-name}/
├── SKILL.md              # Main instructions (YAML frontmatter + body)
├── reference/            # Additional reference files (optional)
│   ├── workflows.md
│   └── templates.md
└── scripts/              # Utility scripts (optional)
    └── validate.py
```

## Skill Authoring Best Practices (Summary)

Generated skills must follow these rules:

### YAML Frontmatter
- `name`: ≤64 chars, lowercase + hyphens, no reserved words ("anthropic", "claude")
- `description`: Non-empty, ≤1024 chars, third person, includes what + when

### Body
- Under 500 lines
- Concise — only include context Claude doesn't already know
- Progressive disclosure — reference files for details, keep SKILL.md as overview
- One-level-deep references (no nested chains)
- Consistent terminology throughout
- No time-sensitive information

### Structure
- Prefer gerund naming: `analyzing-pipeline`, `scoring-leads`, `planning-campaigns`
- Use workflows with checklists for complex multi-step tasks
- Include feedback loops for quality-critical operations
- Provide concrete examples (input/output pairs) over abstract descriptions

### GTM-Specific Patterns
- Include domain terminology (MQL, SQL, ARR, CAC, LTV, ICP, ABM)
- Reference common GTM tools (Salesforce, HubSpot, Outreach, Gong, etc.)
- Template categories: sales enablement, lead scoring, pipeline management, campaign planning, competitive intelligence, customer success, content creation, ABM

## Configuration

### GCS Bucket Resolution

Controlled by the `SKILLMASTER_STORAGE_MODE` env var:

- **`env`** (default) — Use the bucket name from `SKILLMASTER_GCS_BUCKET` directly. Simple single-tenant setup.
- **`db`** — Multi-tenant mode. Look up the bucket per org/team from NeonDB. Raises an error if no bucket is configured for the given org/team.

The DB lookup function is currently stubbed out. Replace the stub in `src/skillmaster/storage/config_resolver.py` once the NeonDB schema is deployed.

### NeonDB Schema (for org/team settings)

```sql
CREATE TABLE org_settings (
    org_id TEXT PRIMARY KEY,
    gcs_bucket TEXT,
    default_skill_prefix TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE team_settings (
    team_id TEXT PRIMARY KEY,
    org_id TEXT REFERENCES org_settings(org_id),
    gcs_bucket TEXT,  -- overrides org-level
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rajeevtk)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rajeevtk)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
