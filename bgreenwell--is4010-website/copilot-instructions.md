## is4010-website

> This repository contains the source code for the official course website for **IS 4010: Application Development with Artificial Intelligence**, taught by Brandon M. Greenwell, PhD, at the University of Cincinnati.

# IS 4010: Application Development with Artificial Intelligence - Website Repository

This repository contains the source code for the official course website for **IS 4010: Application Development with Artificial Intelligence**, taught by Brandon M. Greenwell, PhD, at the University of Cincinnati.

## Directory Overview

This is a **Quarto-based website project**. It uses Quarto Markdown (`.qmd`) files to generate a static website and Reveal.js presentation slides.

- **`index.qmd`**: The landing page of the course website.
- **`syllabus.qmd`**: The comprehensive course syllabus, including policies, grading, and schedule.
- **`project.qmd`**: Details regarding the final solo capstone project.
- **`weeks/`**: Contains weekly module pages (`week01.qmd` through `week14.qmd`) that outline the curriculum.
- **`slides/`**: Contains Quarto source files for Reveal.js lecture slides.
- **`resources/`**: Supplemental guides for students (setup, troubleshooting, packages).
- **`_quarto.yml`**: The main configuration file for the website's structure, navigation, and theme.
- **`styles.css`**: Custom CSS for the website.
- **`.github/workflows/publish.yml`**: GitHub Actions workflow for automated rendering and deployment to GitHub Pages.

## Key Files

- **`_quarto.yml`**: Define the sidebar, navbar, and global site metadata here.
- **`syllabus.qmd`**: The "source of truth" for course dates and policies.
- **`slides/*.qmd`**: Individual slide decks rendered as Reveal.js presentations.
- **`AGENTS.md`** (in parent or root): Contains specific instructions for AI agents regarding tone (AI Co-Pilot), casing (Sentence Case), and security.

## Usage & Development

### Local Development
To preview the website locally as you make changes:
```bash
quarto preview
```

### Rendering
To render the entire site to the `_site/` (or `docs/`) directory:
```bash
quarto render
```

To render only the slides to a specific directory (e.g., for Canvas upload):
```bash
cd slides
quarto render . --output-dir html
```

### Deployment
Deployment is automated. Any push to the `main` branch triggers a GitHub Action that renders the site and publishes it to the `gh-pages` branch.

## Development Conventions

- **Tone**: Maintain an encouraging, professional, and practical "AI Co-Pilot" tone.
- **Formatting**: Use **Sentence case** for all titles, headings, and subheadings (e.g., "Course overview" instead of "Course Overview").
- **Consistency**: Ensure code examples and terminology (Python vs. Rust) align with the weekly modules.
- **Security**: Never commit sensitive instructor materials or API keys to this repository.

---
> Source: [bgreenwell/is4010-website](https://github.com/bgreenwell/is4010-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
