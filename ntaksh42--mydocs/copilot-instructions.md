## mydocs

> **myDocs** is a comprehensive technical documentation and knowledge base website built with Astro and MDX. The project focuses on cutting-edge software development technologies, AI/ML tools, and the Model Context Protocol (MCP) ecosystem. Content is primarily written in Japanese with English technical terminology.

# CLAUDE.md - AI Assistant Guide for myDocs Repository

## Project Overview

**myDocs** is a comprehensive technical documentation and knowledge base website built with Astro and MDX. The project focuses on cutting-edge software development technologies, AI/ML tools, and the Model Context Protocol (MCP) ecosystem. Content is primarily written in Japanese with English technical terminology.

**Key Technologies:**
- **Framework:** Astro 5.15.7 (static site generator)
- **Content Format:** MDX 4.3.10 (Markdown + JSX components)
- **Language:** TypeScript (strict mode enabled)
- **Primary Language:** Japanese with English technical terms

**Project Focus Areas:**
1. AI/ML technologies and tools (Claude, generative AI models, coding agents)
2. Model Context Protocol (MCP) servers and integrations
3. Microsoft ecosystem (.NET 10, C# 14, Blazor)
4. Software development best practices for 2025

---

## Repository Structure

```
/home/user/myDocs/
├── .git/                          # Git repository
├── .vscode/                       # VSCode configuration
│   ├── extensions.json            # Recommended: Astro extension
│   └── launch.json                # Debug configuration
├── src/                           # Astro source directory
│   ├── pages/                     # MDX pages (auto-routed)
│   │   ├── index.mdx              # Homepage
│   │   ├── ai-coding-agents-2025.mdx
│   │   ├── ai-glossary-beginners.mdx
│   │   ├── ai-output-best-practices.mdx
│   │   ├── ai-utilization-techniques-2025.mdx
│   │   ├── blazor-architecture-guide.mdx
│   │   ├── claude-code-features.mdx
│   │   ├── dotnet10-overview.mdx
│   │   └── latest-generative-ai-models-2025.mdx
│   ├── layouts/                   # Layout components
│   │   └── Layout.astro           # Base page layout
│   ├── components/                # Reusable Astro components
│   │   └── Welcome.astro
│   └── assets/                    # Static assets (SVGs, images)
├── public/                        # Public static files
│   └── favicon.svg
├── Root documentation files:
│   ├── csharp-14-features.md      # C# 14 reference (404 lines)
│   ├── microsoft-mcp-servers-2025.md  # MCP guide (1058 lines)
│   ├── mcp-servers-software-development-2025.md
│   └── programming-article-sites-and-mcp-servers-2025.md
├── .gitignore                     # Git exclusions
├── astro.config.mjs              # Astro configuration
├── tsconfig.json                 # TypeScript config (strict mode)
├── package.json                  # Dependencies
├── package-lock.json             # Locked versions
└── README.md                     # Boilerplate Astro README
```

**Important Paths:**
- **MDX Pages:** `/home/user/myDocs/src/pages/`
- **Layouts:** `/home/user/myDocs/src/layouts/`
- **Components:** `/home/user/myDocs/src/components/`
- **Build Output:** `./dist/` (gitignored)

---

## Development Workflow

### Build Commands

All commands run from the repository root:

| Command | Purpose |
|---------|---------|
| `npm install` | Install dependencies (first-time setup) |
| `npm run dev` | Start local dev server at `localhost:4321` |
| `npm run build` | Build production site to `./dist/` |
| `npm run preview` | Preview built site locally |
| `npm run astro ...` | Run Astro CLI commands |

### Local Development Process

1. **Start Development Server:**
   ```bash
   npm run dev
   ```
   Access site at `http://localhost:4321`

2. **Make Changes:**
   - Edit MDX files in `src/pages/`
   - Changes hot-reload automatically

3. **Build for Production:**
   ```bash
   npm run build
   ```
   Verify build succeeds before committing

4. **Preview Production Build:**
   ```bash
   npm run preview
   ```

---

## Git Workflow and Branch Naming

### Branch Strategy

**Main Branch:** (Not explicitly named, but assumed to be `main` or `master`)

**Feature Branches:** Follow the pattern `claude/[description]-[ID]`

Examples from history:
- `claude/ai-glossary-beginners-015fLNhLpkhzSTsJvsKGJR54`
- `claude/dotnet10-overview-mdx-01WVvDFV9acAx6YPJsbW6c3w`
- `claude/blazor-framework-basics-01GFCQGfTbuWTQ1eyVDiEzeA`

**Current Active Branch:**
- `claude/claude-md-mi9p1tbqgmggoov2-018oe3Ru8oDH3HbPEgY1342S`

### Commit Message Format

Follow these patterns based on commit history:

**Format:** `[Action] [descriptive summary] [optional: (#PR-number)]`

**Examples:**
```
Add comprehensive Blazor architecture and mechanism guide in Japanese (#10)
Add comprehensive Microsoft MCP servers documentation in Japanese (#9)
Add comprehensive MCP servers guide for software development 2025 (#8)
Add comprehensive guide for programming article platforms and MCP servers 2025 (#7)
Add comprehensive C# 14 features guide with official documentation (#6)
```

**Conventions:**
- Use "Add" for new documentation/features
- Be descriptive about what was added
- Include language if content is in Japanese
- Reference PR number when merging
- Keep messages under 100 characters when possible

### Pull Request Workflow

1. **Create Feature Branch:**
   ```bash
   git checkout -b claude/[description]-[unique-ID]
   ```

2. **Develop and Commit:**
   ```bash
   git add [files]
   git commit -m "Add comprehensive [topic] guide in Japanese"
   ```

3. **Push to Remote:**
   ```bash
   git push -u origin claude/[branch-name]
   ```
   **Note:** Branch must start with `claude/` and end with matching session ID

4. **Create Pull Request:**
   Use GitHub web interface or CLI

5. **Merge:**
   Commits show merge pattern: `Merge pull request #N from ntaksh42/[branch-name]`

---

## Content and Documentation Conventions

### MDX Page Structure

Every MDX page in `src/pages/` follows this structure:

```mdx
---
layout: ../layouts/Layout.astro
title: [Page Title in Japanese]
description: [Brief description in Japanese]
---

# [Main Heading - Same as Title]

## 概要

[Overview section in Japanese]

## [Section 1]

[Content...]

### [Subsection 1.1]

[Content...]

---

## [Section 2]

[Continue with numbered sections...]
```

**Key Elements:**
1. **Frontmatter (YAML):**
   - `layout:` Always points to `../layouts/Layout.astro`
   - `title:` Page title (appears in browser tab/SEO)
   - `description:` Brief description for SEO/metadata

2. **Main Heading (H1):**
   - Use single `#` for main title
   - Often matches frontmatter title
   - May include year: `(2025年版)`

3. **Section Structure:**
   - Use `##` for major sections
   - Use `###` for subsections
   - Use `---` horizontal rules to separate major sections

4. **Content Guidelines:**
   - Write comprehensive, in-depth content (not short tutorials)
   - Target experienced software engineers
   - Include code examples with proper syntax highlighting
   - Use tables for structured information
   - Include "概要" (overview) section early in document

### File Naming Conventions

**Pattern:** `[topic]-[context]-[year?].mdx`

**Examples:**
- `ai-coding-agents-2025.mdx`
- `claude-code-features.mdx`
- `blazor-architecture-guide.mdx`
- `dotnet10-overview.mdx`

**Rules:**
- Use kebab-case (lowercase with hyphens)
- Include year suffix for time-sensitive content (`-2025`)
- Be descriptive but concise
- No special characters or spaces

### Root-Level Documentation

**Pattern:** `[topic]-[details]-[year].md`

**Examples:**
- `csharp-14-features.md`
- `microsoft-mcp-servers-2025.md`
- `mcp-servers-software-development-2025.md`

**Use Cases:**
- Detailed reference documentation (400+ lines)
- Standalone guides not requiring web layout
- Content that may be converted to MDX pages later

---

## Language and Writing Conventions

### Primary Language: Japanese

**All documentation content should be written in Japanese**, with the following exceptions:

**English Usage:**
- Technical terms (API, MCP, CLI, IDE, etc.)
- Product names (Claude Code, Astro, Blazor, Visual Studio)
- Code examples and syntax
- URLs and references
- Version numbers

**Writing Style:**
- **Formal/Technical:** Use polite but professional tone (です/ます form)
- **Comprehensive:** Provide in-depth explanations, not brief summaries
- **Structured:** Use clear hierarchical organization with numbered sections
- **Practical:** Include code examples, use cases, and real-world scenarios

**Example from existing content:**
```mdx
## 概要

Claude Codeは、AnthropicによるAI駆動の開発支援ツールです。
2025年9月に大規模なアップデートが行われ、より自律的で
強力な開発環境が提供されるようになりました。
```

### Year Indicators

Content should reference **2025** as the current year:
- Use `2025年版` in titles
- Date content as "2025" where relevant
- Focus on latest technologies and updates

---

## Creating New Documentation

### Adding a New MDX Page

1. **Create File in `src/pages/`:**
   ```bash
   touch src/pages/[topic-name]-2025.mdx
   ```

2. **Add Frontmatter and Structure:**
   ```mdx
   ---
   layout: ../layouts/Layout.astro
   title: [Your Title in Japanese]
   description: [Your Description in Japanese]
   ---

   # [Your Title in Japanese] (2025年版)

   ## 概要

   [Write overview in Japanese...]

   ## [Section 1]

   [Content...]
   ```

3. **Write Comprehensive Content:**
   - Target 200+ lines for substantial guides
   - Include multiple sections with clear hierarchy
   - Add code examples with syntax highlighting
   - Use tables for structured data
   - Include practical examples and use cases

4. **Test Locally:**
   ```bash
   npm run dev
   # Navigate to http://localhost:4321/[topic-name]-2025
   ```

5. **Build and Verify:**
   ```bash
   npm run build
   npm run preview
   ```

### Adding Root-Level Documentation

For extensive reference material (500+ lines):

1. **Create Markdown File:**
   ```bash
   touch [topic-name]-2025.md
   ```

2. **Write Comprehensive Guide:**
   - No frontmatter needed (not rendered as web page)
   - Use standard Markdown syntax
   - Include detailed technical information
   - Follow same language conventions (Japanese with English technical terms)

---

## TypeScript and Code Conventions

### TypeScript Configuration

**Strict Mode Enabled:**
```json
{
  "extends": "astro/tsconfigs/strict"
}
```

**Implications:**
- All code must pass strict type checking
- No implicit `any` types
- Null checks enforced
- Unused variables/imports flagged

### Code Style

**In MDX Files:**
```mdx
```bash
# Use language identifiers for syntax highlighting
npm run dev
```

```typescript
// TypeScript examples
const example: string = "Hello";
```

```python
# Python examples
def example():
    return "Hello"
```
```

**Best Practices:**
- Always specify language for code blocks
- Use proper indentation (2 spaces in TypeScript/JavaScript)
- Include comments in code examples for clarity

---

## Quality Standards

### Content Quality

**Each documentation page should:**
- Be comprehensive (200+ lines for major topics)
- Include multiple well-organized sections
- Provide practical examples and use cases
- Use proper Japanese grammar and style
- Include code examples where relevant
- Be technically accurate and up-to-date
- Reference official documentation when applicable

**Before Committing:**
1. ✅ Build succeeds (`npm run build`)
2. ✅ No TypeScript errors
3. ✅ Content is comprehensive and well-structured
4. ✅ Japanese language is grammatically correct
5. ✅ Code examples are tested and accurate
6. ✅ Links and references are valid

### Build Verification

**Always verify before committing:**
```bash
npm run build
```

**Expected Output:**
```
✓ Built successfully
```

**If Build Fails:**
- Check TypeScript errors
- Verify MDX syntax
- Ensure all imports are valid
- Check frontmatter YAML formatting

---

## Key Conventions for AI Assistants

### When Adding New Content

1. **Always Read Before Writing:**
   - Read existing similar files to understand patterns
   - Check commit history for naming conventions
   - Review frontmatter structure in existing MDX files

2. **Use Existing Patterns:**
   - Follow established file naming conventions
   - Match frontmatter structure exactly
   - Maintain consistent section hierarchy
   - Use same language style as existing content

3. **Write in Japanese:**
   - All body content in Japanese
   - Keep technical terms in English
   - Use formal/professional tone (です/ます)

4. **Be Comprehensive:**
   - Don't write short articles (aim for 200+ lines)
   - Include multiple sections and subsections
   - Provide detailed explanations and examples
   - Cover topic thoroughly, not superficially

5. **Test Before Committing:**
   ```bash
   npm run build  # Must succeed
   npm run preview  # Verify output
   ```

### When Modifying Existing Content

1. **Read the Entire File First:**
   - Understand current structure and style
   - Note the existing tone and depth
   - Identify section organization

2. **Maintain Consistency:**
   - Keep same writing style
   - Preserve existing structure
   - Don't change frontmatter unless necessary
   - Match language and formality level

3. **Preserve Quality:**
   - Don't reduce content depth
   - Keep comprehensive explanations
   - Maintain code example quality
   - Ensure accuracy of technical information

### Git Operations

1. **Use Correct Branch:**
   - Current branch: `claude/claude-md-mi9p1tbqgmggoov2-018oe3Ru8oDH3HbPEgY1342S`
   - Always push to this branch
   - Don't create new branches without instruction

2. **Commit Messages:**
   ```bash
   git commit -m "Add comprehensive [topic] guide in Japanese"
   ```

3. **Push with Retry Logic:**
   ```bash
   git push -u origin claude/[branch-name]
   # Retry up to 4 times with exponential backoff if network errors occur
   ```

4. **Before Creating PR:**
   - Ensure build succeeds
   - Verify all changes are committed
   - Check that content meets quality standards

---

## Common Tasks Reference

### Task: Add New Documentation Page

```bash
# 1. Create new MDX file
touch src/pages/my-new-topic-2025.mdx

# 2. Add frontmatter and content (use existing pages as template)
# Edit file in Japanese, following conventions

# 3. Test locally
npm run dev
# Visit http://localhost:4321/my-new-topic-2025

# 4. Build and verify
npm run build
npm run preview

# 5. Commit and push
git add src/pages/my-new-topic-2025.mdx
git commit -m "Add comprehensive [topic] guide in Japanese"
git push -u origin claude/claude-md-mi9p1tbqgmggoov2-018oe3Ru8oDH3HbPEgY1342S
```

### Task: Update Existing Documentation

```bash
# 1. Read existing file first
# Use Read tool on target file

# 2. Make changes maintaining style and structure

# 3. Test changes
npm run dev

# 4. Build and verify
npm run build

# 5. Commit and push
git add src/pages/[filename].mdx
git commit -m "Update [topic] guide with [changes]"
git push -u origin claude/claude-md-mi9p1tbqgmggoov2-018oe3Ru8oDH3HbPEgY1342S
```

### Task: Add Component or Layout

```bash
# 1. Create in appropriate directory
# Components: src/components/
# Layouts: src/layouts/

# 2. Write Astro component with TypeScript
# Follow strict type checking

# 3. Test in context
npm run dev

# 4. Build and verify
npm run build

# 5. Commit
git add src/[components|layouts]/[filename].astro
git commit -m "Add [component name] for [purpose]"
git push -u origin claude/claude-md-mi9p1tbqgmggoov2-018oe3Ru8oDH3HbPEgY1342S
```

---

## Dependencies and Tools

### Core Dependencies

```json
{
  "astro": "^5.15.7",
  "@astrojs/mdx": "^4.3.10"
}
```

### Recommended VSCode Extensions

- `astro-build.astro-vscode` (specified in `.vscode/extensions.json`)

### Node.js Configuration

- **Type:** ES Modules (`"type": "module"` in package.json)
- **Version:** Compatible with Astro 5.x (Node 18+)

---

## Troubleshooting

### Build Fails

**Check:**
1. TypeScript errors: `npm run astro check`
2. MDX syntax errors in frontmatter
3. Missing imports or invalid paths
4. Invalid YAML in frontmatter

**Common Issues:**
- Unclosed code blocks in MDX
- Invalid frontmatter YAML
- TypeScript strict mode violations
- Missing or incorrect layout path

### Dev Server Issues

**Problem:** Changes not reflecting

**Solution:**
1. Stop dev server (Ctrl+C)
2. Clear Astro cache: `rm -rf .astro/`
3. Restart: `npm run dev`

**Problem:** Port 4321 already in use

**Solution:**
```bash
# Kill existing process or use different port
npm run dev -- --port 3000
```

### Git Push Fails

**Problem:** 403 error when pushing

**Cause:** Branch doesn't follow required naming pattern

**Solution:**
- Ensure branch starts with `claude/`
- Ensure branch ends with matching session ID
- Use exact branch name: `claude/claude-md-mi9p1tbqgmggoov2-018oe3Ru8oDH3HbPEgY1342S`

**Problem:** Network errors

**Solution:**
- Retry up to 4 times with exponential backoff (2s, 4s, 8s, 16s)

---

## Contributors and History

**Main Contributors:**
- `ntaksh42` (primary maintainer)
- `Claude` (AI assistant)
- `GitHub Copilot` (AI assistant)

**Recent Activity:**
- 10+ commits in recent development cycle
- Active PR-based workflow
- Focus on 2025 technologies and updates
- Mix of AI/ML, MCP, and Microsoft ecosystem content

**Commit Pattern:**
- Consistent comprehensive documentation additions
- PR-based merges with numbered issues
- Clear, descriptive commit messages
- Regular updates and additions to knowledge base

---

## Summary for AI Assistants

**Primary Goals:**
1. Maintain high-quality, comprehensive Japanese documentation
2. Focus on cutting-edge 2025 technologies (AI, MCP, .NET)
3. Follow established patterns and conventions strictly
4. Ensure builds succeed before committing
5. Use proper git workflow with correct branch naming

**Key Principles:**
- **Comprehensive over brief:** Write detailed guides, not short articles
- **Japanese first:** All content in Japanese with English technical terms
- **Pattern consistency:** Follow existing file structures exactly
- **Quality standards:** Build must succeed, content must be thorough
- **Proper workflow:** Read → Write → Test → Build → Commit → Push

**Quick Checklist:**
- [ ] Content in Japanese (technical terms in English)
- [ ] Frontmatter matches existing pattern
- [ ] File naming follows kebab-case with optional year
- [ ] Comprehensive content (200+ lines for major topics)
- [ ] Code examples with proper syntax highlighting
- [ ] `npm run build` succeeds
- [ ] Commit message follows pattern
- [ ] Push to correct branch: `claude/claude-md-mi9p1tbqgmggoov2-018oe3Ru8oDH3HbPEgY1342S`

---

**Last Updated:** 2025-11-22
**Repository:** ntaksh42/myDocs
**Current Branch:** `claude/claude-md-mi9p1tbqgmggoov2-018oe3Ru8oDH3HbPEgY1342S`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ntaksh42)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ntaksh42)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
