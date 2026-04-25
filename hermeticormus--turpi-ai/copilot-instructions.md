## turpi-ai

> Spanish-first AI tools directory empowering Latinos to build, create, and compete globally. Educational platform philosophy: "teach people like they're five years old." 500M+ Spanish speakers deserve world-class tech education in their language.

# Turpi AI - Project Instructions

## Overview
Spanish-first AI tools directory empowering Latinos to build, create, and compete globally. Educational platform philosophy: "teach people like they're five years old." 500M+ Spanish speakers deserve world-class tech education in their language.

## Current State
- Next.js 16 + React 19 + Tailwind CSS 4
- 45+ tools across 11 categories
- Skill-level toggle system (Principiante/Intermedio/Avanzado) implemented
- Dark theme with amber/orange accent colors
- TurpiMascot (orange/black bird) as mascot

## Feature Roadmap

### Iteration 1: Skill-Level Toggle (Complete)
- [x] Global toggle in header with localStorage persistence
- [x] SkillContent type for adaptive content (tagline, description, turpiTake, limitations)
- [x] 5 tools converted (Claude, ChatGPT, Cursor, Vercel, Supabase)
- [x] Color-code toggle buttons by level (emerald/amber/violet)
- [x] Global theme system - entire UI recolors based on skill level
- [x] "Last updated" timestamps displayed on tool cards and detail pages
- [x] Skill-level content added to 19 key tools:
  - LLMs: Claude, ChatGPT, Gemini, Grok
  - Coding: Cursor, Claude Code, GitHub Copilot
  - Images: Midjourney, DALL-E, Ideogram
  - Hosting: Vercel, Netlify, Railway, Cloudflare Pages
  - Database: Supabase, Firebase, PlanetScale
  - Automation: n8n, Zapier, Make
- [ ] Add skill-level content to remaining ~26 tools (design, languages, frameworks, devtools, opensource)

### Iteration 1.5: Glossary System (Complete)
- [x] Interactive glossary with 25+ technical terms
- [x] Auto-detection of terms in tool descriptions
- [x] Dotted underline + tooltip on hover
- [x] Skill-level adaptive definitions (beginner/intermediate/advanced)
- [x] `/glosario` page with search and category filters
- [x] Categories: Desarrollo, IA, Web, Negocio
- [x] Related terms linking

**Key Files:**
- `src/data/glossary.ts` - Term definitions with skill-level content
- `src/components/GlossaryTerm.tsx` - Tooltip component
- `src/components/GlossaryText.tsx` - Auto-wraps text with glossary terms
- `src/app/glosario/page.tsx` - Glossary page

### Iteration 2: AI Mini-Assistant
- Claude API integration (Haiku model)
- Questions AND answers adapt to skill level
- "What do you want to build?" -> personalized recommendations
- Rate limiting (20 req/hour/IP)

### Iteration 3: Guided Project Workflow
**Vision**: When user states what they want to build, break it down into:
- Step-by-step instructions from zero to completion
- Tool installation guides
- Service connection walkthroughs
- Configuration details
- Verification checkpoints

Example: "I want to build a landing page"
1. Install Node.js (with verification)
2. Create project with Claude Code
3. Set up Vercel account
4. Connect GitHub repo
5. Deploy to Vercel
6. Configure custom domain (optional)

### Iteration 4: Educational Content
- /aprende section with AI fundamentals
- Demystify AI: what it is, what it isn't
- Express limitations clearly
- Cross-link from tools to educational content

### Iteration 5: Data Accuracy & Verification System
**Priority: HIGH** - Current tool information needs fact-checking

**Requirements:**
- Verify ALL tool information is 100% accurate and current
- Display prominent "last updated" timestamps per tool
- Build agent/skill system for automated fact-checking
- Use **Hermetic-Ormus-Archaeologist** pattern for deep research
- Cross-reference official sources (pricing pages, docs, changelogs)
- Flag outdated information automatically

**Data Points to Verify Per Tool:**
- Pricing tiers and limits (changes frequently)
- Feature availability
- API capabilities
- Free tier limitations
- Integration options
- Current version/status

**Agent Architecture:**
- Research agent: Fetches latest info from official sources
- Verification agent: Cross-checks against current data
- Update agent: Proposes changes with citations
- Review workflow: Human approval before publish

## Key Files
- `src/contexts/SkillLevelContext.tsx` - Global skill level state + ThemeColors
- `src/components/SkillToggle.tsx` - Color-coded toggle component
- `src/components/Header.tsx` - Shared header with toggle + glossary link
- `src/components/ToolContent.tsx` - Skill-adaptive tool display with glossary integration
- `src/components/ToolCard.tsx` - Tool card with theme colors and last-updated
- `src/components/CategoryCard.tsx` - Category card with theme colors
- `src/components/CategoryPageContent.tsx` - Theme-aware category page
- `src/data/tools.ts` - Tool definitions with SkillContent type
- `src/data/glossary.ts` - Glossary terms with skill-level definitions
- `src/components/GlossaryTerm.tsx` - Interactive term tooltip
- `src/components/GlossaryText.tsx` - Auto-wraps text with glossary terms
- `src/app/glosario/page.tsx` - Searchable glossary page

## Design System
- Background: stone-950
- Text: stone-100 (headings), stone-400 (body)
- Cards: stone-900 with stone-800 border

### Theme Colors (Dynamic based on skill level)
| Level | Primary | Hex |
|-------|---------|-----|
| Principiante | emerald-500 | #10b981 |
| Intermedio | amber-500 | #f59e0b |
| Avanzado | violet-500 | #8b5cf6 |

## Commands
```bash
npm run dev      # Development server
npm run build    # Production build
npm run lint     # Lint check
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HermeticOrmus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
