## base

> - Static landing site with 3 pages: Home, Reports (/reports), Careers (/careers)


## About Kokorovsky dvur

Kokorak

## Website Purpose

- Static landing site with 3 pages: Home, Reports (/reports), Careers (/careers)
- Professional showcase of team, research work, and job opportunities
- Clean, sleek design without fancy animations or frameworks

## Technical Stack

- **Next.js 15** with TypeScript
- **Static export** - no server-side features
- **Pure CSS** with CSS custom properties (no Tailwind, styled-components, etc.)
- **Next.js Image components** for all images (never use <img> tags)
- **Placeholder images** from placehold.co for all graphics
- **MDX support** for dynamic report content with static generation

## MDX Reports System

The reports section uses MDX files stored in the `/content` directory to create rich, formatted research reports. Each MDX file contains frontmatter metadata (title, date, description, author, tags) and markdown content. The system automatically:

- Scans the content directory for `.mdx` files at build time
- Generates static routes for each report using `generateStaticParams`
- Parses frontmatter to extract metadata for the reports listing page
- Renders markdown content with custom styled components
- Maintains single source of truth - reports are defined only in MDX files, with the listing page dynamically populated from file metadata

## Design Principles

- Clean, modern, professional aesthetic
- System fonts and simple typography
- Consistent spacing and responsive grid layouts
- Subtle shadows and rounded corners
- Blue accent color (#2563eb) with neutral grays
- Navigation with active state underlines

## Deployment

- **Cloudflare** and **Google Cloud Storage (GCS)**
- Must pass `pnpm check` (ESLint + Knip + TypeScript)
- Static build with `next build` and `next export`

---
> Source: [hnykda/kokorovsky-dvur](https://github.com/hnykda/kokorovsky-dvur) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
