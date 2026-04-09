## pieberrykinnie-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a semi-professional personal website built with Astro and configured for GitHub Pages deployment at `https://pieberrykinnie.github.io`. The site features:

- **Bio and Contact** - Homepage with personal information and contact details
- **Experience Section** - Background, skills, with sub-pages for Education, Work, and Extracurricular activities
- **Research** - Summary of research interests and activities
- **Projects** - Filterable project cards with detailed modals showing images, links, and descriptions
- **Coursework** - Course listings with optional writeup links to blog posts
- **Blog** - Personal blog posts and course-related writeups

**Technology Stack:**
- **Astro 5.14.1** - Static site generator with component-based architecture
- **Tailwind CSS 4.1.11** - Utility-first CSS framework
- **TypeScript** - Strict configuration extends `astro/tsconfigs/strict`
- **pnpm** - Package manager (version 10.11.0)

## Development Commands

All commands use pnpm as the package manager:

```bash
pnpm install          # Install dependencies
pnpm dev             # Start development server at localhost:4321
pnpm build           # Build production site to ./dist/
pnpm preview         # Preview build locally before deploying
pnpm astro ...       # Run Astro CLI commands (e.g., astro add, astro check)
```

## Architecture

### File Structure
```
src/
├── assets/          # Static assets (SVGs, images)
├── components/      # Reusable Astro components
│   ├── ui/          # General UI components
│   ├── layout/      # Layout-specific components
│   ├── projects/    # Project-related components (cards, modals)
│   └── blog/        # Blog-specific components
├── content/         # Content data files
│   ├── projects/    # Project data (JSON files)
│   ├── courses/     # Course data (JSON files)
│   └── blog/        # Blog posts (Markdown files)
├── layouts/         # Page layout templates
├── pages/           # File-based routing
│   ├── index.astro         # Homepage (bio & contact)
│   ├── experience.astro    # Experience overview
│   ├── experience/         # Experience sub-pages
│   │   ├── education.astro
│   │   ├── work.astro
│   │   └── extracurricular.astro
│   ├── research.astro      # Research interests
│   ├── projects.astro      # Projects with filtering
│   ├── coursework.astro    # Course listings
│   └── blog/               # Blog section
│       └── index.astro     # Blog listing page
└── styles/          # Global CSS and Tailwind imports
```

### Site Structure & Routing
- **/** - Homepage with bio and contact information
- **/experience** - Experience overview with links to sub-pages
  - **/experience/education** - Educational background
  - **/experience/work** - Work experience 
  - **/experience/extracurricular** - Extracurricular activities
- **/research** - Research interests and activities
- **/projects** - Project showcase with filtering and modal details
- **/coursework** - Course listings with optional blog writeup links
- **/blog** - Blog posts and writeups

### Content Management
- **Projects**: JSON files in `src/content/projects/` with schema for title, category, description, technologies, GitHub/deployed URLs, images
- **Courses**: JSON files in `src/content/courses/` with course details and optional writeup links
- **Blog Posts**: Markdown files in `src/content/blog/` with frontmatter for metadata

### Styling Architecture
- **Global CSS**: `src/styles/global.css` imports Tailwind CSS
- **Component Styles**: Scoped `<style>` blocks in `.astro` components
- **Tailwind Integration**: Configured via Vite plugin in `astro.config.mjs`

### Astro Configuration
- Site URL configured for GitHub Pages deployment
- Tailwind CSS integrated via `@tailwindcss/vite` plugin
- TypeScript strict mode enabled

## Key Development Notes

- Uses Astro's component-based architecture with `.astro` files
- File-based routing: files in `src/pages/` become site routes
- Content stored as JSON (projects/courses) and Markdown (blog) in `src/content/`
- Project cards should include filtering/categorization functionality
- Project modals should display images, descriptions, and links
- Course pages should link to related blog writeups
- Static assets are processed and optimized by Astro
- GitHub Pages deployment configured via `site` setting in astro.config.mjs

## Content Schema Examples

### Project JSON Schema
```json
{
  "title": "Project Name",
  "category": "Web Development",
  "description": "Brief description",
  "detailedDescription": "Detailed technical description",
  "technologies": ["Tech1", "Tech2"],
  "githubUrl": "https://github.com/user/repo",
  "deployedUrl": "https://example.com",
  "images": ["/images/projects/project-1.png"],
  "featured": true,
  "date": "2024-01-01"
}
```

### Course JSON Schema  
```json
{
  "title": "Course Name",
  "code": "CS 101", 
  "institution": "University",
  "semester": "Fall 2024",
  "description": "Course description",
  "grade": "A",
  "writeups": [
    {
      "title": "Writeup Title",
      "url": "/blog/writeup-slug",
      "description": "Writeup description"
    }
  ]
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pieberrykinnie)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pieberrykinnie)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
