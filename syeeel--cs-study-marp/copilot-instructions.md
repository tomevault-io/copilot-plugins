## cs-study-marp

> This is a **Marp-based presentation repository** for a Japanese computer science study session titled "データサイエンティストのためのソフトウェア工学入門" (Introduction to Software Engineering for Data Scientists).

# CLAUDE.md - AI Assistant Guide

## Project Overview

This is a **Marp-based presentation repository** for a Japanese computer science study session titled "データサイエンティストのためのソフトウェア工学入門" (Introduction to Software Engineering for Data Scientists).

**Primary Purpose:** Create and maintain presentation materials in multiple formats (HTML, PDF, PPTX) using Markdown and Marp CLI.

**Language:** Japanese (ja)
**Framework:** Marp (Markdown Presentation Ecosystem)
**Primary Output:** GitHub Pages hosted HTML presentation

---

## Repository Structure

```
cs-study-marp/
├── .devcontainer/           # VS Code devcontainer configuration
│   └── devcontainer.json    # Node.js 22 + Chromium setup
├── .github/                 # GitHub configuration
│   └── dependabot.yml       # Dependabot configuration
├── docs/                    # Build output for GitHub Pages
│   ├── images/              # Copied images for deployment
│   └── index.html           # Generated HTML presentation
├── images/                  # Source images for presentations
│   ├── devcontainer.png
│   ├── space-rocket.jpg
│   ├── study-cs-top.jpeg
│   └── wheel.jpeg
├── src/                     # Source files
│   ├── images/              # Additional images (duplicates from root)
│   ├── CS勉強会.md          # Main presentation source (612 lines)
│   └── CS勉強会.pptx        # Generated PowerPoint file
├── .gitignore               # Standard Node.js gitignore
├── package.json             # NPM dependencies (@marp-team/marp-cli)
├── package-lock.json        # Lock file
└── README.md                # Project documentation
```

---

## Technology Stack

### Core Dependencies
- **@marp-team/marp-cli** (^4.1.0) - Markdown presentation CLI tool
- **Mermaid** (11.4.1) - Diagram and charting tool
- **MathJax** - Mathematical notation rendering
- **Chromium** - Required for PDF/PPTX generation

### Development Environment
- **Node.js 22** (Bookworm)
- **DevContainer** support for consistent development environments
- **VS Code Extensions:**
  - `marp-team.marp-vscode` - Marp preview support
  - `ms-vscode.live-server` - Live server for testing
  - Other dev tools (ESLint, Prettier, Todo Tree, etc.)

---

## Build Commands

### 1. HTML Generation (GitHub Pages)
```bash
npx marp src/CS勉強会.md --html --allow-local-files --theme-set ./style/ -o docs/index.html --embed-cs
```
**Purpose:** Generates `docs/index.html` for GitHub Pages deployment
**Flags:**
- `--html` - Enable HTML rendering
- `--allow-local-files` - Allow local file references
- `--theme-set ./style/` - Custom theme directory (if exists)
- `-o docs/index.html` - Output to docs folder
- `--embed-cs` - Embed custom CSS

### 2. PDF Generation
```bash
npx marp src/CS勉強会.md --pdf --allow-local-files -o docs/index.pdf
```
**Purpose:** Creates a PDF version of the presentation
**Requirements:** Chrome/Chromium must be installed

### 3. PPTX Generation (Editable PowerPoint)
```bash
marp --pptx --pptx-editable CS勉強会.md
```
**Purpose:** Creates an editable PowerPoint file
**Requirements:**
- Chrome or LibreOffice installed
- Best executed locally (not in CI/CD)
- LibreOffice install: `brew install libreoffice`

---

## File Naming Conventions

### Japanese File Names
- Main presentation: `CS勉強会.md` (CS Study Session)
- **IMPORTANT:** Always preserve exact Japanese characters when referencing or modifying files
- File encoding: UTF-8

