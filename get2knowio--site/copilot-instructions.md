## site

> This project is a personal blog built with Next.js 14, designed as a monorepo. It serves as a platform for blog posts, project showcases, and merchandise. The architecture emphasizes a clear separation between the application shell (`apps/site-shell`) and the content (`content/`).

# Paul's Blog - Project Context

## Project Overview
This project is a personal blog built with Next.js 14, designed as a monorepo. It serves as a platform for blog posts, project showcases, and merchandise. The architecture emphasizes a clear separation between the application shell (`apps/site-shell`) and the content (`content/`).

### Key Technologies
- **Framework:** Next.js 14 (App Router)
- **Language:** TypeScript
- **Styling:** Tailwind CSS, Shadcn UI (@radix-ui primitives)
- **Testing:** Jest (Unit), Playwright (E2E)
- **Content:** MDX (Markdown + React components)

## Building and Running

### Prerequisites
- Node.js v18+
- npm

### Development Commands
Run these commands from the root directory:

- **Start Development Server:**
  ```bash
  npm run dev
  ```
  Access at `http://localhost:3000` (or `http://0.0.0.0:3000`).

- **Build for Production:**
  ```bash
  npm run build
  ```
  Includes SEO and Merchandise validation steps.

- **Start Production Server:**
  ```bash
  npm start
  ```

### Testing & Quality
- **Unit Tests (Jest):**
  ```bash
  npm test
  # Watch mode
  npm run test:watch -w apps/site-shell
  ```
- **E2E Tests (Playwright):**
  ```bash
  npm run test:e2e -w apps/site-shell
  ```
- **Linting & Formatting:**
  ```bash
  npm run lint
  npm run format
  npm run typecheck -w apps/site-shell
  ```

## Development Conventions

### Directory Structure
- **`apps/site-shell/`**: The main Next.js application code.
  - `app/`: Next.js App Router pages and layouts.
  - `components/`: React components (UI library).
  - `scripts/`: Utility scripts (e.g., validation).
- **`content/`**: Data source for the blog.
  - `blog/`: Blog posts organized by folder (see Content Management).
  - `merch/`: Product data.
  - `projects.json`: Portfolio data.
- **`specs/`**: Architecture decision records and specifications.

### Content Management
Blog posts require a specific structure for deterministic URLs:
- **Path:** `content/blog/{slug}/index.md`
- **Slug Generation:** Folder names become the URL slug.
- **Verification:**
  - `npm run verify-slugs`: Ensures consistency.
  - `npm run slug:update-manifest`: Updates the manifest when adding new posts.
  - `npm run slug:preview`: Check a slug before creation.

### Coding Style
- **TypeScript:** Strict mode is enabled (`noImplicitAny`, `strictNullChecks`).
- **Formatting:** Prettier is enforced (`npm run format`).
- **Linting:** ESLint with Next.js configuration (`npm run lint`).
- **Imports:** Use absolute imports `@/` for `apps/site-shell` root.

## Important Notes
- This is an **npm** project. Do not use `yarn`.
- If you encounter lockfile issues, remove `yarn.lock` if it appears.
- The `specs` folder contains historical context and plans for features (e.g., `004-preserve-slugs`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/get2knowio)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/get2knowio)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
