## opportunityzonespecialists-com

> Real estate website for Dr. Jan Duffy (NEVER "Janet").

# heyberkshire.com - Project Memory & Rules (February 2026)

## Project Context

Real estate website for Dr. Jan Duffy (NEVER "Janet").
- Agent: Dr. Jan Duffy, License S.0197614.LLC
- Brokerage: Berkshire Hathaway HomeServices Nevada Properties
- Markets: Las Vegas, Henderson, Summerlin
- Phone: (702) 500-1942

## Tech Stack

- **Frontend:** Next.js 14 (App Router), React, TypeScript, Tailwind CSS, shadcn/ui
- **Deployment:** Vercel (primary), Cloudflare Pages (optional)
- **Edge:** Cloudflare Workers
- **AI:** Anthropic Claude 3.5 Sonnet
- **CRM:** Follow Up Boss (native integrations)
- **IDX/MLS:** RealScout (native FUB integration)
- **Scheduling:** Calendly

## Platform Integrations (Already Connected)

### ✅ Follow Up Boss
- Full REST API with webhooks
- **Native RealScout integration** (don't duplicate!)
- Built-in automation, action plans, drip campaigns
- Lead deduplication built-in

### ✅ RealScout
- **Native Follow Up Boss sync** (2-way, automatic)
- AI-powered property matching (Dec 2025 release)
- Natural language search built-in
- Activity tracking built-in
- Don't build custom sync - use native integration!

### ✅ Calendly
- Widget works out of box
- Email notifications built-in
- Calendar sync built-in

## 2026 Best Practices Implemented

### Performance
- Lighthouse scores: 95+ (Performance, Accessibility, SEO)
- Core Web Vitals: LCP <2.5s, CLS <0.1, FID <100ms
- Images: AVIF/WebP with 1-year cache
- Edge caching: 80-90% hit rate

### Infrastructure
- GitHub Actions: Automated Lighthouse CI, preview deployments
- Cloudflare Workers: Custom caching, security headers, image optimization
- Bundle analysis: Track size regressions
- TypeScript: Strict mode

### API Optimization
- Claude AI: Prompt caching (90% cost savings)
- Rate limiting: Prevent 429 errors
- Response caching: Avoid duplicate calls
- Cost tracking: Monitor spending

### Accessibility
- WCAG 2.1 compliant
- `prefers-reduced-motion` support
- Keyboard navigation
- ARIA attributes

## Critical Rules

### 1. Don't Duplicate Platform Features

**BEFORE building any integration:**
- Check if platform already has the feature
- Check for native integrations between platforms
- Research recent updates (2025-2026)

**Examples of what NOT to build:**
- ❌ RealScout → FUB sync (native integration exists)
- ❌ Property recommendations (RealScout AI does this)
- ❌ Activity tracking (RealScout has this)
- ❌ FUB automation wrappers (FUB has action plans)
- ❌ Custom scheduling (Calendly works fine)

**What TO build (genuine enhancements):**
- ✅ Rate limiting (APIs don't provide this)
- ✅ Response caching (reduce duplicate calls)
- ✅ Cost optimization (prompt caching)
- ✅ Custom edge logic (Cloudflare Workers)

### 2. Performance First

- Always use Next.js `<Image>` component
- Enable AVIF/WebP formats
- Implement `useReducedMotion` for animations
- Add ARIA attributes and keyboard nav
- Set appropriate cache headers

### 3. Claude AI Cost Optimization

```typescript
// ✅ DO: Use consistent system prompts (cached)
systemPrompt: realEstateAgentTemplate.system, // Same every time = cached!

// ❌ DON'T: Dynamic prompts (no cache = 10x cost)
systemPrompt: `You are ${agent}...`, // Changes = no cache
```

### 4. Type Safety

- Use TypeScript strict mode
- Define interfaces for all API responses
- No `any` types (use `unknown` if needed)

## Common Pitfalls to Avoid

1. **Over-abstracting** - Don't wrap APIs that work fine
2. **Premature optimization** - Measure before optimizing
3. **Duplicating platforms** - Check docs first!
4. **Breaking working integrations** - RealScout ↔ FUB already works
5. **Building features vs optimizing** - Focus on speed, cost, quality

## File Organization

```
app/                    # Next.js App Router
├── api/               # API routes
│   ├── claude/        # Claude AI endpoints
│   └── webhooks/      # Webhook handlers
components/            # React components
├── ui/               # shadcn/ui components
├── forms/            # Form components
├── sections/         # Page sections
└── layouts/          # Layout components
lib/                   # Utility libraries
├── claude/           # Claude AI client & templates
└── fub/              # Follow Up Boss client (rate limiting only)
hooks/                 # Custom React hooks
workers/               # Cloudflare Workers
.github/workflows/     # CI/CD automation
.cursor/rules/         # Cursor rules (this file)
```

## Development Commands

```bash
npm run dev              # Start dev server
npm run build            # Production build
npm run analyze          # Bundle analysis
npm run lighthouse       # Performance audit
npm run cloudflare:deploy  # Deploy workers
```

## Environment Variables

```env
# Required for AI (optional)
ANTHROPIC_API_KEY=sk-ant-...

# Required for Cloudflare (optional)
CLOUDFLARE_API_TOKEN=...
CLOUDFLARE_ACCOUNT_ID=...

# Optional - FUB
FUB_API_KEY=...
FUB_SYSTEM_KEY=...
```

## Key Learnings (February 2026)

1. **RealScout + FUB are already integrated** - Don't build custom sync
2. **Claude prompt caching** - Biggest cost optimization (90% savings)
3. **Cloudflare Workers** - Best for custom edge logic
4. **GitHub Actions** - Automate quality checks
5. **Check platform docs FIRST** - Avoid building duplicates

## What Makes This Site Unique

Focus on these instead of duplicating platforms:
- YOUR local Las Vegas/Henderson expertise
- YOUR content (blog posts, guides)
- YOUR brand (Dr. Jan Duffy's reputation)
- YOUR client relationships

Platforms (RealScout, FUB) handle the commodity features.
You provide the unique value.

---

Last Updated: February 13, 2026

---
> Source: [LetMeHelpYouREALTY/opportunityzonespecialists.com](https://github.com/LetMeHelpYouREALTY/opportunityzonespecialists.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
