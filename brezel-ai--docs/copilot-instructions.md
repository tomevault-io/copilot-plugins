## docs

> > **Project setup note**: This file is now seeded for BrezelScraper. Ask the user to confirm any preferred product terminology before large information architecture changes or broad rewrites.

> **Project setup note**: This file is now seeded for BrezelScraper. Ask the user to confirm any preferred product terminology before large information architecture changes or broad rewrites.
> For Mintlify product knowledge (components, configuration, writing standards),
> install the Mintlify skill: `npx skills add https://mintlify.com/docs`

# Documentation project instructions

## About this project

- This is the customer-facing documentation site for BrezelScraper
- The product is a SaaS workflow for launching Google Maps scraping jobs, reviewing results, exporting data, and managing credits
- Pages are MDX files with YAML frontmatter
- Configuration lives in `docs.json`
- Run `mint dev` to preview locally
- Run `mint broken-links` to check links
- The source of truth for behavior is the application code:
  - Backend: `/Users/yasseen/Documents/brezel.ai/BrezelScraper/brezelscraper-backend`
  - Frontend: `/Users/yasseen/Documents/brezel.ai/BrezelScraper/brezelscraper-frontend`

## Terminology

- Use **job** as the primary term for a scraping run
- Use **results** for extracted business data
- Use **credits** for usage and billing balance
- Use **dashboard** for the authenticated product area
- Use **integrations** for API keys and webhooks
- Prefer the product name **BrezelScraper** over generic phrases like "the scraper"

## Style preferences

- Use active voice and second person ("you")
- Keep sentences concise
- Use sentence case for headings
- Bold UI elements: Click **Jobs**
- Use code formatting for file names, commands, paths, endpoints, headers, and code references
- Start with the dashboard workflow before the API workflow when both are available
- Explain how a setting affects cost, runtime, or result volume when that tradeoff matters
- Prefer practical examples over abstract feature descriptions

## Content boundaries

- Focus on user-facing SaaS features that exist in the dashboard or public API
- Prioritize job creation, results, CSV downloads, credits, integrations, and support
- Do not document internal admin routes or internal-only operations
- Do not document deployment, infrastructure, or private operational runbooks in customer docs
- Do not trust the existing OpenAPI examples blindly; verify handlers and frontend usage first
- Treat billing, authentication, and API behavior as implementation-sensitive and verify them against code before publishing

---
> Source: [brezel-ai/docs](https://github.com/brezel-ai/docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
