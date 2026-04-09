## arunperiyal-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal academic website for Arun Periyal, a Naval Architect and Ocean Engineer specializing in nonlinear dynamics, fluid-structure interaction, machine learning, and naval architecture. The site is a static HTML/CSS website hosted on GitHub Pages.

## Repository Structure

The website follows a hierarchical structure:

- **Root directory**: Main pages (index.html, research.html, materials.html, about.html, etc.)
- **research/**: Research area pages organized by topic
  - `research/nonlinear_dynamics.html`
  - `research/fluid_structure_interaction.html`
  - `research/naval_architecture.html`
  - `research/machine_learning.html`
  - Subdirectories for detailed research topics (e.g., `research/fluid_structure_interaction/`)
- **materials/**: Code and materials pages by subject area
  - `materials/nonlinear-dynamics-codes.html`
  - `materials/fluid-structure-codes.html`
  - `materials/naval-architecture-codes.html`
  - `materials/machine-learning-codes.html`
- **cv/**: LaTeX CV source files
  - `cv/main.tex` - Main CV LaTeX source
  - `cv/developercv.cls` - Custom LaTeX class file
  - `cv/main.pdf` - Compiled PDF output
- **trial/**: Experimental/testing HTML files
- **backups/**: Historical backups organized by date

## Development Commands

### Building CV
To compile the LaTeX CV to PDF:
```bash
cd cv
pdflatex main.tex
```

### Git Operations
A convenience script is available for git operations:
```bash
./commit.sh "commit message"
```
This script runs `git add -A`, `git commit`, and `git push origin main`.

### Local Development
Since this is a static HTML site, you can:
- Open HTML files directly in a browser for quick testing
- Use a local web server for more accurate testing:
  ```bash
  python3 -m http.server 8000
  ```
  Then navigate to `http://localhost:8000`

## CSS Architecture

### New Consolidated CSS Structure (2024)
The site now uses a modular CSS system:
- **css/main.css**: Shared styles for all pages (CSS variables, typography, cards, layouts)
- **css/homepage.css**: Homepage-specific styles (carousel, sidebar, fixed header)
- **Legacy files** (to be migrated): `styles.css`, `index-style.css`

### CSS Variables
All colors are defined as CSS variables in `css/main.css`:
```css
:root {
    --primary-dark: #142f43;
    --accent-gold: #f4d35e;
    --bg-white: #ffffff;
    --bg-light: #f4f4f9;
    --text-dark: #333333;
    --text-medium: #555555;
}
```

### Color Scheme
- **Primary dark**: `#142f43` (dark blue, used in headers and navigation)
- **Accent**: `#f4d35e` (golden yellow, used for links and highlights)
- **Background**: `#ffffff` (white) or `#f4f4f9` (light gray)
- **Text**: `#333333` (dark gray) or `#555555` (medium gray)

### Common UI Patterns

**Header with Background Image:**
All main pages use a hero header with a cover image overlay:
```css
background: url('https://raw.githubusercontent.com/arunperiyal/arunperiyal.github.io/main/figs/cover_image.jpg')
```
With a dark overlay (`rgba(0, 0, 0, 0.5)`) for text readability.

**Navigation Bar:**
Consistent horizontal navigation with centered links using flexbox.

**Card-based Layout:**
Research and materials pages use responsive card grids:
- Grid layout: `grid-template-columns: repeat(auto-fit, minmax(250px, 1fr))`
- Cards have hover effects with `translateY(-5px)` and enhanced shadows

**Flip Cards:**
The research.html page uses 3D flip card effects for topic navigation.

**Carousel:**
index.html features an auto-rotating carousel (3-second intervals) showcasing current works.

### Typography
- Font family: 'Roboto' from Google Fonts
- Font Awesome icons for social media and visual elements

## Page Architecture

### Homepage (index.html)
- **CSS**: `css/main.css` + `css/homepage.css`
- Features: fixed header, sidebar toggle, carousel, area cards, contact footer
- Sidebar toggle JavaScript function at bottom of page
- Carousel auto-rotation with manual controls

### Research Pages
Research pages follow two patterns:
1. **Overview pages** (research.html): Uses flip cards to navigate to specific research areas
2. **Detail pages** (research/nonlinear_dynamics.html): Currently use inline styles (migration to `css/main.css` recommended)
   - Use `.research-box`, `.topics`, `.topic-card` classes from main.css

### Materials Pages
Materials pages showcase code repositories and scripts:
1. **Overview page** (materials.html): Card grid linking to specific material categories
2. **Detail pages** (materials/nonlinear-dynamics-codes.html): Currently use inline styles (migration to `css/main.css` recommended)
   - Use `.code-section`, `.code-grid`, `.code-card`, `.language-tag` classes from main.css

### Navigation Consistency
All pages should include navigation to:
- Home (index.html)
- Research (research.html)
- Materials (materials.html)
- About Me (about.html)

Research and materials subpages use relative paths (e.g., `../index.html`).

## Content Guidelines

### Academic Focus
Content centers on four main research areas:
1. **Naval Architecture**: Ship design, optimization, and analysis
2. **Fluid Structure Interaction**: VIV (Vortex-Induced Vibration), energy harvesting, marine risers
3. **Nonlinear Dynamics**: Bifurcation analysis, chaos theory, stability analysis, perturbation methods
4. **Machine Learning**: Reduced-order models, PINNs, predictive modeling

### Code Examples
Materials pages document various computational tools:
- Language tags for Python, MATLAB, NumPy, SciPy, SymPy, Jupyter
- Links to GitHub repositories (https://github.com/arunperiyal)
- Descriptions focusing on technical capabilities

## File Naming Conventions

### Current State (Mixed - In Transition)
**Inconsistencies exist** - see `FILE_NAMING_STANDARD.md` for full details:
- Root pages: Mixed (e.g., `linux_tools.html` uses underscores)
- Research pages: Underscores (e.g., `nonlinear_dynamics.html`)
- Materials pages: Hyphens ✓ (e.g., `nonlinear-dynamics-codes.html`)

### Target Standard: Kebab-case (Hyphens)
**All new files should use:**
- **kebab-case** (hyphens) for multi-word files: `my-new-page.html`
- **lowercase** only (no capitals)
- **descriptive** names (avoid abbreviations)
- **Exception**: Single-word files don't need separators (`index.html`, `about.html`)

**Rationale**: Better for SEO, industry standard, more readable URLs

### Planned Migrations
See `FILE_NAMING_STANDARD.md` for:
- Complete list of files to rename
- Git commands for preserving history
- Link update requirements

## External Resources

### Assets
- Cover images hosted on GitHub raw: `https://raw.githubusercontent.com/arunperiyal/arunperiyal.github.io/main/figs/`
- Font Awesome CDN: `https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css`
- Google Fonts: `https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;700&display=swap`

### External Links
- LinkedIn: https://www.linkedin.com/in/arunpvenu
- GitHub: https://github.com/arunperiyal
- Email: periyal.arun@outlook.com

## Responsive Design Considerations

- Use `max-width: 1200px` or `max-width: 1000px` for main content areas
- Grid layouts use `auto-fit` with `minmax()` for responsive columns
- Mobile-first approach with viewport meta tag: `<meta name="viewport" content="width=device-width, initial-scale=1.0">`

## JavaScript Usage

Minimal JavaScript is used:
- Sidebar toggle functionality on index.html
- Carousel auto-rotation and manual controls
- Auto-rotating carousel on research.html
- Inline scripts at bottom of HTML files

## LaTeX/CV Notes

The CV uses a custom class `developercv.cls` with 9pt font size. When editing the CV:
- Main content is in `cv/main.tex`
- Contact information uses minipage layout
- Sections use custom `\cvsect{}` command
- Entries use `\entry{}{}{}{}` format from the custom class
- Compile with `pdflatex main.tex` (may need multiple runs for references)

## Git Branch

- Main branch: `main`
- All changes should be committed to and pushed to the `main` branch

## Security and Best Practices

### .gitignore
A comprehensive `.gitignore` file is in place to prevent:
- SSH keys and credentials
- Editor swap files
- LaTeX auxiliary files
- System files
- Backup directories
- Node modules (if using build tools)

### File Exclusions
**Never commit:**
- Private keys (`.pem`, `.key`)
- SSH keys (`*.pub`, key files)
- Environment files (`.env`)
- Editor temporary files (`*.swp`, `*~`)
- LaTeX build artifacts (`*.aux`, `*.log`, `*.out`)

### Documentation
- `CLAUDE.md` - This file, architecture and conventions
- `CSS_MIGRATION_GUIDE.md` - CSS consolidation instructions
- `FILE_NAMING_STANDARD.md` - File naming conventions and migration plan

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arunperiyal)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/arunperiyal)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
