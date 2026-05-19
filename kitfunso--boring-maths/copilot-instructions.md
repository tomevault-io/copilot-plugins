## boring-maths

> > Shared principles live in `C:\Users\skf_s\clawd\AGENTS.md`. Read that for orchestration patterns, safety rules, and workflow standards.

# AGENTS.md - Boring Maths

> Shared principles live in `C:\Users\skf_s\clawd\AGENTS.md`. Read that for orchestration patterns, safety rules, and workflow standards.

## Project

- **Owner:** Keith
- **Site:** https://boring-math.com
- **Stack:** Astro 5 + Preact (TSX calculators) + Tailwind CSS
- **Purpose:** UK personal finance calculators. SEO-driven traffic, AdSense monetisation.

## Key Paths

- **Calculators:** `src/components/calculators/{Name}/{Name}Calculator.tsx`
- **Pages:** `src/pages/calculators/*.astro`
- **Layouts:** `src/layouts/CalculatorLayout.astro`, `src/layouts/BaseLayout.astro`
- **UI components:** `src/components/ui/`
- **Ads:** `src/components/common/AdUnit.astro`
- **Brand voice:** `C:\Users\skf_s\clawd\brands\boring-maths\voice.md`
- **Brand positioning:** `C:\Users\skf_s\clawd\brands\boring-maths\positioning.md`
- **Marketing log:** `C:\Users\skf_s\clawd\memory\boring-maths-marketing.md`

## Commands

```
# Build
npm run build

# Lint
npm run lint

# Deploy
npx wrangler pages deploy dist --project-name=boring-maths

# Git
git add -A; git commit -m "Kit: <desc>"; git push
```

## Calculator URL Format

`https://boring-math.com/calculators/{slug}`

Examples:
- boring-math.com/calculators/uk-tax-calculator
- boring-math.com/calculators/uk-pension-calculator
- boring-math.com/calculators/compound-interest-calculator

## Voice (for content, X posts, copy)

Bitter but funny British accountant. Dry humour. Deadpan. NOT a brand.

- Lead with specific numbers
- Cynical about HMRC, the tax system, student loans
- Self-deprecating is good
- NO hashtags, NO emoji, NO motivational tone
- BANNED: optimize, maximize, leverage, empower, journey, comprehensive, "Did you know", "Here's a tip"
- British English always (colour, favourite, optimise if you must use it)

## New Calculator Checklist

1. Create `src/components/calculators/{Name}/{Name}Calculator.tsx`
2. Create `src/pages/calculators/{slug}.astro`
3. Add schema markup in frontmatter
4. Verify `npm run build` passes
5. Check calculator renders at `localhost:4321/calculators/{slug}`
6. Deploy

## Safety

- `trash` > `rm`. No destructive commands without confirmation.
- British English spelling in all user-facing content.
- No AI slop words (comprehensive, robust, harness, landscape).
- Valid UI colours only: blue, green, purple, red, yellow, coral, violet, ocean, amber.

## Git

Commit format: `Kit: <brief description>`
Include model attribution trailer: `AI-Model: claude-sonnet-4-6`

---
> Source: [kitfunso/boring-maths](https://github.com/kitfunso/boring-maths) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
