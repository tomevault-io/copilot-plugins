## nexus-site

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Official Nexus AI website — marketing homepage, MDX documentation, auto-generated API reference (from Protobuf descriptors), and cross-platform download page. Deployed on Vercel.

## Commands

```bash
pnpm install          # Install dependencies
pnpm dev              # Start dev server (http://localhost:3000)
pnpm build            # Production build
pnpm lint             # ESLint via next lint
pnpm vitest           # Run tests

# Code generation (requires buf CLI and sibling nexus-ai repo)
pnpm gen:proto        # Build protobuf descriptor from ../nexus-proto/proto → public/api-data/descriptor.json
pnpm gen:api-data     # Parse descriptor → public/api-data/api.json
pnpm gen:context      # Generate public/nexus-context.json for AI assistants
pnpm gen:llms         # Generate public/llms.txt for LLM consumption
pnpm gen              # Runs gen:api-data + gen:context + gen:llms
```

## Tech Stack

- Next.js 15 (App Router) + React 19 + TypeScript (strict)
- Tailwind CSS 4 + shadcn/ui (base-nova style) + lucide-react icons
- Fumadocs (fumadocs-core, fumadocs-mdx, fumadocs-ui) for MDX documentation
- next-intl v4 for i18n (zh-CN default, en secondary)
- next-themes for dark/light mode
- Vitest for testing
- Vercel deployment (`vercel.json`)

## Architecture

### Routing (App Router with `[locale]` segment)

```
app/
  layout.tsx                    Root layout (pass-through, no <html>)
  globals.css                   Tailwind + shadcn CSS variables + fumadocs theme
  [locale]/
    layout.tsx                  Main layout: Inter font, ThemeProvider, NextIntlClientProvider, Navbar
    (marketing)/
      layout.tsx                Children + Footer wrapper
      page.tsx                  Homepage (hero + feature cards)
      download/page.tsx         Download page (reads public/latest-releases.json)
    api/page.tsx                API reference documentation page (NOT an API route)
    docs/
      layout.tsx                Fumadocs DocsLayout with sidebar + search
      [[...slug]]/page.tsx      Dynamic MDX page rendering
      search/route.ts           Fumadocs search API endpoint
```

The root `app/layout.tsx` is a pass-through. The real HTML document is `app/[locale]/layout.tsx`.

### Components

```
components/
  layout/
    navbar.tsx          Sticky header, nav links, locale switcher, theme toggle, mobile menu
    footer.tsx          Footer with links
    locale-switcher.tsx <select> dropdown for zh-CN / en
  api/
    api-client.tsx      Main client component for API reference (~778 lines)
    search-dialog.tsx   Cmd+K search for endpoints, schemas, enums
    service-sidebar.tsx Collapsible sidebar for service navigation
    service-section.tsx Service section renderer
    method-detail.tsx   Individual method documentation
    fields-table.tsx    Field table with oneof grouping, validation display
    option-badge.tsx    Badge for agentOnly/userOnly/skipAuth/sensitive
  download/
    download-client.tsx Platform detection, release cards, CLI downloads
  theme-provider.tsx    next-themes wrapper
  ui/
    button.tsx          shadcn/ui Button (base-ui/react + CVA)
```

### Internationalization

Two systems work together:

1. **next-intl** — UI strings and locale routing:
   - `lib/i18n-config.ts`: locales = `["zh-CN", "en"]`, defaultLocale = `"zh-CN"`
   - `lib/navigation.ts`: `localePrefix: "as-needed"` (zh-CN has no URL prefix, en uses `/en/...`)
   - `messages/zh-CN.json` and `messages/en.json`: flat JSON translation keys

2. **Fumadocs i18n** — MDX content:
   - `lib/fumadocs-i18n.ts`: `defineI18n({ defaultLanguage: "zh-CN", languages: ["zh-CN", "en"], hideLocale: "default-locale", parser: "dir" })`
   - Content organized as `content/docs/{locale}/...`

### Content (MDX)

```
content/docs/
  en/
    meta.json
    index.mdx
    agent/ (quickstart, webhook, websocket, messages)
    concepts/ (authentication, conversations, message-system, sync)
  zh-CN/
    (mirrors en/ structure with Chinese translations)
```

### Code Generation Scripts

Scripts in `scripts/` run via `tsx`:

1. **`gen:proto`** — `buf build` in `../nexus-proto/proto` → `public/api-data/descriptor.json`
2. **`gen:api-data.ts`** — Parses descriptor → `public/api-data/api.json` (services, methods, messages, enums with source comments)
3. **`gen:context.ts`** — API summary + docs index → `public/nexus-context.json`
4. **`gen:llms-txt.ts`** — Site structure → `public/llms.txt`

### Static Data (public/)

```
public/api-data/
  descriptor.json    Raw protobuf FileDescriptorSet
  descriptor.binpb   Binary protobuf descriptor
  api.json           Parsed API data
public/
  latest-releases.json  Release data for download page
  nexus-context.json    AI-friendly context
  llms.txt              LLM-friendly site description
```

### Path Aliases

- `@/*` → project root (configured in `tsconfig.json`)

### Testing

Tests in `lib/__tests__/` using Vitest:
- `api-data-integrity.test.ts` — Validates api.json structure and completeness
- `i18n-completeness.test.ts` — Ensures zh-CN translation keys are complete
- `platform.test.ts` — UA string parsing for platform/arch detection
- `releases-schema.test.ts` — Validates latest-releases.json schema

## Key Conventions

- **Default locale is zh-CN**, not en. English is secondary.
- **`localePrefix: "as-needed"`**: zh-CN URLs have no prefix, en uses `/en/`.
- Fumadocs integration: `next.config.ts` chains `withMDX(withNextIntl(nextConfig))`.
- No custom ESLint or Prettier config — relies on eslint-config-next defaults.
- Inter font via `next/font/google`. CSS variables use oklch color space.
- `api/` and `download/` pages read JSON data at build time via `readFileSync`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pinealctx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
