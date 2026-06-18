## skill-creator

> Guide for creating effective Claude Code skills from scratch or by converting MCP servers into skills. Use when users want to create a new skill, update an existing skill, or convert an MCP server configuration into a context-efficient skill. Covers skill structure (SKILL.md, scripts, references, assets), progressive disclosure patterns, packaging, and MCP-to-skill conversion with dynamic tool invocation.


# Skill Creator

This skill provides guidance for creating effective skills — either from scratch or by converting existing MCP servers into skills.

## About Skills

Skills are modular, self-contained packages that extend Claude's capabilities by providing specialized knowledge, workflows, and tools. They transform Claude from a general-purpose agent into a specialized agent equipped with procedural knowledge.

### What Skills Provide

1. Specialized workflows - Multi-step procedures for specific domains
2. Tool integrations - Instructions for working with specific file formats or APIs
3. Domain expertise - Company-specific knowledge, schemas, business logic
4. Bundled resources - Scripts, references, and assets for complex and repetitive tasks

## Two Creation Modes

### Mode 1: Create from Scratch

Build a brand-new skill using the structured creation process. Best for custom workflows, domain expertise, and unique tool integrations.

### Mode 2: Convert from MCP Server

Convert an existing MCP server into a skill with ~90% context savings. Best when you already have an MCP server and want to reduce its token footprint.

See [references/mcp-conversion.md](references/mcp-conversion.md) for the complete MCP conversion guide.

## Core Principles

### Concise is Key

The context window is a public good. Skills share it with system prompts, conversation history, other skills, and user requests.

**Default assumption: Claude is already very smart.** Only add context Claude doesn't already have. Challenge each piece of information: "Does Claude really need this explanation?" and "Does this paragraph justify its token cost?"

Prefer concise examples over verbose explanations.

### Set Appropriate Degrees of Freedom

Match specificity to the task's fragility and variability:

- **High freedom** (text instructions): Multiple valid approaches, context-dependent decisions
- **Medium freedom** (pseudocode/parameterized scripts): Preferred pattern exists, some variation acceptable
- **Low freedom** (specific scripts, few params): Fragile operations, consistency critical, specific sequence required

### Anatomy of a Skill

```
skill-name/
├── README.md                         # Instructions on how to use the skill and what it does
├── SKILL.md (required)
│   ├── YAML frontmatter (name + description, required)
│   └── Markdown instructions (required)
└── Bundled Resources (optional)
    ├── scripts/       - Executable code (Python/Bash/etc.)
    ├── references/    - Documentation loaded into context as needed
    └── assets/        - Files used in output (templates, icons, fonts)
```

### README.md (required)

Instructions on how to use the skill and what it does.  Convenient link to ./SKILL.md.

#### SKILL.md (required)

- **Frontmatter** (YAML): `name` and `description` fields. These determine when the skill triggers — be clear and comprehensive.
- **Body** (Markdown): Instructions loaded AFTER the skill triggers.

#### Scripts (`scripts/`)

Executable code for tasks needing deterministic reliability or repeatedly rewritten code.

#### References (`references/`)

Documentation loaded into context as needed. Keep SKILL.md lean; move detailed info here.

#### Assets (`assets/`)

Files used in output but not loaded into context (templates, images, fonts, boilerplate).

### Progressive Disclosure

Skills use three-level loading:

1. **Metadata** (name + description) - Always in context (~100 tokens)
2. **SKILL.md body** - When skill triggers (<5k tokens)
3. **Bundled resources** - As needed (unlimited, scripts run without context loading)

Keep SKILL.md under 500 lines. Split content into reference files when approaching this limit.

**Pattern 1: High-level guide with references**

```markdown
## Advanced features
- **Form filling**: See [FORMS.md](references/forms.md)
- **API reference**: See [REFERENCE.md](references/api.md)
```

**Pattern 2: Domain-specific organization**

```
bigquery-skill/
├── SKILL.md (overview + navigation)
└── references/
    ├── finance.md
    ├── sales.md
    └── product.md
```

**Pattern 3: Conditional details**

```markdown
## Editing documents
For simple edits, modify XML directly.
**For tracked changes**: See [REDLINING.md](references/redlining.md)
```

## Skill Creation Process (From Scratch)

### Step 1: Understand the Skill with Concrete Examples

Ask clarifying questions:
- "What functionality should this skill support?"
- "Can you give examples of how it would be used?"
- "What would a user say that should trigger this skill?"

### Step 2: Plan Reusable Skill Contents

For each concrete example, identify what scripts, references, and assets would help when executing workflows repeatedly.

### Step 3: Initialize the Skill

Run the initialization script to generate a template:

```bash
python scripts/init_skill.py <skill-name> --path <output-directory>
```

This creates the directory with SKILL.md template and example resource directories.

### Step 4: Edit the Skill

Remember: the skill is for another instance of Claude to use.

#### Consult Design Patterns

- **Multi-step processes**: See [references/workflows.md](references/workflows.md)
- **Output formats/quality**: See [references/output-patterns.md](references/output-patterns.md)

#### Frontmatter Guidelines

```yaml
---
name: skill-name
description: Complete explanation of what the skill does and when to use it. Include triggers, scenarios, and file types.
---
```

Put ALL "when to use" info in the description — the body only loads after triggering.

#### Body Guidelines

- Use imperative/infinitive form
- Include only non-obvious information
- Test any added scripts by running them

### Step 5: Install the Skill

```bash
npx skills add <path/to/skill-folder>
```

This registers the skill so skill users can use it.

### Step 6: Iterate

1. Use the skill on real tasks
2. Notice struggles or inefficiencies
3. Update SKILL.md or bundled resources
4. Test again

## MCP-to-Skill Conversion Process

For converting existing MCP servers, use the built-in converter:

```bash
python scripts/mcp_to_skill.py --mcp-config config.json --output-dir ./skills/my-mcp-skill
```

This generates a complete skill with:
- `SKILL.md` - Claude instructions with tool list
- `executor.py` - Dynamic MCP communication handler
- `mcp-config.json` - Server configuration
- `package.json` - Dependencies

See [references/mcp-conversion.md](references/mcp-conversion.md) for:
- Config file format and examples
- How progressive disclosure works for MCP
- Context savings analysis
- Troubleshooting guide

### Example MCP Config

```json
{
  "name": "github",
  "description": "GitHub API access via MCP",
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-github"],
  "env": {
    "GITHUB_TOKEN": "your-github-token-here"
  }
}
```

See [assets/example-github-mcp.json](assets/example-github-mcp.json) for a ready-to-use example.

---
> Source: [alexanderchan/skill-creator](https://github.com/alexanderchan/skill-creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
