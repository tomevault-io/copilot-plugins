## tdstudioshq

> This project is a web application for TD Studios, built with the Lovable platform. It serves as a central hub for various TD Studios brands and products, with different user roles and dashboards. The application is built using a modern frontend stack, including React, Vite, and TypeScript. It's styled with Tailwind CSS and shadcn-ui, and it uses Playwright for end-to-end testing.

# TD Studios HQ

## Project Overview

This project is a web application for TD Studios, built with the Lovable platform. It serves as a central hub for various TD Studios brands and products, with different user roles and dashboards. The application is built using a modern frontend stack, including React, Vite, and TypeScript. It's styled with Tailwind CSS and shadcn-ui, and it uses Playwright for end-to-end testing.

### Key Technologies

*   **Framework:** React
*   **Build Tool:** Vite
*   **Language:** TypeScript
*   **Styling:** Tailwind CSS, shadcn-ui
*   **Routing:** React Router
*   **State Management:** React Query
*   **Testing:** Playwright
*   **Deployment:** Vercel

## Building and Running

### Development

To run the application in development mode, use the following commands:

```bash
# Install dependencies
npm i

# Generate sitemap
node scripts/generate-sitemap.mjs

# Start the development server
npm run dev
```

The application will be available at `http://localhost:5173`.

### Building for Production

To build the application for production, use the following command:

```bash
npm run build
```

The production-ready files will be generated in the `dist` directory.

### Testing

To run the end-to-end tests, use the following command:

```bash
npm run test:e2e
```

## Development Conventions

### Code Style

The project uses ESLint to enforce a consistent code style. You can run the linter with the following command:

```bash
npm run lint
```

### Type Checking

The project uses TypeScript for static type checking. You can run the type checker with the following command:

```bash
npm run typecheck
```

### Git Hooks

The project uses `husky` and `lint-staged` to run ESLint on staged files before committing.

### Deployment

The project is automatically deployed to Vercel when changes are pushed to the `main` branch.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tdiorio2323)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tdiorio2323)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
