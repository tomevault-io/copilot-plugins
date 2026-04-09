## rangeway-research

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Rangeway Research - AI-powered research engine for EV charging and hospitality insights. Jekyll-based frontend hosted on GitHub Pages at research.rangewayev.com, with n8n webhook backend connecting to Perplexity AI.

## Build & Development Commands

```bash
# Install dependencies
bundle install

# Local development server
bundle exec jekyll serve

# Build for production
bundle exec jekyll build
```

Local server runs at http://localhost:4000/rangeway-research

## Architecture

```
User → Jekyll Site (GitHub Pages) → n8n Webhook → Perplexity AI
```

### Key Files

```
/
├── _config.yml              # Jekyll config
├── _layouts/default.html    # Base template
├── assets/
│   ├── css/style.css        # Styles
│   ├── js/app.js            # Frontend logic, webhook integration
│   └── images/              # Brand imagery
├── index.html               # Research interface
└── CNAME                    # Custom domain
```

## Frontend

- `index.html` - Research tool interface
- `assets/js/app.js` - Handles user queries, webhook calls, response display
- `assets/css/style.css` - Styling

## Backend Integration

The site sends research queries to an n8n webhook, which processes them through Perplexity AI and returns results. The webhook URL is configured in `assets/js/app.js`.

## Deployment

Automatic via GitHub Pages on push to `main` branch.

## Dependencies

- Jekyll (via github-pages gem)
- jekyll-seo-tag plugin

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Redwood-Mobility)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Redwood-Mobility)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
