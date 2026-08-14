## duv0-x-github-io

> This document provides comprehensive guidance for AI assistants working on the duv0-x.github.io repository.

# CLAUDE.md - AI Assistant Guide

This document provides comprehensive guidance for AI assistants working on the duv0-x.github.io repository.

## Repository Overview

**Type**: Personal Portfolio Website (GitHub Pages)
**Owner**: Duván Ballén (duv0_x)
**Tech Stack**: Static HTML/CSS
**Purpose**: Professional profile and activity log showcasing real daily work across SRE/DevOps, cloud engineering, personal projects, and AI tooling.
**Theme**: Retro pixel-art / terminal aesthetic with dark color scheme; log entries rendered as terminal lines with `>>` prompt

## Codebase Structure

```
duv0-x.github.io/
├── index.html          # Main landing page (single-page site)
├── styles.css          # All styling and theme definitions
├── README.md           # Repository description
├── .gitignore          # Git ignore patterns
├── files/              # CV/Resume PDFs in multiple versions
│   ├── CV_Duvan_Ballen_SRE_DevOps.pdf
│   ├── CV_Duvan_Ballen_SRE_DevOps_EN.pdf
│   ├── CV_Duvan_Ballen_SRE_DevOps_EN_v2.pdf
│   └── CV_Duvan_Ballen_SRE_DevOps_v2.pdf
└── images/             # Profile pictures and visual assets
    ├── duv0-x-px.png
    ├── duv0-x-small.jpg
    ├── duv0-x.jpg
    └── x-0vub-small.jpg
```

## Design & Styling Conventions

### Color Palette
The site uses a **consistent dark theme** with specific brand colors:

- **Background**: `#000` (black body) and `#0c1015` (container backgrounds)
- **Primary Accent**: `#27a888` (teal/green - used for headers)
- **Secondary Accent**: `#98d1cc` (light teal - used for text and borders)
- **Link Colors**:
  - `#e06c76` (coral/pink - LinkedIn)
  - `#98d1cc` (light teal - GitHub, Mastodon)

**CRITICAL**: When making any color changes:
1. Maintain visual consistency across the site
2. Ensure sufficient contrast for readability
3. Test on both light and dark backgrounds
4. Update all instances of a color if changing brand colors

### Typography
- **Primary Font**: "Press Start 2P" (retro pixel font from Google Fonts)
- **Base Font Size**: 10px (very small, pixelated aesthetic)
- **Footer Font Size**: 8px
- **Line Height**: 1.5

### Visual Style
- **Pixel-art aesthetic**: Uses `image-rendering: pixelated` for images
- **Border style**: 4px solid borders with background colors
- **Spacing**: Minimal margins (5px) for compact layout
- **Box shadows**: Subtle `0px 0px 10px rgba(0, 0, 0, 0.1)`
- **Border radius**: 10px for rounded corners on images

## HTML Structure Conventions

### Page Sections (index.html:13-47)
The page is divided into semantic containers:

1. **Header** (`<header>`) - Site title/branding
2. **About Container** (`.about-container`) - Profile image and personal info
3. **Tech Stack** (`.section` with `.section-title`) - Pixelated pills derived from post hashtags, styled as a terminal package list
4. **Work Log** (`.section` with `#work-log`) - Terminal-style log entries with pagination (5 per page)
5. **Links** (`.section` with `.section-title`) - Professional and social links
6. **Footer** (`<footer>`) - Build credits

### Log Entries

Posts use `.log-entry` containers with:
- `.log-line` - One terminal line containing:
  - `.log-prompt` - `>>` prefix in coral accent color
  - `.log-timestamp` - Date stamp in primary accent color
  - `.log-message` - Activity description in secondary accent color
- `.log-tags` - Hashtags in primary accent color

The Work Log uses vanilla JavaScript for pagination: 5 entries per page, with `prev`/`next` buttons.

New posts are added by duplicating an existing `<article class="log-entry">` block inside `#log-entries`. When a new technology hashtag is introduced, add the corresponding `.stack-pill` to the Tech Stack cloud. Remember to keep the total entry count aligned with the pagination logic (`pageSize = 5` in the inline script).

### Layout Patterns
- **Flexbox for about section**: `display: flex` with `align-items: center` and `gap: 10px`
- **Max-width containers**: 500px for readability
- **Responsive images**: `max-width: 80%` on images
- **Text alignment**: Left-aligned in about section, center-aligned elsewhere

## Development Workflows

### Git Workflow

**Branch Naming Convention**:
- Feature branches: `claude/descriptive-name-xxxxx`
- Branch names must start with `claude/` for AI assistant work
- Must include session ID suffix for push authentication

**Commit Message Style** (based on recent commits):
- Use descriptive, imperative mood messages
- Examples:
  - "Color refactor"
  - "Some links deletion"
  - "Change theme colors"
  - "Update index.html" (for minor changes)

**Pull Request Pattern**:
- Create feature branches for all changes
- Merge via pull requests (even for small changes)
- Clean commit history with merge commits
- Recent PRs: #8 (links-deletion), #7 (color-refactor), #6 (change-colors)

