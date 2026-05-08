## agent-use-skills

> AgentUse Skills is a curated library of reusable procedural knowledge for AI Agents. This repository provides standardized skills that enable AI coding agents to follow structured workflows for software development tasks.

# AGENTS.md - AI Agent Guidelines

## Project Overview

AgentUse Skills is a curated library of reusable procedural knowledge for AI Agents. This repository provides standardized skills that enable AI coding agents to follow structured workflows for software development tasks.

## Build/Lint/Test Commands

### Repository Structure Validation
```bash
# List all markdown files to verify structure
find . -name "*.md" -type f

# Verify skill directories have required files
ls -la awesome-skills/skills/*/SKILL.md

# Verify introduction files exist for both languages
ls -la awesome-skills/introductions/zh/*.md
ls -la awesome-skills/introductions/en/*.md
```

### Python Scripts
Some skills include Python scripts for automation:

```bash
# Install dependencies for a specific skill
cd awesome-skills/skills/<skill-name>
pip install -r requirements.txt

# Run a Python script directly
python3 scripts/<script-name>.py [args]

# Example: Run the skill market CLI
python3 awesome-skills/skills/skill-market/scripts/market.py list
python3 awesome-skills/skills/skill-market/scripts/market.py info <skill-name>
```

### Node.js Scripts
Skills with Node.js dependencies (e.g., playwright-skill):

```bash
# Install dependencies
cd awesome-skills/skills/playwright-skill
npm install && npx playwright install chromium

# Run Playwright tests
node run.js /tmp/playwright-test-*.js
```

### Manual Validation Checklist
1. Verify each skill has both Chinese (`-zh.md`) and English (`-en.md`) installation guides
2. Verify SKILL.md has valid YAML frontmatter with `name` and `description`
3. Check all internal links are valid relative paths
4. Ensure kebab-case naming for all files and directories

## Code Style Guidelines

### File Structure Conventions

**Directory Structure:**
```
.agent/
└── skills/                     # Installed skills location

awesome-skills/
├── skills/
│   └── <skill-name>/
│       ├── SKILL.md            # Required: Skill definition with YAML frontmatter
│       ├── _meta.json          # Optional: Metadata for skill market
│       ├── scripts/            # Optional: Python/JS/Shell scripts
│       ├── assets/             # Optional: Static files (templates, images)
│       ├── references/         # Optional: Reference documentation
│       ├── agents/             # Optional: Agent configuration files
│       └── requirements.txt    # Optional: Python dependencies
├── introductions/
│   ├── zh/<skill-name>.md      # Chinese skill introduction
│   └── en/<skill-name>.md      # English skill introduction
├── claudecode/<skill-name>/
│   ├── INSTALL-zh.md
│   └── INSTALL-en.md
├── cursor/<skill-name>/
├── codex/<skill-name>/
├── opencode/<skill-name>/
├── openclaw/<skill-name>/
└── qoder/<skill-name>/

agent-ready/
├── zh/                         # Agent-ready Chinese guides
└── en/                         # Agent-ready English guides
```

**Naming Conventions:**
- Use **kebab-case** for all file and directory names (e.g., `superpowers`, `ui-ux-pro-max-skill`)
- Skill names in `SKILL.md` use lowercase with hyphens
- Platform directories are lowercase (e.g., `claudecode`, `opencode`)
- Script files use kebab-case (e.g., `md_to_docx.py`, `generate_image.py`)

### Documentation Style

**Skill Introduction Documents (`skills/{zh,en}/<skill-name>.md`):**
```markdown
# <Skill Name>

**<Skill Name>** is ... (one-line summary)

## Core Philosophy
- Point 1
- Point 2

## Key Features & Workflow
1. **Feature A**: Description...
2. **Feature B**: Description...

## Skills Library Overview
- **Category 1**: List of sub-skills...

## Installation & Support
<Skill Name> supports the following AI editors and platforms:
- [Claude Code](../../claudecode/<skill-name>/INSTALL-en.md)
- [Cursor](../../cursor/<skill-name>/INSTALL-en.md)
- [OpenCode](../../opencode/<skill-name>/INSTALL-en.md)

---
For more information, visit: [GitHub - <repo>](<url>)
```

**Installation Guide Documents (`INSTALL-*.md`):**
- Include numbered steps with clear commands
- Add verification steps to confirm successful installation
- Link to detailed documentation when applicable
- End with reference link to original project

