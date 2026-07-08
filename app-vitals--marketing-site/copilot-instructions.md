## marketing-site

> - **Framework**: Astro (static site generation, no SSR)

# CLAUDE.md — App Vitals Marketing Site

## Stack
- **Framework**: Astro (static site generation, no SSR)
- **Styling**: Tailwind CSS
- **Hosting**: Vercel
- **Domain**: app-vitals.com

## Project Structure
```
src/
  components/   # Reusable .astro components
  layouts/      # Page layouts (BaseLayout.astro)
  pages/        # File-based routing
  styles/       # Global CSS (global.css)
public/         # Static assets (favicon, images, robots.txt)
planning/       # PRD, task breakdown, clarifying questions
```

## Conventions
- **Commits**: Conventional commits (`feat:`, `fix:`, `docs:`, `chore:`)
- **Branches**: Feature branches (`feat/av1-description`) + squash merge PRs
- **Never commit to main**
- **No runtime JS** unless absolutely needed (Astro islands only if required)
- **Mobile-first** responsive design
- **Files**: ~200 lines max per file

## Design Tokens
- **Background**: Deep navy `#080E1E` (navy-950), `#0F172A` (navy-900)
- **Accent**: Electric blue `#3B82F6`, Cyan `#22D3EE`, Violet `#8B5CF6`
- **Text**: White on dark, `text-white/40-50` for body, `text-white/25` for subtle
- **Display font**: Space Grotesk (Google Fonts) — headings, brand
- **Body font**: General Sans (Fontshare) — paragraphs, UI
- **Mono font**: JetBrains Mono — labels, tags, metadata
- **Max width**: `max-w-6xl` (1152px)
- **Aesthetic**: Premium editorial — grain texture overlay, animated gradient glow orbs, scroll reveals, gradient text, mouse-tracking card glow effects
- **Inspiration**: Linear.app, Stripe annual letters — dark, confident, memorable

## Content Rules
- NO client names (anonymize everything)
- NO specific pricing or rates
- NO email capture forms
- Cal.com link for booking: https://cal.com/app-vitals/discovery
- Co-founders: Dan McAulay + Dave O'Dell

## Commands
```bash
npm run dev      # Local dev server
npm run build    # Production build
npm run preview  # Preview production build
```

## Deploy
```bash
source /home/pi/.openclaw/workspace/.secrets.env
npx vercel --prod --token "$VERCEL_TOKEN_APP_VITALS"
```

---
> Source: [app-vitals/marketing-site](https://github.com/app-vitals/marketing-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