### Testing Checklist

Before committing changes to this **static site**, verify:

1. **HTML Validation**: Ensure valid HTML5 syntax
2. **CSS Validation**: Check for syntax errors in styles.css
3. **Visual Testing**:
   - Check layout at different viewport sizes
   - Verify color consistency across all sections
   - Test all links (external APIs, social links)
4. **Cross-browser Testing**: Test on major browsers
5. **External Resources**:
   - Verify Google Fonts loading
   - Check GitHub Stats API widgets are rendering
6. **Mobile Responsiveness**: Ensure proper display on mobile devices

### Deployment

**Platform**: GitHub Pages
**Branch**: Typically deploys from `main` branch
**Process**: Automatic deployment on merge to main
**URL Pattern**: `https://duv0-x.github.io/`

## File Management Conventions

### Images
- **Location**: `/images/` directory only
- **Naming**: Use lowercase with hyphens (e.g., `x-0vub-small.jpg`)
- **Optimization**: Keep file sizes reasonable for web
- **Variants**: Maintain both full-size and small versions

### Files (PDFs/Documents)
- **Location**: `/files/` directory only
- **Naming**: Descriptive with version indicators
  - Pattern: `CV_Name_Specialty_Language_Version.pdf`
  - Example: `CV_Duvan_Ballen_SRE_DevOps_EN_v2.pdf`
- **Versions**: Keep multiple versions for different purposes

## Key Conventions for AI Assistants

### When Making Changes

1. **Read Before Edit**: ALWAYS read `index.html` and `styles.css` before making changes
2. **Preserve Aesthetic**: Maintain the retro/pixel-art theme - this is intentional design
3. **Color Consistency**: Never introduce new colors without discussion - stick to the established palette
4. **Minimal Changes**: This is a personal portfolio - avoid over-engineering
5. **Test Links**: Verify all external links work (GitHub stats, social media)

### Common Modification Scenarios

**Updating Personal Information** (index.html:22-30):
- Edit the structured text within `.about-container`
- Maintain the format: `Label:<br> Value<br><br>`
- Keep the personal, casual tone

**Adding/Removing Social Links** (index.html:38-43):
- Categorized into "Pro links" and "Chill links"
- Include emoji prefix for visual interest
- Use appropriate accent colors for different platforms
- Recent change: Links have been pruned, so be conservative about adding new ones

**Updating Styling**:
- All styles in `styles.css` - no inline styles except link colors in HTML
- Comment your CSS changes clearly
- Test changes across all page sections

**Updating CVs/Resumes**:
- Add new versions to `/files/` directory
- Follow existing naming convention
- Keep old versions unless explicitly asked to remove

### What NOT to Do

✅ **JavaScript is allowed only for small, self-contained interactions** like pagination; keep it inline and minimal
❌ **Don't** create new CSS files - all styles go in `styles.css`
❌ **Don't** add build tools, preprocessors, or frameworks
❌ **Don't** change the font family without explicit permission
❌ **Don't** modify the pixel-art aesthetic (e.g., removing `image-rendering: pixelated`)
❌ **Don't** add analytics, tracking, or third-party scripts without permission
❌ **Don't** create documentation files (like additional READMEs) unless requested
❌ **Don't** re-add the GitHub starred repositories section without explicit permission

## External Dependencies

### Google Fonts
```html
<link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
```
- Using "Press Start 2P" font
- Loaded via Google Fonts CDN
- Preconnect to fonts.googleapis.com and fonts.gstatic.com for performance

### GitHub Top Languages Stats

**Current Service**: Using `stats.programcx.cn` (alternative instance)

```html
https://stats.programcx.cn/api/top-langs/?username=duv0-x&layout=compact&theme=gotham&hide_border=true&langs_count=8
```

**Why the alternative instance?**
- The original `github-readme-stats.vercel.app` experienced downtime and rate limiting issues (Jan 2026)
- Switched to `stats.programcx.cn` - a community-hosted instance with better uptime
- Uses same API as the original github-readme-stats project

**Styling**:
- Uses `gotham` theme to match site colors
- Hides borders for cleaner integration
- Shows top 8 languages in compact layout

**Note**: Only the Top Languages widget is used. Other stats widgets (GitHub Stats, Streak Stats) were removed due to API reliability issues and replaced with static Tech Stack badges.

### Tech Stack Badges

**Service**: Using `shields.io` (static badge generator)

Located in index.html lines 37-51. Displays key technologies and tools using shields.io badges.

**Current Technologies**:
- **Containers**: Docker, Kubernetes
- **Cloud**: AWS
- **IaC**: Terraform
- **Languages**: Python, Go
- **CI/CD**: GitHub Actions, CircleCI
- **OS**: Linux

**Badge Format**:
```html
<img src="https://img.shields.io/badge/TechName-ColorHex?style=flat&logo=logoname&logoColor=white" alt="TechName"/>
```