### Bilingual Requirements

- All documents must exist in **both Chinese** (`-zh.md`) and **English** (`-en.md`)
- Content structure must match between language versions
- Use consistent terminology across translations
- Chinese documents go in `zh/` directories, English in `en/`

### SKILL.md Format

Each skill must include a `SKILL.md` file with:
1. YAML frontmatter with `name` and `description`
2. Clear workflow steps with numbered phases
3. Directory structure diagrams where applicable
4. Important notes and edge cases

Example frontmatter:
```yaml
---
name: skill-name
description: A standardized workflow for...
---
```

### Markdown Formatting

- Use GitHub-flavored markdown
- Headers use ATX style (`#`, `##`, `###`)
- Code blocks specify language for syntax highlighting
- Tables use standard markdown table syntax
- Links use relative paths within the repository
- Maximum line length: 120 characters (except code blocks)

### Git Conventions

**Branch Naming:**
- `feature/<skill-name>` - Adding new skills
- `docs/<description>` - Documentation updates
- `fix/<description>` - Bug fixes

**Commit Messages:**
- Use conventional commits format: `type(scope): description`
- Types: `feat`, `docs`, `fix`, `refactor`, `chore`
- Scope: skill name or affected area

### Script Code Style (Python)

Scripts in `awesome-skills/skills/*/scripts/` should follow these conventions:

```python
#!/usr/bin/env python3
"""Module docstring describing purpose."""

import argparse
import sys
from pathlib import Path

# Constants at top (UPPER_SNAKE_CASE)
DEFAULT_TIMEOUT = 30
API_ENDPOINT = "https://api.example.com"

def main_function(arg: str) -> dict:
    """Function docstring with Args/Returns."""
    pass

if __name__ == "__main__":
    main()
```

**Python Conventions:**
- Use `#!/usr/bin/env python3` shebang for executable scripts
- Prefer standard library (`urllib`, `json`, `pathlib`) over third-party dependencies
- External dependencies documented in `requirements.txt` with version pins
- Use `argparse` for CLI argument parsing
- Print errors to `sys.stderr`, not stdout
- Exit with `sys.exit(1)` on errors
- Type hints encouraged for function signatures

### Script Code Style (JavaScript/Node.js)

```javascript
#!/usr/bin/env node
// Brief description at top

const args = process.argv.slice(2);

function processQuery(query, options) {
  // Implementation
}

await main();  // Use top-level await for async
```

**JavaScript Conventions:**
- Use `#!/usr/bin/env node` shebang for executable scripts
- Use ES modules (`.mjs` extension or `"type": "module"` in package.json)
- Use `const`/`let`, avoid `var`
- Async/await preferred over `.then()` chains
- Use `console.error()` for errors, `console.log()` for output

### Content Guidelines

**Do:**
- Keep skill descriptions concise and actionable
- Use active voice in workflow instructions
- Include verification steps for all procedures
- Link to official documentation for external references
- Maintain parallel structure in bilingual content

**Don't:**
- Embed installation commands directly in introduction docs (use hyperlinked lists instead)
- Create installation guides for unsupported platforms
- Mix Chinese and English content in the same file
- Use absolute URLs for internal repository links

### Quality Checklist

Before submitting any skill:
- [ ] Both language versions created with matching structure
- [ ] Installation guides for all supported platforms
- [ ] All internal links verified
- [ ] Kebab-case naming throughout
- [ ] SKILL.md includes complete workflow
- [ ] References link to official sources

### Platform Support Matrix

When documenting platform support, verify actual compatibility:
- **Claude Code**: Uses `/plugin add` or `/plugin-add` commands
- **Cursor**: Uses `/plugin-add` in Agent mode
- **OpenCode**: Uses `skill` tool with file-based loading
- **Codex**: File-based skill loading
- **OpenClaw**: Platform-specific integration
- **Qoder**: Platform-specific integration

Only create installation guides for platforms explicitly supported by the skill.

---

## Existing Rules

No Cursor rules (`.cursor/rules/` or `.cursorrules`) or Copilot rules (`.github/copilot-instructions.md`) are currently defined in this repository.

---
> Source: [Zerone-Agent/agent-use-skills](https://github.com/Zerone-Agent/agent-use-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
