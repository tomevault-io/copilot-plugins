## cicosts-app

> Next.js 14 frontend for the CICosts platform.

# CICosts App

Next.js 14 frontend for the CICosts platform.

## URLs

| Environment | URL |
|-------------|-----|
| Production | https://app.cicosts.dev |
| Development | https://dev.cicosts.dev |

## Tech Stack

- **Framework**: Next.js 14 (App Router)
- **Styling**: Tailwind CSS
- **Components**: shadcn/ui
- **Auth**: GitHub OAuth (via backend)
- **Hosting**: Vercel
- **Monitoring**: Sentry

## Project Structure

```
src/
├── app/
│   ├── page.tsx              # Landing page
│   ├── blog/                 # MDX blog
│   ├── features/             # Feature pages
│   ├── pricing/              # Pricing page
│   ├── resources/            # Changelog, case studies
│   ├── login/                # Login page
│   ├── setup/                # GitHub App callback
│   └── dashboard/            # Authenticated dashboard
│       ├── page.tsx          # Overview
│       ├── workflows/        # Workflows list
│       ├── alerts/           # Alerts management
│       └── settings/         # User settings
├── components/
│   ├── ui/                   # shadcn/ui components
│   ├── marketing/            # Header, footer
│   └── dashboard/            # Dashboard components
├── content/
│   └── blog/                 # MDX blog posts
└── lib/
    ├── api.ts                # API client
    ├── auth.tsx              # Auth context
    └── dashboard-context.tsx # Dashboard state

e2e/                          # Playwright E2E tests (23 passing)
```

## Pages

### Marketing (Public)
- `/` - Landing page with hero, features, pricing
- `/blog` - Blog listing
- `/blog/[slug]` - Blog post (MDX)
- `/features/*` - Feature pages
- `/pricing` - Interactive pricing
- `/resources/changelog` - Version history
- `/resources/case-studies` - Customer stories
- `/privacy`, `/terms` - Legal

### Dashboard (Authenticated)
- `/dashboard` - Overview with charts
- `/dashboard/workflows` - Workflow list
- `/dashboard/alerts` - Alert management
- `/dashboard/settings` - User settings

## Running Locally

```bash
# Install dependencies
npm install

# Run dev server
npm run dev

# Run tests
npm test

# Run E2E tests
npx playwright test

# Build
npm run build
```

## Environment Variables

```env
NEXT_PUBLIC_API_URL=https://dev-api.cicosts.dev
NEXT_PUBLIC_GITHUB_CLIENT_ID=xxx
SENTRY_DSN=xxx
```

## Deployment

Push to main triggers CI/CD pipeline:
1. Lint (ESLint, TypeScript)
2. Test (Vitest)
3. Build
4. Deploy to dev (Vercel preview)
5. E2E tests (Playwright)
6. Deploy to prod (requires approval)

## Brand Colors

```css
--ink-black: #040f0f;
--forest-green: #248232;
--medium-jungle: #2ba84a;
--jet-black: #2d3a3a;
--porcelain: #fcfffc;
```

## Pricing

| Plan | Price | Features |
|------|-------|----------|
| Free | $0 | 3 repos, 30 days history |
| Pro | $29/mo | Unlimited repos, 365 days |
| Team | $59/mo | + 5 team members |

## Case Study Companies

For consistency, use these names across landing page and case studies:
- Sales Amplifiers (Sarah Chen, VP of Engineering)
- Ecoden (Marcus Johnson, DevOps Lead)
- Devcraft Labs (Emily Rodriguez, Technical Director)

---
> Source: [phonotechnologies/cicosts-app](https://github.com/phonotechnologies/cicosts-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