**Updating Tech Stack**:
- Add/remove badges as needed to reflect current skills
- Use official brand colors for consistency
- Keep logos from simple-icons.org (used by shields.io)
- Maintain `style=flat` for clean, minimal look
- All badges should have descriptive alt text for accessibility

**Finding Badge URLs**:
1. Visit https://shields.io
2. Use format: `/badge/<LABEL>-<COLOR>?style=flat&logo=<LOGO>&logoColor=white`
3. Find logo names at https://simpleicons.org
4. Use official brand colors when possible

## HTML Best Practices for This Site

1. **Language**: Set to Spanish (`lang="es"`) but content is multilingual
2. **Meta Tags**: Include viewport meta for mobile responsiveness
3. **Semantic HTML**: Use proper semantic tags (`<header>`, `<footer>`, `<nav>` if needed)
4. **Accessibility**: Include `alt` attributes on images
5. **External Links**: Use `rel="me"` for social verification where appropriate (see Mastodon link)

## CSS Best Practices for This Site

1. **No CSS Variables**: Currently using direct hex values - maintain this pattern
2. **Class Naming**: Use kebab-case (e.g., `.about-container`, `.social-networks`)
3. **Specificity**: Keep selectors simple, avoid deep nesting
4. **Comments**: Use CSS comments for sections or non-obvious styling
5. **Duplicate Rules**: Some duplicate rules exist (lines 44-56 in styles.css) - clean up if adding nearby

## Recent Changes & Context

### Latest Refactoring (PRs #6-#8)
- **Color Refactor (#7)**: Standardized color scheme across the site
- **Links Deletion (#8)**: Removed some social links - keep link section minimal
- **Change Colors (#6)**: Updated theme colors to current palette

This indicates:
- The owner is actively maintaining and refining the site
- Prefer simplicity and minimalism
- Color scheme is recently finalized - respect current choices

## Troubleshooting Common Issues

### Images Not Displaying
- Check file path is relative: `images/filename.jpg`
- Verify file exists in `/images/` directory
- Check filename case matches exactly (Linux is case-sensitive)

### Styles Not Applying
- Ensure `<link rel="stylesheet" href="styles.css">` is in `<head>`
- Check CSS syntax (missing semicolons, brackets)
- Verify selector names match HTML classes/elements

### GitHub Stats Not Loading
- These are external API calls - test the URL directly
- Check username parameter is correct: `username=duv0-x`
- Verify API parameters are URL-encoded properly

## Quick Reference

### Important File Locations
- Main page: `/index.html`
- Styles: `/styles.css`
- Profile images: `/images/`
- Resume/CV: `/files/`

### Current Active Links
- LinkedIn: https://www.linkedin.com/in/duvanballen
- GitHub: https://github.com/duv0-x
- Mastodon: https://mastodon.social/@duv0_x

### Brand Colors (Hex Codes)
- `#000` - Body background
- `#0c1015` - Container backgrounds
- `#27a888` - Headers and primary accent
- `#98d1cc` - Text and secondary accent
- `#e06c76` - LinkedIn link color

## Working with This Repository

### Typical AI Assistant Tasks

1. **Content Updates**: Modify personal info, status, or descriptions in index.html
2. **Link Management**: Add/remove social or professional links
3. **Style Tweaks**: Adjust spacing, colors, or layout in styles.css
4. **CV Updates**: Replace or add new resume versions in /files/
5. **Image Updates**: Swap or optimize profile images

### Example Workflows

**Task: Update employment status**
1. Read index.html (line 28-29)
2. Edit the "Status:" section with new information
3. Maintain the casual, personal tone
4. Commit: "Update employment status"

**Task: Add a new timeline post**
1. Read index.html timeline section
2. Duplicate an existing `<article class="sms-post">` block at the top (most recent first)
3. Update date, message content, and hashtags
4. If the post introduces a new technology hashtag, add a corresponding `<span class="stack-pill">` to `.tech-stack`
5. Commit: "Add work log entry for [date]"

**Task: Add a new social link**
1. Read index.html (links section)
2. Add with emoji prefix and appropriate color
3. Commit: "Add [platform] link"

**Task: Adjust spacing/padding**
1. Read styles.css
2. Identify the specific selector to modify
3. Make minimal changes to achieve desired effect
4. Test visual appearance
5. Commit: "Adjust [element] spacing"

## Summary

This is a **simple, intentional, and personal** portfolio site. The retro pixel aesthetic and dark color scheme are core to its identity. When working on this repository:

- **Respect the minimalist approach** - don't add complexity
- **Maintain the visual identity** - colors, fonts, and style are carefully chosen
- **Test thoroughly** - it's a public-facing portfolio
- **Commit clearly** - follow the established git workflow
- **Ask when in doubt** - especially for design decisions

This site represents the owner's professional brand - treat changes with care and consideration.

---

**Last Updated**: 2026-01-04
**For Questions**: Consult the repository owner or check recent PRs for context

---
> Source: [duv0-x/duv0-x.github.io](https://github.com/duv0-x/duv0-x.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
