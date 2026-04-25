## swipd-web

> Landing page and marketing website for the Swipd iOS app - a swipe-based affiliate shopping platform.

# Swipd Web

Landing page and marketing website for the Swipd iOS app - a swipe-based affiliate shopping platform.

## Tech Stack

- **Framework**: Next.js 15 (App Router)
- **Language**: TypeScript 5
- **React**: 19 (Server Components)
- **Styling**: Tailwind CSS 4 + shadcn/ui
- **Components**: Radix UI primitives
- **Forms**: React Hook Form + Zod validation
- **Icons**: Lucide React
- **Testing**: Playwright (E2E)
- **Analytics**: Google Analytics 4

## Project Structure

```
swipd-web/
├── app/                    # Next.js App Router
│   ├── layout.tsx          # Root layout (metadata, fonts, theme)
│   ├── page.tsx            # Landing page
│   └── globals.css         # Global styles + CSS variables
├── components/
│   ├── ui/                 # shadcn/ui components
│   │   ├── button.tsx
│   │   ├── card.tsx
│   │   ├── dropdown-menu.tsx
│   │   └── separator.tsx
│   ├── theme-provider.tsx  # next-themes wrapper
│   ├── theme-toggle.tsx    # Dark/light/system switcher
│   └── google-analytics.tsx
├── lib/
│   ├── constants.ts        # App metadata & config
│   └── utils.ts            # Utilities (cn function)
└── public/                 # Static assets
```

## Development

```bash
# Install dependencies
npm install

# Start dev server (port 3000)
npm run dev

# Production build
npm run build

# Start production server
npm start

# Lint
npm run lint
```

## Environment Variables

Copy `.env.example` to `.env.local`:

| Variable | Required | Description |
| -------- | -------- | ----------- |
| NEXT_PUBLIC_APP_STORE_URL | Yes | iOS App Store link |
| NEXT_PUBLIC_SITE_URL | Yes | Site URL for metadata |
| NEXT_PUBLIC_SITE_NAME | Yes | Site name |
| NEXT_PUBLIC_GA_MEASUREMENT_ID | No | Google Analytics ID |

## Key Features

- **Landing Page**: Hero, features grid, target users, CTA sections
- **Theme System**: Light/dark/system modes via next-themes
- **Responsive**: Mobile-first Tailwind design
- **Analytics**: Optional GA4 tracking
- **Accessible**: Radix UI primitives

## Styling

Uses OKLch color space CSS variables in `globals.css`:

- Theme colors: background, foreground, primary, secondary, accent, muted, destructive
- Automatic dark mode via `.dark` class
- Tailwind utilities with `tw-animate-css` for animations

## Adding shadcn/ui Components

```bash
npx shadcn@latest add [component-name]
```

Configuration in `components.json` uses:

- Style: new-york
- Path alias: @/*
- Icons: lucide

## Testing

E2E tests with Playwright:

```bash
npx playwright test
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/swipd-app) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
