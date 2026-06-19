## skill-forge

> Automated skill creation workshop with intelligent source detection, smart path management, and end-to-end workflow automation. This skill should be used when users want to create a new skill or convert external resources (GitHub repositories, online documentation, or local directories) into a skill. Automatically fetches, organizes, and packages skills with proactive cleanup management.


# Skill Forge

An automated skill creation workshop that provides end-to-end guidance for forging effective skills. Features intelligent source detection, smart path management, automatic material fetching from GitHub repositories, online documentation (with llms.txt support), or local directories, and comprehensive cleanup tools.

## About Skills

Skills are modular, self-contained packages that extend Claude's capabilities by providing
specialized knowledge, workflows, and tools. Think of them as "onboarding guides" for specific
domains or tasks—they transform Claude from a general-purpose agent into a specialized agent
equipped with procedural knowledge that no model can fully possess.

### What Skills Provide

1. Specialized workflows - Multi-step procedures for specific domains
2. Tool integrations - Instructions for working with specific file formats or APIs
3. Domain expertise - Company-specific knowledge, schemas, business logic
4. Bundled resources - Scripts, references, and assets for complex and repetitive tasks

### Anatomy of a Skill

Every skill consists of a required SKILL.md file and optional bundled resources:

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter metadata (required)
│   │   ├── name: (required)
│   │   └── description: (required)
│   └── Markdown instructions (required)
└── Bundled Resources (optional)
    ├── scripts/          - Executable code (Python/Bash/etc.)
    ├── references/       - Documentation intended to be loaded into context as needed
    └── assets/           - Files used in output (templates, icons, fonts, etc.)
```

#### SKILL.md (required)

**Metadata Quality:** The `name` and `description` in YAML frontmatter determine when Claude will use the skill. Be specific about what the skill does and when to use it. Use the third-person (e.g. "This skill should be used when..." instead of "Use this skill when...").

#### Bundled Resources (optional)

##### Scripts (`scripts/`)

Executable code (Python/Bash/etc.) for tasks that require deterministic reliability or are repeatedly rewritten.

- **When to include**: When the same code is being rewritten repeatedly or deterministic reliability is needed
- **Example**: `scripts/rotate_pdf.py` for PDF rotation tasks
- **Benefits**: Token efficient, deterministic, may be executed without loading into context
- **Note**: Scripts may still need to be read by Claude for patching or environment-specific adjustments

##### References (`references/`)

Documentation and reference material intended to be loaded as needed into context to inform Claude's process and thinking.

- **When to include**: For documentation that Claude should reference while working
- **Examples**: `references/finance.md` for financial schemas, `references/mnda.md` for company NDA template, `references/policies.md` for company policies, `references/api_docs.md` for API specifications
- **Use cases**: Database schemas, API documentation, domain knowledge, company policies, detailed workflow guides
- **Benefits**: Keeps SKILL.md lean, loaded only when Claude determines it's needed
- **Best practice**: If files are large (>10k words), include grep search patterns in SKILL.md
- **Avoid duplication**: Information should live in either SKILL.md or references files, not both. Prefer references files for detailed information unless it's truly core to the skill—this keeps SKILL.md lean while making information discoverable without hogging the context window. Keep only essential procedural instructions and workflow guidance in SKILL.md; move detailed reference material, schemas, and examples to references files.

##### Assets (`assets/`)

Files not intended to be loaded into context, but rather used within the output Claude produces.

- **When to include**: When the skill needs files that will be used in the final output
- **Examples**: `assets/logo.png` for brand assets, `assets/slides.pptx` for PowerPoint templates, `assets/frontend-template/` for HTML/React boilerplate, `assets/font.ttf` for typography
- **Use cases**: Templates, images, icons, boilerplate code, fonts, sample documents that get copied or modified
- **Benefits**: Separates output resources from documentation, enables Claude to use files without loading them into context

### Progressive Disclosure Design Principle

Skills use a three-level loading system to manage context efficiently:

1. **Metadata (name + description)** - Always in context (~100 words)
2. **SKILL.md body** - When skill triggers (<5k words)
3. **Bundled resources** - As needed by Claude (Unlimited*)

*Unlimited because scripts can be executed without reading into context window.

## Path Management Overview

Three types of paths with different management strategies:

| Type | Location | Strategy |
|------|----------|----------|
| **Materials** (temp) | `.claude/temp-materials/` or `~/skill-materials/` | Auto-detected |
| **Skill** (permanent) | User chooses location | Ask user |
| **Zip** (package) | Inside skill directory | Default |

📖 **Detailed Guide**: See [path-management.md](references/path-management.md) for smart path detection, cleanup strategies, and examples.

## When to Use This Skill

This skill should be triggered when users request to create a new skill or convert external resources into a skill. Common trigger patterns:

- "Create a skill from [URL/path]"
- "Turn this into a skill: [URL/path]"
- "Help me make a skill for [description]"
- "Convert [repository/docs] to a skill"
- "Build a skill from [source]"

### Automatic Source Detection

Automatically detect source type and act accordingly:

**Decision Tree:**
```
GitHub URL (github.com/*) → fetch_source.py --git <url>
Documentation URL (docs.*, */docs/) → fetch_source.py --docs <url> --name <name>
Local Path (~/, /, ./) → Use directly (skip fetch)
Ambiguous → Ask user for clarification
```

**Quick Examples:**
- `https://github.com/user/repo` → Auto-clone to materials directory
- `https://docs.example.com` → Auto-scrape documentation
- `~/my-tools/` → Use local directory directly
- `"Create skill for X"` → Ask for source clarification

