## docs-oraxen-com

> - **Framework**: Next.js 16 (App Router) + Nextra 4 (Docs theme) + React 19 + TypeScript 5.


## Documentation Site Architecture

- **Framework**: Next.js 16 (App Router) + Nextra 4 (Docs theme) + React 19 + TypeScript 5.
- **Runtime/PM**: Bun (see `bun.lock`). Prefer `bun run` for scripts.

### Routing & Rendering

- `app/[[...mdxPath]]/page.tsx`
  - Uses `nextra/pages` (`importPage`, `generateStaticParamsFor`) to map MDX under `content/` to routes.
  - Exports `generateMetadata` to surface page metadata from MDX frontmatter.
- `app/layout.tsx`
  - Wraps pages with `nextra-theme-docs` `Layout` and `Navbar`.
  - Navbar logo uses `/public/logo_lowres.png` and must render with `imageRendering: "pixelated"` to preserve pixel art.

### Content Organization

- `content/` holds MDX pages and `_meta.js` files to drive sidebar order/titles.
- Theme examples live under `content/themes/**`.
- Images live in `public/` and can be referenced via Markdown or `next/image`.

### MDX Components

- `mdx-components.ts` merges `nextra-theme-docs` components with local overrides.
- `next-mdx-import-source-file.ts` re-exports MDX components for aliasing.

### Configuration

- `next.config.mjs`
  - Enables Nextra with `latex: true` and built-in search (`codeblocks: false`).
  - Provides a Turbopack alias for `next-mdx-import-source-file`.
- `tsconfig.json`
  - Strict TypeScript, modern ESM (`moduleResolution: Bundler`).

### Build & Commands

- Scripts in `package.json`: `dev`, `build`, `start` (use `bun run <script>`).
- Static params and metadata are generated at build time via Nextra.

### Authoring Notes

- Prefer `nextra/components` (`Callout`, `Tabs`, `Bleed`) in MDX.
- LaTeX is available via KaTeX (enabled in Next config).
- Keep `_meta.js` files close to content to control sidebar and page titles.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oraxen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
