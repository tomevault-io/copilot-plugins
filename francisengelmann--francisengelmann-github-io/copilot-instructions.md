## francisengelmann-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a personal academic website (GitHub Pages) for Francis Engelmann, generated from YAML data files and HTML templates. The site showcases research publications, news, and professional information.

## Build System

The website is generated using a Python script that processes YAML data and HTML templates:

```bash
# Generate the website (outputs to index.html)
python generate_page.py

# Install dependencies
pip install -r requirements.txt
```

The GitHub Actions workflow (`.github/workflows/main.yml`) automatically runs `generate_page.py` on every push to master and deploys to GitHub Pages.

## Architecture

### Core Generation Script: `generate_page.py`

The main script assembles the final HTML page from modular components:

1. **Data Sources** (YAML files in `data/`):
   - `publications.yaml` - Research papers with metadata (title, authors, venue, links, teasers)
   - `news.yaml` - News items with dates and markdown-formatted text
   - `authors.yaml` - Author name to website URL mappings
   - `conferences.yaml` - Conference abbreviation to full name mappings

2. **Templates** (HTML fragments in `templates/`):
   - `header.html` - Meta tags, CSS, fonts, and styles
   - `bio.html` - Profile photo and biographical information
   - `footer.html` - Footer with navigation and JavaScript includes

3. **Generation Functions**:
   - `get_news()` - Parses news YAML, converts markdown to HTML, adds collapsible "Previous news" section after 10 items
   - `get_publications()` - Processes publications with automatic BibTeX generation, author linking, and award badges
   - `generate_page()` - Assembles all components into final HTML output

### Key Features

- **Automatic BibTeX Generation**: BibTeX entries are generated from publication metadata using format `{FirstAuthorLastname}{Year}{FirstTitleWord}`
- **Author Linking**: Author names are automatically hyperlinked based on `authors.yaml`
- **Shared Authorship**: Authors marked with `*` in YAML get superscript asterisks
- **Award Highlighting**: Awards shown as green badges with optional secondary text
- **Markdown Support**: News items support markdown with GitHub Flavored Markdown extensions
- **Ordinal Superscripts**: Date ordinals (1st, 2nd, etc.) automatically converted to superscripts in news

## Project Structure

```
├── generate_page.py       # Main generation script
├── requirements.txt       # Python dependencies (markdown, pyyaml)
├── data/                  # YAML data sources
│   ├── publications.yaml  # Research papers
│   ├── news.yaml         # News items
│   ├── authors.yaml      # Author links
│   └── conferences.yaml  # Conference names
├── templates/            # HTML fragments
│   ├── header.html       # Page head with CSS
│   ├── bio.html         # Biography section
│   └── footer.html      # Footer with scripts
├── teasers/             # Publication teaser images
├── pyviz3d_examples/    # Demo pages for PyViz3D library
├── index.html           # Generated output (main page)
└── .github/workflows/   # CI/CD for GitHub Pages
```

## Making Changes

### Adding a Publication

1. Add entry to `data/publications.yaml`:
```yaml
- title: "Your Paper Title"
  authors: "Author One, Author Two*, Francis Engelmann"  # * for shared first authorship
  conference: "CVPR"  # Must exist in conferences.yaml
  year: 2025
  paper: "https://arxiv.org/abs/..."
  project: "https://yourproject.github.io/"  # optional
  code: "https://github.com/..."  # optional
  award: "Oral Presentation, top 3%"  # optional, comma-separated for badge + text
  teaser: "teasers/your_teaser.jpg"
```

2. Place teaser image in `teasers/` directory
3. Run `python generate_page.py` to regenerate the site

### Adding News

Add entry to `data/news.yaml` (newest first):
```yaml
- date: December 2025
  text: "Your news with [markdown links](http://example.com) and **bold text**"
```

### Modifying Bio or Styling

- Edit `templates/bio.html` for biographical content
- Edit `templates/header.html` for CSS styling
- Changes require running `generate_page.py` to take effect

### Testing Locally

```bash
# Generate the page
python generate_page.py

# Serve locally (Python 3)
python -m http.server 8000

# Visit http://localhost:8000/index.html
```

## Important Notes

- The generated file is `index.html` (note: there's also an `index2.html` which appears to be a test/alternative version)
- All changes to YAML or templates require regenerating via `generate_page.py`
- The site uses Bootstrap 4.4.1, Font Awesome 6.7.2, and Academicons
- Publication teasers should be 160px wide (aspect ratio maintained)
- News items after the 10th are automatically collapsed under "Previous news"
- BibTeX entries use the format: `{LastName}{Year}{FirstWord}` (automatically cleaned of special characters)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/francisengelmann)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/francisengelmann)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