📖 **Detailed Guide**: See [source-detection.md](references/source-detection.md) for complete patterns, edge cases, and examples.

### Proactive Fetching

✅ **Do:** Immediately fetch when source type is clear
❌ **Don't:** Ask permission for obvious actions (GitHub URLs, documentation sites)
❓ **Ask:** Only when genuinely ambiguous

## Skill Creation Process

Follow these steps in order. Skip only when clearly not applicable.

📖 **Complete Workflow**: See [workflow-guide.md](references/workflow-guide.md) for detailed step-by-step instructions and examples.

### Step 0: Fetch Source Materials (Automatic)

**When:** User provides external source (GitHub/docs URL)
**Skip:** When using local directory

#### For GitHub Repositories

```bash
scripts/fetch_source.py --git https://github.com/user/repo
```

No additional dependencies needed (requires `git` command only).

#### For Online Documentation

**🆕 llms.txt Detection (Recommended)**

Many modern documentation sites support the [llms.txt standard](https://llmstxt.org/) - a LLM-optimized documentation format that's 10x faster to fetch.

First, check if the site has llms.txt:

```bash
scripts/detect_llms_txt.py https://docs.example.com
```

If found, use the llms.txt URL directly:
```bash
# Much faster! ⚡
scripts/fetch_source.py --docs https://docs.example.com/llms-full.txt --name project-name
```

**Note:** `fetch_source.py` automatically detects llms.txt and recommends it if available.

**Regular Documentation Scraping**

If no llms.txt available, use regular scraping with markitdown:

First, verify markitdown is installed:

```python
try:
    from markitdown import MarkItDown
    print("✅ MarkItDown is installed")
except ImportError:
    print("❌ Need to install: pip install 'markitdown[all]'")
```

If not installed:
```bash
pip install 'markitdown[all]'
```

Then fetch documentation:
```bash
scripts/fetch_source.py --docs https://docs.example.com --name project-name
```

#### For PDF and Office Documents

markitdown also supports PDF, Word, PowerPoint, and Excel files:

**PDF Documents:**
```bash
# From URL
scripts/fetch_source.py --docs https://example.com/manual.pdf --name manual

# From local file
scripts/fetch_source.py --docs /path/to/document.pdf --name doc
```

**Other Formats:**
```bash
# Word document
scripts/fetch_source.py --docs /path/to/spec.docx --name spec

# PowerPoint
scripts/fetch_source.py --docs /path/to/slides.pptx --name slides

# Excel
scripts/fetch_source.py --docs /path/to/data.xlsx --name data
```

**Notes:**
- Text-based PDFs (most common) work out of the box with markitdown
- Scanned PDFs: Provide the file directly for visual text extraction
- Complex PDFs with tables/graphics may need manual review

#### Combined Mode

```bash
scripts/fetch_source.py --git <url> --docs <url> --name combo
```

**Smart Path Detection:** Materials auto-saved to:
- Project mode: `<project-root>/.claude/temp-materials/`
- Global mode: `~/skill-materials/`

**Common Options:** `--depth 1` (shallow clone), `--branch <name>`, `--output <path>`

📖 See [workflow-guide.md#step-0](references/workflow-guide.md#step-0-fetch-source-materials-automatic) for full options and examples.

### Step 1: Understanding the Skill with Concrete Examples

**Goal:** Clearly understand how the skill will be used through concrete examples.

**Process:**
- Ask users for specific use cases
- Generate example scenarios for validation
- Identify trigger patterns
- Clarify scope and functionality

**Example Questions:**
- "What functionality should this skill support?"
- "What would users say to trigger this skill?"
- "Can you give concrete examples of how it would be used?"

**When to conclude:** Clear sense of the skill's purpose and usage patterns.

📖 See [workflow-guide.md#step-1](references/workflow-guide.md#step-1-understanding-the-skill-with-concrete-examples) for detailed question strategies and best practices.

### Step 2: Planning the Reusable Skill Contents

**Goal:** Identify what scripts, references, and assets to bundle with the skill.

**Analysis Questions:**
1. What code gets rewritten repeatedly? → `scripts/`
2. What documentation needs referencing? → `references/`
3. What templates or files are used in output? → `assets/`

**Common Patterns:**
- Repetitive code → Script (e.g., `scripts/rotate_pdf.py`)
- Schemas/API docs → Reference (e.g., `references/schema.md`)
- Boilerplate/templates → Asset (e.g., `assets/template/`)

**Output:** List of reusable resources to include.

📖 See [workflow-guide.md#step-2](references/workflow-guide.md#step-2-planning-the-reusable-skill-contents) for detailed examples and analysis patterns.

### Step 3: Initializing the Skill

**Goal:** Create skill directory structure using `init_skill.py`.
**Skip:** If skill already exists (jump to Step 4).

**Choosing Skill Location:**

Unlike temporary materials (auto-detected), skills are permanent. **Always ask user** where to create it:

```
Where would you like to create the <skill-name> skill?
1. Project skills (.claude/skills/) - For this project only
2. Global skills (~/.claude/skills/) - Available everywhere
3. Custom path - Specify your own location
```

**Usage:**
```bash
# Based on user choice (1, 2, or 3):
scripts/init_skill.py <skill-name> --path <user-chosen-path>
```

**What it creates:**
- SKILL.md template with frontmatter
- `scripts/`, `references/`, `assets/` directories
- Example files (customize or delete as needed)

📖 See [workflow-guide.md#step-3](references/workflow-guide.md#step-3-initializing-the-skill) for detailed path selection guidance and examples.

### Step 4: Edit the Skill

**Goal:** Implement bundled resources and write SKILL.md instructions.

**Start with Resources:**
1. Implement `scripts/`, `references/`, `assets/` from Step 2
2. Delete unused example files from initialization
3. May require user input (e.g., brand assets, documentation)

**🆕 Smart Processing for Large Documentation Files:**

If documentation was fetched (especially via llms.txt), check file size:

```bash
# Check if documentation file is large
ls -lh <path-to-fetched-docs>
```

**For large files (>50KB or >10,000 words):**

Split into logical sections following Progressive Disclosure principle:

**Example Structure:**
```
references/
├── overview.md          # High-level concepts, getting started
├── core-concepts.md     # Main ideas, architecture
├── api-reference.md     # API documentation
└── advanced.md          # Advanced usage, examples
```

**Splitting Strategy:**
1. Read the large markdown file
2. Identify natural sections (based on headings, topics)
3. Split into 3-5 focused files (each <20KB)
4. Name files descriptively (overview, api, examples, etc.)
5. Update SKILL.md to reference specific files

**Benefits:**
- ✅ Faster context loading (load only needed sections)
- ✅ Better grep-ability (search specific files)
- ✅ Follows Progressive Disclosure design

**For small files (<50KB):**
- Keep as single file in `references/`

**Update SKILL.md:**

**Writing Style:** Use **imperative/infinitive form** (verb-first), not second person.
- ✅ "To accomplish X, do Y"
- ❌ "You should do X"

**Answer These Questions:**
1. What is the purpose? (a few sentences)
2. When should it be used? (trigger patterns)
3. How should Claude use it? (reference all bundled resources)

**Focus:** Include procedural knowledge and domain-specific details that benefit another Claude instance.

📖 See [workflow-guide.md#step-4](references/workflow-guide.md#step-4-edit-the-skill) for writing guidelines and examples.

### Step 5: Completion and Optional Packaging

**CRITICAL:** After Step 4, the skill is already installed and ready to use!

**Required Actions After Step 4:**

1. **Auto-cleanup temporary materials** (if materials exist):
   ```bash
   # Silently clean up temp-materials - user doesn't need to know
   scripts/cleanup_materials.py <skill-name>
   ```

   **Do this automatically without asking.** Materials are no longer needed after skill creation.

2. **Auto-cleanup temporary helper scripts** (if any exist in skill's scripts/ directory):

   **Common temporary scripts to remove:**
   - `split_docs.py` - Document splitting helper (if created during Step 4)
   - `process_*.py` - Any temporary processing scripts
   - `temp_*.py` - Any scripts with "temp" prefix

   **Check and clean:**
   ```bash
   # Example: Remove split_docs.py if it exists
   rm -f <skill-path>/scripts/split_docs.py
   ```

   **Do this silently.** Only keep scripts that are part of the skill's permanent functionality.

3. **Inform user of completion:**
   ```
   ✅ Skill created and installed successfully!
   📁 Location: ~/.claude/skills/crewai/
   🎉 The skill is ready to use immediately!
   ```

4. **ONLY THEN ask about packaging:**
   ```
   📦 Would you like to package it as a .zip for sharing with others? (Optional)
   ```

**⚠️ DO NOT package automatically - must ask user first!**

---

**If user wants to package (optional):**

**Goal:** Validate and package skill into distributable .zip file for sharing.

**Usage:**
```bash
scripts/package_skill.py <path/to/skill-folder>

# Custom output location:
scripts/package_skill.py <path/to/skill-folder> ./dist
```

**Process:**
1. **Validates:** Frontmatter, naming, structure, descriptions
2. **Packages:** Creates `<skill-name>.zip` in skill directory (default)
3. **Reports:** Shows errors if validation fails

**After Packaging:**

```
✅ Skill packaged successfully!
📦 Package: ~/.claude/skills/crewai/crewai.zip
📁 Installed: ~/.claude/skills/crewai/ (ready to use)
```

---

**Note:** Temporary materials are automatically cleaned up after skill creation. Users don't need to manually manage cleanup.

📖 See [workflow-guide.md#step-5](references/workflow-guide.md#step-5-packaging-a-skill) and [path-management.md](references/path-management.md) for detailed workflow documentation.

### Step 6: Iterate

**When:** After testing skill on real tasks
**Trigger:** Users request improvements based on skill performance

**Workflow:**
1. Use skill on real tasks
2. Notice struggles/inefficiencies
3. Update SKILL.md or resources
4. Test again

📖 See [workflow-guide.md#step-6](references/workflow-guide.md#step-6-iterate) for iteration patterns.

---
> Source: [WilliamSaysX/skill-forge](https://github.com/WilliamSaysX/skill-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
