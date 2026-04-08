## paulinachacin-com

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Jekyll-based GitHub Pages portfolio website for **Paullina Chacin** (born 2008), a high school junior preparing for college applications. The site serves as her academic and extracurricular portfolio, showcasing volunteering, events, activities, skills, and resume.

## Key Information

- **Name**: Paullina Chacin
- **Purpose**: College preparation portfolio and contact hub
- **Technology**: Jekyll static site generator with GitHub Pages
- **Theme**: Custom theme based on Catppuccin color scheme (Latte for light mode, Macchiato for dark mode)
- **Style**: Modern, academic, professional

## Design Requirements

### Theme System
- **Dark/Light/Auto Modes**: Implement theme switcher with three options
- **Color Scheme**: Use Catppuccin Latte (light) and Macchiato (dark) palettes
- **Reference Sites**: Follow design patterns from carloschac.in and vanilla-di.com
- **Accessibility**: Ensure WCAG compliance and readable contrast ratios

### Content Structure

The site should include these main sections:

1. **Home/Landing Page**: Introduction and highlight reel
2. **About/Contact**: Skills, bio, contact information (email/phone to be added later)
3. **Resume**:
   - Web-based resume page
   - PDF download button
   - Site serves as single source of truth for resume content
4. **Volunteering**: Showcase volunteer work and community service
5. **Events**: Participation in academic and extracurricular events
6. **Activities**: Extra-curricular activities and achievements

## Development Commands

### Local Development
```bash
# Install dependencies
bundle install

# Serve locally with live reload
bundle exec jekyll serve

# Serve with drafts
bundle exec jekyll serve --drafts

# Build for production
bundle exec jekyll build
```

### Common Tasks
```bash
# Clear cache and rebuild
bundle exec jekyll clean && bundle exec jekyll build

# Check for broken links (if html-proofer is added)
bundle exec htmlproofer ./_site
```

## Architecture Notes

### Jekyll Structure
- **_config.yml**: Site configuration, metadata, build settings
- **_posts/**: Blog posts (if blog functionality is needed)
- **_layouts/**: Page templates and structure
- **_includes/**: Reusable components (header, footer, navigation)
- **_sass/**: Styling with Sass/SCSS (use for Catppuccin theme)
- **assets/**: Static files (CSS, JS, images, PDFs)

### Theme Implementation Strategy
- Override Minima theme by creating custom layouts in `_layouts/`
- Create theme switcher using JavaScript in `assets/js/`
- Define Catppuccin color variables in `_sass/` using CSS custom properties
- Use `prefers-color-scheme` media query for auto mode

### Resume Page
- Create dedicated layout for resume with print-friendly CSS
- Store resume content in structured data (YAML in `_data/resume.yml`)
- Implement PDF generation or provide downloadable PDF in `assets/`
- Keep web version synchronized with PDF version

## Configuration Notes

### _config.yml Updates Needed
- Update `title`, `email`, `description` with Paullina's information
- Configure navigation menu with new pages
- Set up collections if organizing content by type (volunteering, events, etc.)
- Add theme configuration for dark/light mode persistence

### Important Behaviors
- After editing `_config.yml`, restart Jekyll server (changes not hot-reloaded)
- GitHub Pages builds automatically on push to main branch
- Test locally before pushing to ensure builds succeed

## Content Management

### Adding New Content
- **Pages**: Create `.md` files in root with YAML front matter
- **Collections**: Define in `_config.yml` for structured content types
- **Data Files**: Use `_data/` directory for structured content (resume, skills, etc.)
- **Images**: Store in `assets/images/` with descriptive names

### Front Matter Template
```yaml
---
layout: page
title: Page Title
permalink: /url-path/
---
```

## Styling Guidelines

### Catppuccin Implementation
- **Latte (Light)**: Base: #eff1f5, Text: #4c4f69, Accent colors from Latte palette
- **Macchiato (Dark)**: Base: #24273a, Text: #cad3f5, Accent colors from Macchiato palette
- Maintain consistent spacing, typography, and component styles
- Use semantic color naming (e.g., `--color-primary`, `--color-background`)

### Academic & Professional Tone
- Clean typography with readable font sizes
- Ample whitespace and clear hierarchy
- Professional color choices from Catppuccin palette
- Responsive design for mobile/tablet/desktop

## Privacy & Security
- Email and phone number to be added later (not in initial commit)
- Keep sensitive information in environment variables or separate config
- Review all commits before pushing to public repository

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cchacin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cchacin)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
