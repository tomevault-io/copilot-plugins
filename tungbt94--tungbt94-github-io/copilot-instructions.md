## tungbt94-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal blog built with **VuePress 1.x** - a Vue-powered static site generator. The site is written primarily in Vietnamese.

## Common Commands

```bash
# Install dependencies
pnpm install

# Run development server
pnpm vuepress dev

# Build for production
pnpm vuepress build

# Deploy to GitHub Pages
./deploy.sh

# Deploy to Surge.sh
./deploy-surge.sh
```

## Project Structure

```
.vuepress/
├── config.js       # Site configuration (nav, sidebar, plugins)
├── components/     # Vue components usable in markdown files
├── public/         # Static assets (avatar, manifest)
└── dist/           # Build output (gitignored)

post/               # Blog posts (markdown)
til/                # "Today I Learned" short notes
about/              # About pages
coaching/           # Coaching-related content with Vue components
assets/img/         # Images used in posts
```

## Architecture Notes

- **Custom Vue Components**: Components in `.vuepress/components/` can be used directly in markdown files without imports. Components include skill assessment tools (`BaSkill.vue`, `Disc.vue`, `Enneagram.vue`, `Mbti.vue`).

- **Navigation & Sidebar**: Configured in `.vuepress/config.js`. When adding new posts, update the `sidebar` array to include them.

- **PWA Support**: Enabled via `@vuepress/plugin-pwa` with service worker and update popup.

- **Deployment**: The `deploy.sh` script builds the site and force-pushes to the master branch of the GitHub Pages repo.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tungbt94)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tungbt94)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
