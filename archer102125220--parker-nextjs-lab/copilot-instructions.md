## parker-nextjs-lab

> Build and Development Tooling Standards (Webpack Enforcement)

# Build & Dev Tooling (Webpack Required)

Due to SCSS `:export` syntax incompatibility with Turbopack, you MUST use the following scripts:
- **Dev**: `yarn dev:webpack` (HTTP) or `yarn dev-https:webpack` (HTTPS)
- **Build**: `yarn build:webpack`

**Do NOT use `yarn dev` or `yarn build` as they may attempt to use Turbopack or lack necessary configurations.**

**Environment Check**: When starting the development server, ALWAYS check if `NEXT_PUBLIC_API_BASE` and `NEXT_PUBLIC_DOMAIN` in `.env` match the port/domain settings in `package.json` scripts. If there is a mismatch (e.g., .env port 3000 vs script port 3001), OR if `.env` is gitignored and unreadable by the IDE, you MUST wait for user confirmation before proceeding.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/archer102125220) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
