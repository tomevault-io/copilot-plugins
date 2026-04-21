## deep-reading-analyst-skill

> This skill guides users through a 5-step analysis process using **10+ proven thinking frameworks**:

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Deep Reading Analyst** is a Claude AI skill (.skill file) that provides systematic framework for deep analysis of articles, papers, and long-form content using multiple thinking models. It transforms surface-level reading into deep learning through structured workflows.

### Core Purpose

This skill guides users through a 5-step analysis process using **10+ proven thinking frameworks**:

**Quick Analysis (15min)**:
- SCQA Framework (structure thinking)
- 5W2H Analysis (completeness check)

**Standard Analysis (30min)**:
- Critical Thinking (argument evaluation, logical fallacy detection)
- Inversion Thinking (risk and failure mode analysis)

**Deep Analysis (60min)**:
- Mental Models (multi-discipline perspectives: physics, biology, psychology, economics)
- First Principles (assumption stripping, essence extraction)
- Systems Thinking (causal loops, knowledge connections)
- Six Thinking Hats (multi-perspective evaluation)

**Research Level (120min+)**:
- Cross-Source Comparison (multi-article synthesis)

## Repository Structure

### Source Files

- **`src/deep-reading-analyst/`** - Source files for the skill (tracked in Git)
  - `SKILL.md` - Main workflow and decision tree
  - `references/` - Detailed framework documentation (10+ frameworks)
    - `scqa_framework.md` - McKinsey structure thinking (NEW!)
    - `5w2h_analysis.md` - Completeness check with 7 questions (NEW!)
    - `critical_thinking.md` - Argument analysis framework
    - `inversion_thinking.md` - Charlie Munger's risk analysis (NEW!)
    - `mental_models.md` - 30+ models from multiple disciplines (NEW!)
    - `first_principles.md` - Elon Musk's assumption identification
    - `systems_thinking.md` - Relationship mapping and causal loops
    - `six_hats.md` - Edward de Bono's multi-perspective protocol
    - `output_templates.md` - 8 different output formats
    - `comparison_matrix.md` - Multi-source comparison methodology

### Built Package

- **`deep-reading-analyst.skill`** - The packaged skill file (built from `src/`)
  - Format: Standard Claude skill package (ZIP archive)
  - Size: ~40KB (optimized for context window efficiency)
  - Built using: `./build.sh`

### Build Script

- **`build.sh`** - Bash script to package source files into `.skill` file
  - Copies files from `src/deep-reading-analyst/`
  - Creates ZIP archive with proper structure
  - Shows size and contents summary

### Documentation Files

- **`README.md`** - Main documentation with comprehensive usage guide (English)
- **`README_CN.md`** - Complete Chinese documentation with proper spacing
- **`CONTRIBUTING.md`** - Contribution guidelines and skill development rules
- **`CHANGELOG.md`** - Version history
- **`GITHUB_GUIDE.md`** - GitHub repository management guide
- **`examples/EXAMPLES.md`** - Real-world usage examples with timing estimates

### GitHub Configuration

- `.github/ISSUE_TEMPLATE/` - Bug report and feature request templates
- `.github/PULL_REQUEST_TEMPLATE.md` - PR template

## Development Workflow

### Editing the Skill

**Standard workflow:**

```bash
# 1. Edit source files directly
vim src/deep-reading-analyst/SKILL.md
vim src/deep-reading-analyst/references/critical_thinking.md

# 2. Rebuild the skill package
./build.sh

# 3. Test by importing the .skill file into Claude

# 4. Commit both source and built files
git add src/ deep-reading-analyst.skill
git commit -m "feat: add new thinking framework"
```

**Examining the packaged skill:**

```bash
# List contents of the skill archive
unzip -l deep-reading-analyst.skill

# View specific file without extracting
unzip -p deep-reading-analyst.skill deep-reading-analyst/SKILL.md
```

### Key Architectural Decisions

1. **Layered Framework Loading**: The skill uses progressive framework loading based on analysis depth:
   - Level 1 (Quick): Structure only - 15 min
   - Level 2 (Standard): + Critical Thinking - 30 min
   - Level 3 (Deep): + First Principles + Systems + Six Hats - 60 min
   - Level 4 (Research): + Cross-source comparison via web_search - 120+ min

2. **Reference Documents as Knowledge Base**: Instead of embedding all frameworks in SKILL.md, they're separated into `references/` for:
   - Context window efficiency
   - Modular updates
   - Clearer framework documentation

3. **Workflow-Driven Design**: The SKILL.md follows a strict 5-step process:
   - Step 1: Initialize (gather user goals and depth preference)
   - Step 2: Structural Understanding (always performed)
   - Step 3: Apply Thinking Models (based on depth level)
   - Step 4: Synthesis & Output (customized to user goal)
   - Step 5: Knowledge Activation (actionable next steps)

