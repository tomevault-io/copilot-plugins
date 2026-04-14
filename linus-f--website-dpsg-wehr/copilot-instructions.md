## website-dpsg-wehr

> This is the source code for the "DPSG Wehr" website, built with **Next.js 16** (App Router). The site is designed to be **completely statically generated** (`output: "export"`) and deployed to a static host.

# Website DPSG Wehr - Project Context

## Project Overview

This is the source code for the "DPSG Wehr" website, built with **Next.js 16** (App Router). The site is designed to be **completely statically generated** (`output: "export"`) and deployed to a static host.

**Key Features:**

- **Static Site Generation (SSG):** Optimized for performance and simple hosting.
- **Dynamic Routing:** Content is decoupled from logic. MDX files are stored in a central `content/` directory and rendered via dynamic route templates.
- **Automated SEO:** Social previews (Open Graph and Twitter Cards) are automatically generated for every page and post using content excerpts.
- **Automated Testing:** Comprehensive suite including Unit, Component, and E2E tests with unified reporting.
- **Image Optimization:** Uses `next-image-export-optimizer` for build-time optimization compatible with static export.
- **Hybrid CMS Strategy:** Uses Tina Cloud for content management but a custom Git-based store for media to support local optimization.

## Tech Stack

- **Framework:** [Next.js](https://nextjs.org) (App Router)
- **CMS:** [TinaCMS](https://tina.io) (Hybrid Mode)
- **Language:** [TypeScript](https://www.typescriptlang.org)
- **Styling:** [Tailwind CSS](https://tailwindcss.com), [React Icons](https://react-icons.github.io/react-icons/)
- **Content:** MDX rendered via `next-mdx-remote`, `gray-matter` for frontmatter.
- **Testing:** [Vitest](https://vitest.dev/) (Unit/Component), [Playwright](https://playwright.dev/) (E2E), [Monocart Reporter](https://github.com/cenfun/monocart-reporter).
- **Package Manager:** [pnpm](https://pnpm.io)

## Architecture & CMS Configuration

This project uses a unique hybrid architecture to combine the convenience of Tina Cloud with the performance of local static image optimization.

### 1. The "Hybrid" Tina Configuration

- **Content:** Managed via Tina Cloud (Production) or Local GraphQL Server (Dev/CI).
- **Media:** Managed via a custom `GitMediaStore` (`tina/git-media-store.ts`).
    - **Why?** Tina Cloud's default media store hosts images on their CDN (`assets.tina.io`). This bypasses our `next-image-export-optimizer` pipeline, which requires local files.
    - **How?** The `GitMediaStore` uploads images directly to the GitHub repository (via API) into `public/media`.
    - **Result:** MDX files save local paths (e.g., `/media/images/photo.jpg`), allowing the build process to optimize them.

### 2. Environment Modes (`tina/config.ts`)

- **Local / CI (`TINA_PUBLIC_IS_LOCAL=true`):**
    - `clientId` / `token`: `undefined` (Disabled).
    - `contentApiUrlOverride`: `http://localhost:4001/graphql` (Absolute URL required for Node.js build steps).
    - Used for: `pnpm dev` and `pnpm build:ci`.
- **Production (`TINA_PUBLIC_IS_LOCAL=false`):**
    - `clientId` / `token`: Set from env vars.
    - `contentApiUrlOverride`: `undefined` (Uses Tina Cloud).
    - Used for: The deployed Admin SPA (`/admin`).

### 3. Security (VPS Deployment)

- **Problem:** The `GitMediaStore` requires a GitHub PAT with `repo` scope to be present in the client-side bundle.
- **Solution:** The `/admin` route is protected by **Nginx Basic Auth**.
    - Credentials are managed via `.htpasswd` (mounted into Docker container).
    - This ensures only authorized admins can load the JS bundle containing the PAT.

## Directory Structure

- `app/`: Next.js App Router directory.
- `content/`: Raw content managed as MDX files.
- `tina/`: CMS Configuration.
    - `config.ts`: Main config with environmental logic.
    - `git-media-store.ts`: Custom media store implementation.
- `docs/`: Project documentation (Architecture, etc.).
- `scripts/`: Utility scripts (`setup-vps.sh`, `generate-ics.ts`).
- `docker-compose.yml`: VPS deployment config.
- `nginx.conf`: Nginx config with Basic Auth and Security Headers.

## Development & Testing

### Key Commands

- **Start Development Server:** `pnpm dev` (Runs at `http://localhost:3000`).
- **Run Unit Tests:** `pnpm test` (Vitest watch mode).
- **Run E2E Tests:** `pnpm test:e2e` (Playwright).
- **Run All Tests:** `pnpm test:all` (Sequential Vitest + Playwright).
- **Build (CI):** `pnpm build:ci` (Local mode, skips cloud checks).
- **Setup VPS:** `./scripts/setup-vps.sh` (Generates env/auth files).

## Release Workflow

The project uses **Semantic Release** to automate versioning and changelog generation.

1.  **Release Workflow (`release.yml`):** Triggered on push to `main`.
2.  **Docker Build Workflow (`docker.yml`):** Triggered on new tags.
3.  **Tests Workflow (`tests.yml`):** Triggered on push/PR.
    - Note: Builds in "Local Mode" to verify the codebase without needing Cloud credentials.

## Agent Workflows

### Automated SEO Check

When modifying metadata logic, verify that `<meta>` tags for `og:title`, `og:description`, and `og:image` are correctly rendered using the `e2e/social.spec.ts` test.

### ICS Generation

ICS files are generated as part of the build process from `lib/events.public.ts` and `lib/events.internal.ts`. The logic is isolated in `scripts/generate-ics.ts`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Linus-f)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Linus-f)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