### Directory Conventions
- Source files: `src/`
- Build output: `docs/` (for GitHub Pages)
- Static assets: `images/` (root level)
- Duplicated assets: `src/images/` (for relative references)

---

## Content Structure

### Markdown Front Matter
Every presentation file includes YAML front matter:
```yaml
---
marp: true
theme: default
paginate: true
lang: "ja"
header: "データサイエンティストのためのソフトウェア工学入門"
footer: "©2025 Satoshi Yoshimura"
style: |
  /* Inline CSS styles */
math: mathjax
mermaid: true
---
```

### Key Styling Elements
1. **Font Family:** "Noto Sans JP", "Hiragino Kaku Gothic ProN", "Hiragino Sans", Meiryo
2. **Primary Color:** `#230ee0` (for headings)
3. **Font Sizes:**
   - H1: 40px
   - H2: 32px
   - Body: 24px
   - Lists: 22px
   - Nested lists: 20px
4. **Image Default Width:** 400px (centered)

### Mermaid Integration
Mermaid diagrams are embedded using:
```html
<pre class="mermaid">
<!-- Mermaid code here -->
</pre>

<script type="module">
import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11.4.1/dist/mermaid.esm.min.mjs';
mermaid.initialize({ startOnLoad: true });
</script>
```

### Slide Separators
- Use `---` to create new slides
- Each `---` represents a slide break in Marp

---

## Development Workflow

### Initial Setup
1. **Clone repository**
2. **Install dependencies:** `npm install`
3. **DevContainer (optional):**
   - Opens in VS Code with all extensions pre-configured
   - Auto-installs Chromium via `postCreateCommand`
   - Sets `CHROME_PATH` and `PUPPETEER_EXECUTABLE_PATH`

### Editing Workflow
1. **Edit:** Modify `src/CS勉強会.md`
2. **Preview:** Use Marp VS Code extension for live preview
3. **Build:** Run appropriate build command (HTML/PDF/PPTX)
4. **Test:** Check output in `docs/` directory
5. **Commit:** Commit both source and generated files

### Environment Variables (DevContainer)
- `CHROME_PATH=/usr/bin/chromium`
- `PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium`

---

## Git Conventions

### Commit History Analysis
Recent commits show:
- `153bd8a` - Edit gitignore file
- `d4827af` - Add PowerPoint convert method
- `1aab751` - Fix tiny code
- `a660b36` - Edit some image URL
- `95e57c8` - Edit some image URL

### Commit Message Patterns
- Use English for commit messages
- Use imperative mood: "Add", "Edit", "Fix"
- Be concise and descriptive
- Examples: "Add Power point convert method", "Fix tiny code"

### Branching Strategy
- Main branch for stable releases
- Feature branches following pattern: `claude/claude-md-*` for AI-assisted development
- Always push to designated feature branch during development

---

## Important Notes for AI Assistants

### 1. File Path Handling
- **CRITICAL:** Use absolute paths: `/home/user/cs-study-marp/...`
- Japanese characters in file names must be preserved exactly
- Main source file: `/home/user/cs-study-marp/src/CS勉強会.md`

### 2. Build Process
- Always build to `docs/` directory for GitHub Pages
- HTML builds require `--embed-cs` flag for custom styles
- PDF/PPTX generation requires Chromium/LibreOffice

### 3. Image Management
- Images exist in two locations: `images/` and `src/images/`
- When adding new images:
  - Place in `images/` directory
  - May need to copy to `src/images/` for relative references
  - Update references in markdown
  - Rebuild HTML to include in `docs/images/`

### 4. Content Modifications
- **Language:** All content is in Japanese - preserve language consistency
- **Styling:** Inline CSS in front matter - prefer editing there over external stylesheets
- **Mermaid:** Always include the script import block when using Mermaid diagrams
- **Line Length:** Main file is 612 lines - consider readability when editing

