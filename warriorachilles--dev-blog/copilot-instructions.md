## dev-blog

> A Next.js-based development blog and portfolio site for Zion Emond, derived from the **tailwind-nextjs-starter-blog** template and highly customized ([GitHub repo](https://github.com/WarriorAchilles/dev-blog)).

# dev-blog (Zion Emond Portfolio)

A Next.js-based development blog and portfolio site for Zion Emond, derived from the **tailwind-nextjs-starter-blog** template and highly customized ([GitHub repo](https://github.com/WarriorAchilles/dev-blog)).

---

## Project structure & organization

- **Directories & key files**:
    - `.devcontainer/`, `.github/`, `.husky/`, `.vscode/`, `.yarn/releases/`: Dev environment and workflow tooling.
    - `app/`, `components/`, `layouts/`, `faq/`, `data/`, `scripts/`: Main application logic and content organization.
    - `css/tailwind.css`: Tailwind CSS entry point—used instead of `tailwind.config.js`.
    - `public/static/`: Static assets for deployment.
    - Root configuration: `.env.example`, `.gitignore`, `LICENSE`, `README.md`, `contentlayer.config.ts`, `eslint.config.mjs`, `jsconfig.json`, `next-env.d.ts`, `next.config.js`, `package.json`, `package-lock.json`, `postcss.config.js`, `prettier.config.js`, `tsconfig.json`.

---

## Tooling & dependencies

- **Framework**: Next.js v15.2.4
- **Styling**: Tailwind CSS v4.0.5 via `css/tailwind.css`, processed through PostCSS
    - Tailwind-related packages:
        - `@tailwindcss/forms` v0.5.9
        - `@tailwindcss/postcss` v4.0.5
        - `@tailwindcss/typography` v0.5.15
- **Linting**: ESLint v9.14.0 with TypeScript support and Prettier integration
- **Formatting**: Prettier v3.0.0 with `prettier-plugin-tailwindcss`
- **TypeScript**: v5.1.3
- **Content Layer**: `contentlayer2` v0.5.5 with `next-contentlayer2`
- **3D & animations**: `three`, `@react-three/fiber`, `@react-three/drei`, `ogl`, `gsap`
- **Markdown/MDX processing**: `remark`, `rehype` ecosystem (KaTeX, math, syntax highlighting, etc.)
- **Other notable libraries**:
    - `openai` v5.0.1 for AI integration
    - `pliny` v0.4.1 for RSS, sitemap, metadata
    - `body-scroll-lock`, `gray-matter`, `reading-time`, `github-slugger`, and others for UI and content handling

---

## Scripts

From `package.json`:

```bash
npm run dev        # Launches the development server
npm run start      # Alias for next dev
npm run build      # Builds for production and runs postbuild script
npm run serve      # Starts production server
npm run analyze    # Builds with bundle analyzer enabled
npm run lint       # Runs ESLint (with auto-fix) across key directories
npm run prepare    # Installs Husky git hooks
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WarriorAchilles) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
