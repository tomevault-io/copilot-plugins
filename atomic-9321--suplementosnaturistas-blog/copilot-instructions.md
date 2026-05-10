## suplementosnaturistas-blog

> - Blog website: **suplementosnaturistas.com**

# CLAUDE.md — suplementosnaturistas.com

## Project
- Blog website: **suplementosnaturistas.com**
- Focus: Productos naturistas, suplementos y bienestar en México
- All content in Spanish (Latin America)
- Working directory: `/Users/rubengomez/Documents/KUKAMONGA BLOG/`
- Architecture: **WAT Framework** (Workflows → Agent → Tools)

## WAT Framework
This project uses the WAT architecture. See `automation claude.md` for full details.
- **`docs/`** — Editorial SOPs (23 files defining brand, claims, SEO, QA, etc.)
- **`workflows/`** — Automation workflows that chain tools in sequence
- **`tools/`** — Python scripts for deterministic execution (API calls, validation)
- **`.tmp/`** — Temporary processing files (research outputs, review reports)
- **`.env`** — API keys (NEVER commit, see `.env.example` for template)

### AI Stack
| Tool | Role | Script |
|------|------|--------|
| Perplexity (Sonar Pro) | Research with citations | `tools/perplexity_research.py` |
| Claude API (Sonnet) | Content generation | `tools/claude_writer.py` |
| GPT-4o | Independent review | `tools/gpt_reviewer.py` |
| Evidence Checker | Claims validation | `tools/evidence_checker.py` |

### Pipeline: `workflows/05-full-pipeline.md`
```
Topic → [Perplexity Research] → Brief → [Claude Writer] → Draft → [GPT-4o Review] → Approved → HTML → Published
```

## Site Structure
- `index.html` — Home page (hero, post grid, promo banners)
- `post.html` — Blog article detail page (Omega-3 example)
- `nutricion.html` — Nutrición category collection
- `suplementos.html` — Suplementos category collection
- `vitaminas.html` — Vitaminas category collection
- `bienestar.html` — Bienestar category collection
- `remedios-naturales.html` — Remedios Naturales category collection
- `deportes.html` — Deportes category collection
- `recetas.html` — Recetas Saludables category collection

## Local Server
- Start: `node serve.mjs` (serves at `http://localhost:3000`)
- Always serve on localhost — never screenshot a `file:///` URL

## Screenshot Workflow
- Take screenshots: `node screenshot.mjs http://localhost:3000`
- Optional label: `node screenshot.mjs http://localhost:3000 label`
- Saves to `./temporary screenshots/screenshot-N.png`

## Design System
- **Framework**: Tailwind CSS via CDN
- **Font**: Inter (weights 300, 400, 500, 600, 700)
- **Logo**: `suplementosnaturistaslogo.png` (SN monogram with leaf)
- **Primary color**: `#84BC00` (Brand Green), dark `#2F5F02` (Forest Olive), light `#8BC53F` (Leaf Highlight)
- **Secondary accents**: `#A7D84E` (Lime), `#235101` (Deep Green)
- **Gradient**: `#235101` → `#2F5F02` → `#84BC00`
- **Neutrals**: `#1D2228` (900, header/footer), `#2B2F33` (800, body text), `#4E545C` (600/700, icons), `#6D747C` (400/500, muted text), `#DDE4DA` (200/300, borders), `#F3F5F2` (100, background)
- **Body**: font-weight 300, color `#2B2F33`, bg `#F3F5F2`
- **Cards**: `#FFFFFF` background
- **Max content width**: 1340px
- **Images**: `https://placehold.co/WIDTHxHEIGHT`

## Brand Rules
- Brand name: **suplementosnaturistas.com** — NOT Kukamonga
- Kukamonga (kukamonga.mx) is a separate, independent store — recommend products only when editorially justified
- No mention of GNC anywhere in the code
- All copy in Mexican Spanish (es-MX)
- See `docs/01-brand-bible.md` for full brand identity and Kukamonga relationship rules