### 5. Testing Outputs
- **HTML:** Can be previewed with live-server or direct file opening
- **PDF:** Requires Chromium - verify installation before running
- **PPTX:** Best tested locally with LibreOffice or PowerPoint

### 6. DevContainer Awareness
- If working in devcontainer, Chromium is pre-installed
- VS Code extensions are auto-configured
- Environment variables are set for Puppeteer
- PostCreateCommand runs `npm install` automatically

### 7. Dependencies Management
- Only one production dependency: `@marp-team/marp-cli`
- Dependabot configured for devcontainer updates (weekly)
- No separate npm scripts defined - use `npx marp` directly

### 8. GitHub Pages Deployment
- Deployment source: `docs/index.html`
- Images must be copied to `docs/images/`
- All assets should be embedded or relatively referenced

### 9. Common Tasks

**Adding a new slide:**
```markdown
---

# New Slide Title

Content here...
```

**Adding an image:**
```markdown
<img src="../images/image-name.jpg" alt="description" style="width: 400px;">
```

**Adding a Mermaid diagram:**
```html
<pre class="mermaid">
graph LR
    A --> B
</pre>
```

**Rebuilding for deployment:**
```bash
npx marp src/CS勉強会.md --html --allow-local-files -o docs/index.html --embed-cs
```

### 10. What NOT to Do
- ❌ Don't create new style directories - styles are inline
- ❌ Don't remove the Mermaid script import
- ❌ Don't change Japanese characters without explicit instruction
- ❌ Don't use English in content unless requested
- ❌ Don't skip the `--embed-cs` flag when building HTML
- ❌ Don't commit `node_modules/` (already in `.gitignore`)
- ❌ Don't create GitHub Actions without discussing first (none currently exist)

---

## Quick Reference

### File Locations
| Purpose | Path |
|---------|------|
| Main presentation source | `/home/user/cs-study-marp/src/CS勉強会.md` |
| HTML output | `/home/user/cs-study-marp/docs/index.html` |
| Source images | `/home/user/cs-study-marp/images/` |
| DevContainer config | `/home/user/cs-study-marp/.devcontainer/devcontainer.json` |

### Common Commands
| Task | Command |
|------|---------|
| Install dependencies | `npm install` |
| Build HTML | `npx marp src/CS勉強会.md --html --allow-local-files -o docs/index.html --embed-cs` |
| Build PDF | `npx marp src/CS勉強会.md --pdf --allow-local-files -o docs/index.pdf` |
| Build PPTX | `marp --pptx --pptx-editable src/CS勉強会.md` |

### VS Code Extensions
- Marp for VS Code (`marp-team.marp-vscode`)
- Live Server (`ms-vscode.live-server`)

---

## Troubleshooting

### Build Issues
1. **PDF/PPTX generation fails:** Verify Chromium is installed
   ```bash
   which chromium
   # Should output: /usr/bin/chromium
   ```

2. **Images not showing in HTML:** Check that images are copied to `docs/images/`

3. **Mermaid not rendering:** Ensure script import is present in markdown

4. **Japanese characters broken:** Verify file encoding is UTF-8

### DevContainer Issues
1. **Extensions not loading:** Check `.devcontainer/devcontainer.json` configuration
2. **Chromium not found:** Rebuild container to trigger `postCreateCommand`

---

## Version Information

**Last Updated:** 2025-11-15
**Marp CLI Version:** 4.1.0
**Node.js Version:** 22 (Bookworm)
**Mermaid Version:** 11.4.1
**Content Length:** 612 lines in main presentation

---

## Additional Resources

- [Marp Documentation](https://marpit.marp.app/)
- [Mermaid Documentation](https://mermaid.js.org/)
- [Devcontainer Specification](https://containers.dev/)
- Original README: `/home/user/cs-study-marp/README.md`

---

*This document is maintained for AI assistants working on this repository. Keep it updated when making significant structural changes.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/syeeel)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/syeeel)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
