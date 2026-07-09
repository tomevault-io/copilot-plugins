## zevlo-github-io

> This is **Zachary's Blog** - a personal Jekyll-powered GitHub Pages site at [zevlo.github.io](https://zevlo.github.io). The site documents projects, blog posts, and thoughts on Artificial Intelligence, Complex Systems, and DevOps.

# CLAUDE.md

## Project Overview

This is **Zachary's Blog** - a personal Jekyll-powered GitHub Pages site at [zevlo.github.io](https://zevlo.github.io). The site documents projects, blog posts, and thoughts on Artificial Intelligence, Complex Systems, and DevOps.

## Tech Stack

- **Static Site Generator**: Jekyll with Kramdown markdown parser
- **Hosting**: GitHub Pages (auto-builds on push)
- **Syntax Highlighting**: Rouge
- **Styling**: Custom CSS (`style.css`)

## File Structure

```
zevlo.github.io/
├── _config.yml           # Jekyll configuration
├── _layouts/
│   ├── default.html      # Main layout (header, nav, footer)
│   └── post.html         # Layout for individual content pages
├── _posts/               # Blog posts (YYYY-MM-DD-title.md)
├── _projects/            # Project showcases (project-name.md)
├── _thoughts/            # Short reflections (YYYY-MM-DD-title.md)
├── assets/images/        # Image assets
├── index.html            # Home page
├── posts.html            # Blog listing page
├── projects.html         # Projects listing page
├── thoughts.html         # Thoughts listing page
├── style.css             # Site-wide styles
├── CNAME                 # Custom domain config
└── HOWTO.md              # Content writing guide
```

## Content Collections

### Posts (`_posts/`)
Full blog articles. Filename format: `YYYY-MM-DD-title-slug.md`

Required frontmatter:
```yaml
---
layout: post
title: "Post Title"
date: YYYY-MM-DD
---
```

### Projects (`_projects/`)
Project showcases. Filename format: `project-name.md`

Required/optional frontmatter:
```yaml
---
layout: post
title: "Project Name"
date: YYYY-MM-DD
description: "Short description"  # Optional, shown in listings
github_url: "https://..."         # Optional
demo_url: "https://..."           # Optional
---
```

### Thoughts (`_thoughts/`)
Short reflections and observations. Filename format: `YYYY-MM-DD-title.md`

Required frontmatter:
```yaml
---
date: YYYY-MM-DD
---
```

## Development Commands

```bash
# Local development (requires Jekyll installed)
bundle exec jekyll serve

# Build site locally
bundle exec jekyll build
```

## Deployment

- Push to main branch triggers automatic GitHub Pages build
- Build takes ~1-2 minutes
- Check GitHub Actions tab for build status

## Important Conventions

- **Dates matter**: Posts/thoughts require `YYYY-MM-DD-` filename prefix
- **Future dates**: Content with future dates won't display
- **Markdown**: Write content in Markdown (HTML also supported)
- **Frontmatter**: Always enclosed in `---` delimiters
- **Images**: Store in `assets/images/`, reference as `/assets/images/filename.png`

## URLs/Permalinks

- Posts: `/posts/:title/`
- Projects: `/projects/:title/`
- Thoughts: `/thoughts/:title/`

---
> Source: [zevlo/zevlo.github.io](https://github.com/zevlo/zevlo.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
