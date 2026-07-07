## shopify-theme-design

> This skill forces UNIQUE design direction for every project through the **Theme DNA** process.

# Shopify Theme Design Plugin

## Overview

This plugin provides a comprehensive workflow for creating distinctive, marketplace-quality Shopify themes. It emphasizes the "anti-AI-slop" approach - avoiding the generic patterns that plague AI-generated themes - while ensuring strong brand cohesion across all templates.

## Skills Included

| Skill | Trigger | Purpose |
|-------|---------|---------|
| shopify-theme-design | "shopify theme", "build a theme", "create a store theme", "e-commerce theme" | Full theme design workflow with Vibe Discovery and Theme DNA |

## Core Philosophy

### The Problem This Solves

AI-generated Shopify themes tend to converge on the same patterns:
- Dawn clones with slightly different colors
- Generic product grids everywhere
- Stock photo energy
- Feature bloat and overwhelming settings
- That recognizable "this is clearly Shopify" vibe

This skill forces UNIQUE design direction for every project through the **Theme DNA** process.

### The Theme DNA Mandate

**BEFORE any templates, you MUST:**
1. Run Vibe Discovery (7 questions)
2. Synthesize a unique aesthetic direction
3. Write out a Theme DNA document
4. Pass the Freshness Check

No shortcuts. No "I'll figure out the style as I go."

## Key Patterns

### Priority Stack
1. Brand cohesion across all touchpoints
2. Homepage hero + collection grid
3. Product page
4. Mobile experience

### The Design-First Approach
We don't start with code - we start with vision. The Theme DNA document becomes the source of truth that every template references.

### Anti-Convergence Rules
1. **No hex code memory** - Generate colors fresh from reference
2. **Font rotation required** - Can't reuse display fonts across projects
3. **Collision must show** - Both influences must be visible
4. **Wildcard is mandatory** - Every theme needs one "doesn't fit" element
5. **Name it** - Unnamed vibes become generic

## Template Hierarchy

**Core (Build First):**
1. layout/theme.liquid (DNA becomes CSS vars)
2. Homepage (50% of effort on hero)
3. Collection page
4. Product page
5. Cart

**Secondary (As Needed):**
- Blog templates
- Page templates
- Customer accounts
- Search, 404

## Anti-AI-Slop Checklist

**Fonts to AVOID:**
- System UI / -apple-system
- Helvetica Neue
- Assistant (Dawn default)

**Icons to AVOID:**
- Heroicons (too common in Shopify)
- Feather icons

**Layouts to AVOID:**
- 4-column product grids everywhere
- Centered everything
- The Dawn grid

**Use INSTEAD:**
- Fonts: Fraunces, Instrument Serif, Clash Display, Outfit, Satoshi
- Icons: Phosphor, Iconify Solar, Tabler, custom SVGs
- Layouts: Asymmetric grids, full-bleed moments, varied rhythm

## Development Workflow

When this skill triggers:

1. **Vibe Discovery** - Ask the 7 questions
2. **Theme DNA** - Write the full document before any code
3. **Pass Freshness Check** - Verify uniqueness
4. **HTML Preview** - Build static HTML mockups first
5. **Design Approval** - Get explicit approval before Liquid conversion
6. **Liquid Conversion** - Convert approved HTML to Shopify templates
7. **Polish and validate** - Run through all checklists
8. **Marketplace prep** - If submitting, full review checklist

### HTML-First Approach

We build in HTML before touching Liquid:
- Faster iteration without Shopify complexity
- Easy to preview in browser (`bun run preview/serve.ts`)
- Design approval checkpoint before conversion
- Clear mapping from HTML sections to Liquid sections

```
preview/
├── index.html          # Homepage mockup
├── collection.html     # Collection page
├── product.html        # Product page
├── cart.html           # Cart page
├── css/theme.css       # All styles (DNA tokens)
└── serve.ts            # Bun preview server
```

## CLI Integration

```bash
# Start new theme
shopify theme init [name]

# Development
shopify theme dev --store [store.myshopify.com]

# Validate
shopify theme check

# Package for submission
shopify theme package
```

---
> Source: [dylanreed/shopify-theme-design](https://github.com/dylanreed/shopify-theme-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
