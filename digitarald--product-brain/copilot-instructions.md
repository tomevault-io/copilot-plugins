## product-brain

> > Demo workspace for agentic PM workflows at Contoso — outdoor gear e-commerce

# Contoso Product Brain

> Demo workspace for agentic PM workflows at Contoso — outdoor gear e-commerce

IMPORTANT: When adding new features, update README.md and prototypes/demo-narrative.

---

## Project Guidelines

### Folder Structure
| Folder | Purpose |
|--------|---------|
| `specs/` | PRDs with Problem Statement, Proposed Solution, Success Metrics, Open Questions |
| `insights/` | Synthesized research with quotes, recommendations, and spec backlinks |
| `research/` | Market analysis, competitive intel, user studies |
| `research/data/` | Raw CSV datasets for analysis |
| `roadmap/` | Quarterly plans, OKRs, prioritization |
| `prototypes/` | Disposable HTML demos — no build step |
| `analysis/notebooks/` | Jupyter notebooks for data exploration |

### Conventions
- **File naming**: `kebab-case.md` everywhere
- **Spec headers**: Use Markdown metadata (not YAML frontmatter): `**Status:** Draft`, `**Author:**`, `**Last Updated:**`, `**Stakeholders:**`
- **Cross-linking**: Specs link to `insights/`, insights link back to `specs/` — unlinked content is "unvalidated"
- **Prototype stack**: HTML + [Tailwind CDN](https://cdn.tailwindcss.com) + Alpine.js — stakeholders open files directly

### Design System — Adventure Editorial Dark Theme

**Colors:**
| Token | Value | Usage |
|-------|-------|-------|
| Dark base | `#0a0c0a` | Page background |
| Card bg | `rgba(17, 20, 17, 0.85)` | Glass cards |
| Border | `rgba(163, 171, 163, 0.12)` | Subtle borders |
| Text primary | `#e8ebe8` | Body text |
| Text secondary | `#a3aba3` | Muted text |
| Forest | `#4ade80` | Primary accent, success, CTAs |
| Sage | `#7a9b7e` | Brand accent |
| Ember | `#f97316` | Warning, secondary CTA |
| Earth | `#8b7b6b` | Muted accents |

**Typography:**
- **Display:** Source Serif 4 (editorial drama, headers)
- **Body:** Outfit (modern, readable)

**Card Pattern:**
```css
background: rgba(17, 20, 17, 0.85);
border: 1px solid rgba(163, 171, 163, 0.12);
backdrop-filter: blur(20px);
border-radius: 16px;
```

**Ambient Background:**
```css
background: 
  radial-gradient(ellipse 80% 50% at 50% -20%, rgba(74, 222, 128, 0.06) 0%, transparent 50%),
  radial-gradient(ellipse 60% 40% at 100% 80%, rgba(249, 115, 22, 0.04) 0%, transparent 50%),
  #0a0c0a;
```

---

## About Contoso

Contoso is an outdoor gear e-commerce retailer with 50K+ products across camping, hiking, climbing, and water sports. We're committed to sustainability: 40% of our catalog features eco-certified products, and we offset shipping emissions for all orders.

## What Is This Repo?

This is our **Product Brain** — a knowledge repository where Product Management maintains specs, research, insights, and roadmaps. Think of it as the single source of truth for product decisions.

## Active Product Areas

- **checkout-redesign** — Streamlining purchase flow, reducing cart abandonment
- **loyalty-program-v2** — Revamping rewards program with sustainability incentives

## Team Personas

- **Alex** (PM Lead) — Owns checkout-redesign, focuses on conversion metrics
- **Jordan** (UX Researcher) — Runs user studies, synthesizes feedback into insights
- **Sam** (Engineering Lead) — Technical feasibility, implementation estimates

## Integration

This repo integrates with:
- **GitHub Issues** — Tasks, bugs, and feature requests
- **GitHub Projects** — Sprint planning boards
- **Figma** — Design files linked from specs

---

*This is a demo workspace for conference presentation purposes.*

---
> Source: [digitarald/product-brain](https://github.com/digitarald/product-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