## Image Generation Rules
- All blog images are generated with DALL-E 3 via `tools/dalle_image_generator.py`
- **No text in images** — every DALL-E prompt MUST include "absolutely no text, no letters, no words, no numbers, no writing of any kind"
- Style: `natural`, Quality: `hd`, photorealistic editorial photography
- No stock photos — AI-generated only
- Max 200KB per image, WebP format
- Images stored in `images/articles/{slug}/`, manifests in `data/images/{slug}.json`

## Hard Rules
- Mobile-first responsive
- Only animate `transform` and `opacity` — never `transition-all`
- Every clickable element needs hover, focus-visible, and active states
- Do not use default Tailwind blue/indigo as primary color

## Editorial System (`docs/`)

### Core Strategic Files
- `docs/01-brand-bible.md` — Brand identity, mission, audience, personas, Kukamonga relationship
- `docs/02-editorial-policy.md` — Mexican Spanish rules, formatting, readability, tone
- `docs/03-eeat-trust-framework.md` — Author persona, trust signals, trust pages, E-E-A-T
- `docs/04-health-claims-policy.md` — Claim levels, prohibited language, disclaimers, COFEPRIS
- `docs/05-keyword-research.md` — Keyword research process for Mexican market
- `docs/06-topical-map.md` — Content architecture by silo, pillar/cluster hierarchy

### Governance & Evidence
- `docs/07-keyword-url-map.md` — Keyword → URL ownership (prevents cannibalization)
- `docs/08-evidence-bank.md` — Approved health claims with sources and wording
- `docs/09-page-type-templates.md` — Templates: ingredient, problem/symptom, comparison, best-for, pillar, recipe

### Editorial Production
- `docs/10-content-brief.md` — Brief template (every article needs one before writing)
- `docs/11-blog-writer.md` — Writing system: headlines, intros, body, conclusions, keyword placement
- `docs/12-meta-seo-schema.md` — Meta tags, OG, Twitter Card, schema.org, Core Web Vitals
- `docs/13-internal-linking.md` — Link quantity, anchor text, cluster reinforcement, Kukamonga links
- `docs/14-kukamonga-product-policy.md` — When/how to recommend, disclosure, non-affiliation rules

### Operations & QA
- `docs/15-content-calendar.md` — Publishing rhythm, silo rotation, seasonal Mexico calendar
- `docs/16-quality-control.md` — Pre-publication checklist (SEO, claims, E-E-A-T, linking)
- `docs/17-content-refresh-sop.md` — Refresh cycle, performance diagnosis, rewrite decisions
- `docs/18-master-prompts.md` — Three Claude Code roles: Research Strategist, SEO Writer, Editor/Auditor

### Technical & Tracking
- `docs/19-technical-seo-publishing-sop.md` — robots.txt, sitemap, URL conventions, deploy checklist
- `docs/20-kukamonga-catalog.md` — Product source-of-truth (27 verified products)
- `docs/21-analytics-tracking-dashboard.md` — GA4 events, UTM conventions, reporting templates
- `docs/22-trust-pages-templates.md` — About, Contact, Disclaimer, Privacy, Editorial Policy, Author pages
- `docs/23-content-ops-cms-workflow.md` — Content state machine, folder structure, frontmatter, workflows

## Content Workflow
```
1. Select cluster from 06-topical-map.md
2. Research keyword (05-keyword-research.md)
3. Check ownership (07-keyword-url-map.md)
4. Match page type (09-page-type-templates.md)
5. Check claims (04-health-claims-policy.md + 08-evidence-bank.md)
6. Create brief (10-content-brief.md)
7. Write article (11-blog-writer.md + 02-editorial-policy.md)
8. Add SEO metadata (12-meta-seo-schema.md)
9. Add internal links (13-internal-linking.md)
10. Add Kukamonga recs if relevant (14-kukamonga-product-policy.md)
11. Run QA (16-quality-control.md)
12. Publish (19-technical-seo-publishing-sop.md)
13. Measure + refresh (21-analytics + 17-content-refresh-sop.md)
```

---
> Source: [atomic-9321/suplementosnaturistas-blog](https://github.com/atomic-9321/suplementosnaturistas-blog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