## Skill Trigger Conditions

The skill auto-triggers when users:

- Say: "analyze this article", "help me understand", "deep dive into", "extract insights from"
- Provide URLs or long-form content for analysis
- Request specific frameworks: "use critical thinking", "apply first principles"

## Adding New Features

### Adding a New Thinking Framework

1. Create `src/deep-reading-analyst/references/new_framework.md` following this structure:
   - Overview: What is this framework?
   - When to use: Specific scenarios
   - How to use: Step-by-step process
   - Examples: Concrete applications
   - Common mistakes: What to avoid

2. Reference it in `src/deep-reading-analyst/SKILL.md` at the appropriate depth level

3. Update `README.md` features section

4. Rebuild: `./build.sh`

5. Test the new skill package in Claude

### Adding New Output Templates

1. Add to `src/deep-reading-analyst/references/output_templates.md`
2. Include: clear structure, examples, when to use
3. Make it practical and actionable
4. Rebuild: `./build.sh`

### Modifying the Workflow

- Core workflow is in `src/deep-reading-analyst/SKILL.md`
- Workflow decision tree starts at line 10
- Each step is clearly demarcated with `## Step N:` headers
- Quality standards are defined at lines 138-152
- After changes, run `./build.sh` to rebuild the package

## Testing Changes

Before committing skill modifications:

1. Edit files in `src/deep-reading-analyst/`
2. Rebuild: `./build.sh`
3. Import `deep-reading-analyst.skill` into Claude (Desktop or Web)
4. Test with various content types:
   - Short articles (< 1000 words)
   - Long papers (> 5000 words)
   - Different depth levels (1-4)
   - Different output formats
5. Verify all reference documents load correctly
6. Check existing functionality still works
7. Commit both source and built files

## Important Conventions

### Writing Style for Skill Content

- **Be concise** - Context window is shared resource
- **Use examples** - Concrete over abstract
- **No jargon** - Explain technical terms
- **Action-oriented** - Always end with practical steps

### Commit Message Format

- `feat:` - New feature or framework
- `fix:` - Bug fix in workflow or references
- `docs:` - Documentation changes
- `refactor:` - Restructuring without behavior change
- `test:` - Adding test cases or examples

### File Encoding and Format

- All markdown files use UTF-8 encoding
- Line endings: LF (Unix-style)
- Max line length: No hard limit (markdown flows naturally)
- Skill archive: Standard ZIP format

## Common Tasks

**Edit skill workflow:**

```bash
# Edit the main workflow
vim src/deep-reading-analyst/SKILL.md

# Or view it directly
cat src/deep-reading-analyst/SKILL.md | less
```

**Edit thinking frameworks:**

```bash
# Edit a specific framework
vim src/deep-reading-analyst/references/critical_thinking.md

# View all frameworks
ls -lh src/deep-reading-analyst/references/
```

**Build and test:**

```bash
# Build the skill package
./build.sh

# Check built file size
ls -lh deep-reading-analyst.skill

# Validate structure
unzip -t deep-reading-analyst.skill

# View packaged contents
unzip -l deep-reading-analyst.skill
```

**Quick comparison:**

```bash
# Compare source vs packaged version
diff <(cat src/deep-reading-analyst/SKILL.md) \
     <(unzip -p deep-reading-analyst.skill deep-reading-analyst/SKILL.md)
```

## Quality Standards

When modifying the skill, ensure:

- ✅ Analysis stays faithful to original content (no misrepresentation)
- ✅ Facts distinguished from opinions
- ✅ Concrete examples provided
- ✅ Actionable steps included
- ✅ Total size remains under 100KB for context efficiency

Avoid:

- ❌ Overwhelming users with all frameworks at once
- ❌ Academic jargon without explanation
- ❌ Analysis without application
- ❌ Copying text verbatim

## Version Information

- **Current version: 2.0.0**
- **What's new in 2.0:**
  - Added 4 major frameworks: SCQA (McKinsey), 5W2H Analysis, Mental Models (30+ models), Inversion Thinking
  - Total framework count: 6 → 10+
  - All content in idiomatic English for global accessibility
  - Comprehensive usage guide merged into README.md
  - Chinese documentation with proper character spacing
- Initial release: 2025-11-02 (v1.0.0)
- v2.0.0 release: 2025-11-04
- Compatible with: Claude Sonnet 4.5+
- License: MIT

---
> Source: [ginobefun/deep-reading-analyst-skill](https://github.com/ginobefun/deep-reading-analyst-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
