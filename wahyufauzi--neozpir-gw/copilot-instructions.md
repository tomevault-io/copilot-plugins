## neozpir-gw

> This is a web application built with [SolidJS](https://www.solidjs.com/) for the frontend and [Cloudflare Workers](https://workers.cloudflare.com/) for the backend. The project uses [TypeScript](https://www.typescriptlang.org/), [Vite](https://vitejs.dev/) for building, [Tailwind CSS](https://tailwindcss.com/) for styling, and [Hono](https://hono.dev/) for backend routing.

# Project Overview

This is a web application built with [SolidJS](https://www.solidjs.com/) for the frontend and [Cloudflare Workers](https://workers.cloudflare.com/) for the backend. The project uses [TypeScript](https://www.typescriptlang.org/), [Vite](https://vitejs.dev/) for building, [Tailwind CSS](https://tailwindcss.com/) for styling, and [Hono](https://hono.dev/) for backend routing.

The application appears to be a company website, featuring pages for products/services and a blog. The architecture is based on a static frontend served by Cloudflare Pages, with a serverless API backend providing data for products and blog posts.

## Building and Running

The following commands are available in `package.json` to manage the application:

- **`npm run dev:frontend`**: Starts the frontend development server using Vite.
- **`npm run dev:backend`**: Starts the backend development server using Wrangler.
- **`npm run build`**: Builds the application for production.
- **`npm run deploy:worker`**: Deploys the application to Cloudflare.

## Development Conventions

### Routing

- **Frontend**: Client-side routing is handled by `@solidjs/router`. All routes are defined in `src/App.tsx`.
- **Backend**: API routing is managed by `hono`. The API endpoints are defined in `worker/index.ts`.

### Internationalization (i18n)

The project includes a custom internationalization setup using `I18nContext`. Translation files are located in `src/i18n/locales` for English (`en.json`) and Indonesian (`id.json`).

### Styling

Styling is primarily done with Tailwind CSS. There are also some component-specific CSS modules (e.g., `src/App.module.css`).

### Data Fetching

The frontend fetches data from the backend's API endpoints. The backend serves product information from JSON files located in the `worker/ecommerce` directory and blog content from a Cloudflare D1 database.

### API

The backend exposes a RESTful API under the `/api` path. The available endpoints include:

- `/api/product/:product?langKey=:langKey`: Fetches product information.
- `/api/blog/:langkey`: Fetches a list of blog posts for a given language.
- `/api/blog/:langkey/:title`: Fetches a specific blog post.

---
> Source: [WahyuFauzi/neozpir-gw](https://github.com/WahyuFauzi/neozpir-gw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
