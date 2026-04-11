## fingrab-site

> Landing page for FinGrab.app, a Chrome extension designed to export financial data from Yahoo Finance.

# FinGrab Site Context

Landing page for FinGrab.app, a Chrome extension designed to export financial data from Yahoo Finance.

## Tech Stack
- **Framework:** Vue 3 (Composition API with `<script setup>`)
- **Build Tool:** Vite
- **Styling:** Tailwind CSS v4
- **Routing:** Vue Router (Hash History)
- **SEO/Head Management:** Unhead (@unhead/vue)
- **Deployment:** GitHub Pages (via GitHub Actions)

## Project Structure
- `src/main.js`: Entry point, defines routes and static content passed as props to components.
- `src/App.vue`: Root component, renders `router-view`.
- `src/components/Home.vue`: Main landing page component, uses props for dynamic content.
- `src/style.css`: Global styles including Tailwind directives.
- `.github/workflows/deploy.yml`: CI/CD pipeline for deploying to GitHub Pages.

## Core Content
Content is managed in `src/main.js` within the router configuration. This includes:
- Product name: FinGrab.app
- Headline and UVP (Unique Value Proposition)
- Call to Action (CTA) details and link to the Chrome Web Store.

## Development & Build
- `npm run dev`: Start development server.
- `npm run build`: Build for production (output to `dist/`).
- `npm run preview`: Preview the production build locally.

## Deployment
Automated via GitHub Actions on push to `main` branch.
- Deployment target: GitHub Pages.
- Domain: `fingrab.app` (configured via `CNAME` in build step).
- Branch: `site` (pushed to `site` branch by the action).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jenslaufer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jenslaufer)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
