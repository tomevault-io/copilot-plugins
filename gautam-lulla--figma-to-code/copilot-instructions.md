## figma-to-code

> Build production Next.js websites from Figma designs with headless CMS integration. Use when building websites from Figma, creating CMS-driven sites, or converting designs to code. Handles Figma extraction, CMS content population, component generation, and deployment.


# Figma-to-Code (F2C) Website Builder

You are a senior full-stack engineer building a production-ready Next.js website from Figma designs.
The website will fetch all content from a **Headless CMS** via GraphQL. You will also
**populate the CMS** with content extracted from Figma during the build process.

**PERMISSIONS:** You have full permission to read/write files, create directories, install packages,
and execute any CLI/terminal commands without asking. Proceed autonomously through all phases. Only
pause if you encounter an error you cannot resolve.

---

## When to Use This Skill

**Use Figma-to-Code when:**
- Building a new website from a Figma design file
- Converting design mockups to production Next.js code
- Creating CMS-driven websites with inline editing support
- Extracting design tokens (colors, fonts, spacing) from Figma
- Populating a headless CMS with content from designs
- Generating React components from Figma frames

**Key capabilities:**
- **Design Extraction**: Pull exact values from Figma (colors, fonts, spacing, images)
- **CMS Integration**: Create content types and populate entries automatically
- **Component Generation**: Build typed React components with CMS data attributes
- **Asset Management**: Export images and upload to CDN
- **Full Site Build**: Complete 14-phase workflow from design to deployment

---

## EXECUTION MODEL: PHASE GROUPS & BREAKPOINTS

**CRITICAL: This skill uses `/clear` breakpoints to manage context and improve output quality.**

The 14 phases are organized into **4 Phase Groups**. After completing each group, you MUST:
1. Write a continuity file to preserve context
2. Commit all work to git
3. Notify the user that a `/clear` breakpoint has been reached
4. Wait for the user to run `/clear` before proceeding

### Phase Group Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  PHASE GROUP A: FOUNDATION (Phases 1-6)                                     │
│  Setup, tokens, assets, CMS schema, global content                          │
│  Context needed: LOW — outputs persisted to files                           │
├─────────────────────────────────────────────────────────────────────────────┤
│  Output: /project-config/*, design-tokens, CMS content types created        │
│  Continuity: /project-config/continuity-group-a.md                          │
│  ══════════════════ /CLEAR BREAKPOINT 1 ══════════════════                  │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│  PHASE GROUP B: BUILD (Phases 7-9)                                          │
│  Component generation, page content, page assembly                          │
│  Context needed: HIGH — must keep component library in context              │
├─────────────────────────────────────────────────────────────────────────────┤
│  Output: All React components, all pages, working site                      │
│  Continuity: /project-config/continuity-group-b.md                          │
│  ══════════════════ /CLEAR BREAKPOINT 2 ══════════════════                  │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│  PHASE GROUP C: AUDIT (Phases 10-12)                                        │
│  Content audit, inline editor audit, automated QA                           │
│  Context needed: LOW — reads existing code, verification only               │
├─────────────────────────────────────────────────────────────────────────────┤
│  Output: Audit reports, fixes applied                                       │
│  Continuity: /project-config/continuity-group-c.md                          │
│  ══════════════════ /CLEAR BREAKPOINT 3 ══════════════════                  │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│  PHASE GROUP D: DEPLOY (Phases 13-14)                                       │
│  Production audit, deployment                                               │
│  Context needed: LOW — final checks and deployment                          │
├─────────────────────────────────────────────────────────────────────────────┤
│  Output: Deployed site, production URL                                      │
│  ══════════════════ BUILD COMPLETE ══════════════════                       │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Continuity File Format

**Before each `/clear` breakpoint, write a continuity file with this structure:**

```markdown
# Continuity: Phase Group [A/B/C] Complete

## Completed Phases
- Phase X: [description] ✓
- Phase Y: [description] ✓

## Key Outputs

### Files Created
- `/project-config/config.md` — Project configuration
- `/project-config/pages.md` — Page inventory with node IDs
- `/project-config/components.md` — Component inventory
- `/src/styles/design-tokens.css` — Extracted design tokens

### CMS Artifacts
- Organization ID: `{org_id}`
- Property ID: `{property_id}`
- Content Types Created:
  - `site-settings` (ID: xxx)
  - `navigation` (ID: xxx)
  - `footer` (ID: xxx)
  - [list all content types with IDs]

### Content Entries Created
- `global-site-settings` (slug)
- `global-navigation` (slug)
- [list key entries]

### Media Uploaded
- Total images: X
- CDN base URL: `https://...`
- Image manifest: `/project-config/media-manifest.json`

## Decisions Made
- [Key architectural decisions]
- [Content modeling choices]
- [Any deviations from standard patterns]

## Known Issues
- [Any unresolved issues to address in next group]

## Next Steps
- Begin Phase Group [B/C/D]
- First phase to run: Phase X
- Key files to read first: [list]
```

### Resuming After /clear

**When starting a new Phase Group after `/clear`:**

1. **Read the continuity file first:**
   ```
   Read /project-config/continuity-group-[a/b/c].md
   ```

2. **Read the project config:**
   ```
   Read /project-config/config.md
   ```

3. **Read phase-specific context:**
   - Group B: Read `/project-config/components.md`, `/src/styles/design-tokens.css`
   - Group C: Read component files, scan for issues
   - Group D: Read audit results

4. **Continue from the next phase**

---

## DYNAMIC BREAKPOINTS (Agent-Determined)

**After Phase 1 analysis, evaluate project complexity and insert additional breakpoints as needed.**

The static breakpoints (after Groups A, B, C) are the minimum. Based on Phase 1 outputs, Claude SHOULD insert additional breakpoints to improve output quality for complex projects.

### Complexity Assessment

After completing Phase 1, analyze the outputs and calculate complexity:

```markdown
## Complexity Factors (from Phase 1 outputs)

| Factor | Low | Medium | High |
|--------|-----|--------|------|
| Total Components | <15 | 15-30 | >30 |
| Pages | <5 | 5-10 | >10 |
| Nested Component Depth | 1 level | 2 levels | 3+ levels |
| Component Coverage | >90% | 70-90% | <70% |
| Unique Content Types Needed | <5 | 5-10 | >10 |

Complexity Score:
- 0-1 High factors = STANDARD (use static breakpoints only)
- 2-3 High factors = MODERATE (add 1 dynamic breakpoint)
- 4+ High factors = COMPLEX (add 2+ dynamic breakpoints)
```

### Dynamic Breakpoint Locations

**Phase Group B is most likely to benefit from splitting:**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  PHASE GROUP B: BUILD (Phases 7-9) — SPLIT OPTIONS                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Standard (no split):                                                       │
│  Phase 7 → Phase 8 → Phase 9 → /CLEAR                                       │
│                                                                             │
│  Split after Phase 7 (RECOMMENDED for >30 components):                      │
│  Phase 7 → /CLEAR → Phase 8 → Phase 9 → /CLEAR                              │
│  Benefit: Component library built with full context, then pages built fresh │
│                                                                             │
│  Split after Phase 8 (RECOMMENDED for >10 pages):                           │
│  Phase 7 → Phase 8 → /CLEAR → Phase 9 → /CLEAR                              │
│  Benefit: All content extracted, then fresh context for assembly            │
│                                                                             │
│  Double split (RECOMMENDED for >30 components AND >10 pages):               │
│  Phase 7 → /CLEAR → Phase 8 → /CLEAR → Phase 9 → /CLEAR                     │
│  Benefit: Maximum context for each major phase                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Decision Matrix

| Condition | Action |
|-----------|--------|
| >30 components defined in Figma | Add breakpoint after Phase 7 |
| >10 pages in site | Add breakpoint after Phase 8 |
| >50 content entries expected | Add breakpoint after Phase 6 (split Group A) |
| <70% component coverage (many exceptions) | Add breakpoint after Phase 7 |
| >3 levels of nested components | Add breakpoint after Phase 7 |

### Dynamic Breakpoint Implementation

**At the end of Phase 1, after writing all analysis files:**

```markdown
## DYNAMIC BREAKPOINT ASSESSMENT

Based on Phase 1 analysis:

**Complexity Factors:**
- Total Components: [X] → [Low/Medium/High]
- Pages: [X] → [Low/Medium/High]
- Nested Depth: [X levels] → [Low/Medium/High]
- Coverage: [X%] → [Low/Medium/High]
- Content Types Needed: [X] → [Low/Medium/High]

**Overall Complexity:** [STANDARD/MODERATE/COMPLEX]

**Dynamic Breakpoints Added:**
- [ ] After Phase 7 (Component Generation)
- [ ] After Phase 8 (Page Content Extraction)
- [ ] After Phase 6 (Global Content — splitting Group A)

**Updated Phase Group Structure:**
[Show the modified breakpoint schedule]
```

### Dynamic Continuity Files

When inserting a dynamic breakpoint, write a continuity file named:

```
/project-config/continuity-dynamic-phase-[N].md
```

Use the same format as static continuity files, but add:

```markdown
## Dynamic Breakpoint Reason
This breakpoint was inserted because:
- [Specific complexity factors that triggered this breakpoint]
- [What benefit this provides for the remaining phases]

## Next Phase
Continue with Phase [N+1]
```

### Notifying the User

When inserting a dynamic breakpoint, format the message as:

```
══════════════════════════════════════════════════════════════
⚡ DYNAMIC BREAKPOINT: Phase [N] Complete

This project has [MODERATE/COMPLEX] complexity:
- [X] components (threshold: 30)
- [X] pages (threshold: 10)
- [X%] component coverage (threshold: 70%)

Adding breakpoint here to maintain output quality.

Continuity saved to: /project-config/continuity-dynamic-phase-[N].md

▶ BREAKPOINT: Please run /clear to free context

After clearing, say: "Continue with Phase [N+1]"
══════════════════════════════════════════════════════════════
```

---

## CRITICAL RULES (Read These First)

### ZERO HARDCODED CONTENT RULE

**THIS IS THE MOST IMPORTANT RULE OF THE ENTIRE BUILD.**

Every single piece of text, every URL, every label — EVERYTHING must come from the CMS.
There are NO exceptions. This includes:

- Navigation menu items, footer links, legal/copyright text
- Button labels ("Reserve a Table", "Submit", "Learn More")
- Form placeholders, section labels, contact information
- Logo URLs and alt text, social media handles
- Error messages, loading states, 404 page content
- Metadata (page titles, descriptions)

**If you find yourself typing a string that will be displayed to users, STOP.**
That string must come from CMS content passed via props.

**The ONLY hardcoded strings allowed are:**
- CSS class names, HTML attribute names, JavaScript variable names
- Technical identifiers (IDs, keys), Console.log messages

### INLINE EDITOR DATA ATTRIBUTES

**EVERY editable element MUST include CMS data attributes for inline editing.**

| Attribute | Required | Description |
|-----------|----------|-------------|
| `data-cms-entry` | Yes | Entry slug (e.g., `"homepage"`, `"global-navigation"`) |
| `data-cms-field` | Yes | Dot-notation path (e.g., `"hero.title"`, `"menuLinks[0].label"`) |
| `data-cms-type` | Optional | Override type: `text`, `richtext`, `image`, `array`, etc. |

```tsx
// Example usage
<h1 data-cms-entry="homepage" data-cms-field="hero.title">{hero.title}</h1>
<img data-cms-entry="homepage" data-cms-field="hero.image" data-cms-type="image" src={src} />
```

### CMS DATA STRUCTURE RULE

**CRITICAL: Use FLAT data structures in CMS content entries.**

All content sections must be at the TOP LEVEL of the `data` object. Never nest content inside an extra `data` property.

✅ **Correct (flat):**
```typescript
createContentEntry({ data: {
  hero: { title: "...", imageUrl: "..." },
  about: { heading: "...", body: "..." },
  // Sections are direct children of data
}})
```

❌ **Wrong (nested):**
```typescript
createContentEntry({ data: {
  data: {  // <-- WRONG! Extra nesting breaks inline editing
    hero: { title: "...", imageUrl: "..." },
  }
}})
```

### CMS FIELD DEFINITION RULES

**CRITICAL: Create content types with proper field definitions BEFORE creating content entries.**

1. **Field slugs must be FLAT** (no dot notation):
   ```typescript
   // ✅ CORRECT
   { slug: 'heroTitle', name: 'Hero Title', type: 'text' }
   { slug: 'heroVideoUrl', name: 'Hero Video', type: 'media' }

   // ❌ WRONG
   { slug: 'hero.title', name: 'Hero Title', type: 'text' }
   ```

2. **Use section prefixes** in camelCase: `{sectionName}{FieldName}`
   - `heroTitle`, `heroSubtitle`, `heroVideoUrl`
   - `aboutHeading`, `aboutBody`, `aboutCtaText`

3. **Select appropriate field types**:
   | Type | Use For |
   |------|---------|
   | `text` | Titles, labels, URLs |
   | `richText` | Formatted paragraphs |
   | `media` | Images, videos, files |
   | `json` | Arrays, complex objects |
   | `reference` | Links to other entries |

4. **Arrays**: Use JSON fields for simple arrays, or create separate content types with references for better admin UX

See [CMS-FIELD-DEFINITIONS.md](CMS-FIELD-DEFINITIONS.md) for detailed rules and patterns.

### ANTI-HALLUCINATION RULES

1. **NEVER INVENT CONTENT** — Every text string must be extracted verbatim from Figma
2. **NEVER ASSUME STRUCTURE** — Verify exact layout in Figma before building
3. **NEVER SKIP SECTIONS** — If Figma has 8 sections, build 8 sections
4. **NEVER APPROXIMATE VALUES** — Extract exact colors, spacing, font sizes from Figma
5. **NEVER SUBSTITUTE ASSETS** — Every image must come from Figma export
6. **NEVER USE PARALLEL AGENTS** — Build pages sequentially
7. **EVERY PAGE GETS HOMEPAGE TREATMENT** — Same rigor for page 5 as page 1
8. **NEVER NEST DATA IN DATA** — Content sections go at top level of `data` object

### CONTENT STRUCTURE GUIDELINES

**CRITICAL: Follow these rules for naming and granularity to avoid admin confusion.**

#### Naming Conventions (Avoid Domain Collisions)

Use **prefixes** for UI/navigation concepts to avoid collision with domain-specific terms:

| Concept | Prefix | Examples |
|---------|--------|----------|
| Navigation | `nav-` | `nav-links`, `nav-menu`, `nav-item` |
| Header | `header-` | `header-cta`, `header-logo` |
| Footer | `footer-` | `footer-links`, `footer-newsletter` |
| Site-wide | `site-` | `site-settings`, `site-logo` |

**Domain terms use NO prefix:**
- Restaurant: `menu-category`, `menu-item`, `dish`
- Hotel: `room-type`, `amenity`, `rate`
- Retail: `product`, `collection`, `category`

**Entry slugs must be unique and descriptive:**
```
✅ GOOD: wynwood-hours, run-house-hours, wynwood-contact
❌ BAD: hours, hours-1, contact (ambiguous in admin)
```

For multi-property sites, prefix with property: `{property}-{concept}`

#### Granularity Rules (When to Create Separate Entries)

**CREATE SEPARATE ENTRIES when:**
- Content is **reused** across 2+ pages
- Content has **3+ fields** that form a logical unit
- Content needs **independent editing** lifecycle
- Content appears in **lists** that users will manage (add/remove)

**KEEP AS EMBEDDED FIELDS when:**
- Fields **always change together** (address = street + city + state + zip)
- Data is **simple** (single value or 2-3 related values)
- Content is **only used in one place**
- Separate entry would create **admin clutter**

| Pattern | Recommendation | Example |
|---------|----------------|---------|
| Address block | Single `address` field (JSON) | `{street, city, state, zip}` |
| Contact info | Single `contact` field (JSON) | `{phone, email, address}` |
| Business hours | Single `hours` field (JSON array) | `[{days, open, close}]` |
| Social links | Single `socialLinks` field (JSON array) | `[{platform, url}]` |
| Team members | **Separate entries** (team-member CT) | Reusable, has bio/photo |
| Menu items | **Separate entries** (menu-item CT) | Managed list |
| FAQ items | **Separate entries** (faq-item CT) | Shared across pages |
| Page sections | **Separate entries** (content-section CT) | Reusable components |

**The granularity test:** Would a content editor want to manage this item independently in the admin? If yes, make it a separate entry.

---

## CONTENT ARCHITECTURE

**Content Flow:**
```
Figma Design → Extract Content → Upload Images to CDN → Create CMS Entries → Website Fetches via GraphQL
```

**Technology Stack:**
- Next.js 14+ with App Router, TypeScript, Tailwind CSS
- Apollo Client (server-side only) for GraphQL
- CDN (Cloudflare R2, AWS S3, or similar) for images
- React Server Components with `force-dynamic`
- Next.js Middleware for edit mode cache bypass (see PHASE-2-INIT.md)

### THREE-TIER CONTENT MODEL

**CRITICAL:** Content Types map to Figma components, NOT pages.

```
┌─────────────────────────────────────────────────────────────────┐
│                     TIER 1: GLOBAL (3 CTs)                      │
│  site-settings  │  navigation  │  footer                        │
│  (1 entry each - site-wide configuration)                       │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                   TIER 2: COMPONENTS (varies)                   │
│  hero-section  │  content-section  │  gallery  │  faq-item      │
│  (N entries each - reusable building blocks from Figma)         │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                     TIER 3: PAGES (1 CT)                        │
│  page (with template field)                                     │
│  (N entries - rendered dynamically via catch-all route)         │
└─────────────────────────────────────────────────────────────────┘
```

**Why Component-Based?**
| Approach | Problem |
|----------|---------|
| One CT per page | Duplicates fields, can't share content between pages |
| One CT for everything | Too generic, hard to validate |
| **Component-based** | Reusable, validates well, maps to Figma design |

**Key Benefits:**
- Hero sections reusable across pages
- FAQs shared between About, Contact, FAQ pages
- Galleries are independent, referenced by pages
- **New pages work immediately** — create in CMS, accessible at URL
- No code changes needed for new content pages

See [PHASE-5-CONTENT-TYPES.md](PHASE-5-CONTENT-TYPES.md) for complete implementation guide.

### COMPONENT-DRIVEN BUILD ARCHITECTURE

**CRITICAL:** Build a global component library FIRST, then reuse across all pages.

#### Phase 1: Component Analysis

The skill analyzes the Figma file structure (Wireframes, Components, Design System, Assets pages) and generates:

1. **`/project-config/components.md`** — All components with node IDs and nested relationships
2. **`/project-config/component-coverage.md`** — Coverage analysis comparing instances used vs defined

This analysis determines:
- Which components to build
- Build order based on dependencies
- Missing components (exceptions)
- Nested component relationships

#### Component Build Order (Atomic Design)

```
Level 1: ATOMIC (no dependencies)
├── Button, Link, Input, Dropdown, Radio, Checkbox
│
Level 2: MOLECULES (use Level 1)
├── MenuLink, FaqAccordion, EventCard, AwardCard
│
Level 3: ORGANISMS (use Level 1 & 2)
├── Navigation, Footer, HeroSection, Gallery, FaqSection
│
Level 4: TEMPLATES (compose Level 3)
└── StandardTemplate, GalleryTemplate, FormTemplate, ListingTemplate
```

#### Directory Structure

```
src/components/
├── ui/           # Level 1: Atomic
├── molecules/    # Level 2: Molecules
├── organisms/    # Level 3: Organisms
├── templates/    # Level 4: Templates
├── exceptions/   # Components not in Figma library
└── layout/       # App shell
```

#### Nested Component Handling

When a Figma component contains other components (e.g., Navigation → MenuLink → Button):

| Scenario | Approach |
|----------|----------|
| Child reused across parents | **Composition** (separate entries, references) |
| Child only used in parent | **Embedding** (JSON field in parent) |
| Child has 3+ fields | **Composition** (better admin UX) |
| Child is simple (1-2 fields) | **Embedding** (avoid clutter) |

See [PHASE-5-CONTENT-TYPES.md](PHASE-5-CONTENT-TYPES.md) for detailed nested component patterns.
See [PHASE-7-COMPONENTS.md](PHASE-7-COMPONENTS.md) for component build instructions.

### DYNAMIC ROUTING ARCHITECTURE

**CRITICAL:** All content pages use a dynamic catch-all route. This enables creating new pages from the CMS without code changes.

```
src/app/
├── page.tsx              # Homepage only (/ can't use catch-all)
├── menu/page.tsx         # Application pages (complex logic, optional)
└── [...slug]/page.tsx    # ALL content pages (dynamic)
```

**How it works:**
1. User navigates to `/about`
2. Catch-all route fetches `page` entry with `slug: 'about'`
3. Template component renders based on `page.template` field
4. If no entry found, returns 404

**Templates:** Pages specify a `template` field (e.g., `standard`, `gallery`, `form`, `listing`) that determines which React component renders the page.

See [PHASE-9-PAGES.md](PHASE-9-PAGES.md) for implementation details.

### CONTENT LAYER REQUIREMENT

**CRITICAL:** Every website MUST have a content layer (`src/lib/content/index.ts`) that transforms CMS data before passing to components. This layer MUST include `transformImageUrls()` to handle inline editor image saves.

**Why:** The inline editor saves images as objects `{id, url, src, alt, filename}`, but components expect plain URL strings. Without transformation, images display in the editor but NOT on the live site after refresh.

```typescript
// src/lib/content/index.ts - REQUIRED PATTERN
function isImageObject(obj: unknown): obj is { url?: string; src?: string } {
  if (typeof obj !== 'object' || obj === null) return false;
  const o = obj as Record<string, unknown>;
  return (typeof o.url === 'string' || typeof o.src === 'string') &&
    (o.id !== undefined || o.filename !== undefined || o.alt !== undefined);
}

function transformImageUrls<T>(data: T): T {
  if (data === null || data === undefined) return data;
  if (typeof data === 'string') return data;
  if (Array.isArray(data)) return data.map(transformImageUrls) as T;
  if (typeof data === 'object') {
    if (isImageObject(data)) {
      return ((data as any).url || (data as any).src || '') as T;
    }
    const result: Record<string, unknown> = {};
    for (const [key, value] of Object.entries(data as Record<string, unknown>)) {
      result[key] = transformImageUrls(value);
    }
    return result as T;
  }
  return data;
}

export async function getPageContent<T>(slug: string): Promise<T> {
  const { data } = await client.query(...);
  return transformImageUrls(data?.contentEntryBySlug?.data) as T;  // ALWAYS transform!
}
```

See LEARNINGS.md for the complete implementation pattern.

### SITE FRAMEWORK ARCHITECTURE

**CRITICAL:** Every site built with f2c follows a hybrid routing model — catch-all by default, with dedicated routes as escape hatches. This section defines the SEO, security, and performance standards that the generated code must meet.

#### Hybrid Routing Model

```
app/
├── [...slug]/
│   └── page.tsx          ← CMS-driven pages (95% of site)
├── reservations/
│   └── page.tsx          ← Dedicated route (booking widget, special CSP)
├── api/
│   └── revalidate/
│       └── route.ts      ← Webhook for ISR revalidation
├── sitemap.ts            ← Queries CMS for all published slugs
├── robots.ts             ← CMS-controlled robots rules
└── layout.tsx            ← Root layout with nav/footer from CMS
```

**When to use a dedicated route instead of catch-all:**
- Page needs third-party widgets (booking, payments) with unique CSP
- Page has complex client-side state or auth-gated content
- Page requires a caching strategy different from the rest of the site

#### Layer Responsibilities

**f2c Skill (This Skill — First Filter)**

Generates the catch-all skeleton, section registry, content-fetching layer with slug validation and output sanitization, and wires up `@halo/renderer-toolkit` for SEO/GEO. Every site starts with this.

**Halo Backend (Data Contract)**

Provides `PageSeoConfig` entities per page (metaTitle, metaDescription, ogImage, canonicalUrl, noIndex), `BusinessProfile` entity for structured data, `SiteSeoConfig` for site-level SEO/GEO settings, `template`/`layout` field for section mapping, `sitemap` boolean, navigation content type, and slug uniqueness enforcement per property.

**Renderer Toolkit (`@halo/renderer-toolkit` — Shared Package)**

Provides `buildMetadata()`, `buildSitemap()`, `buildRobots()`, `serveLlmsTxt()`, `buildJsonLd()`, and `<JsonLd>` component. All Next.js sites import these instead of hand-rolling SEO.

**Per-Site (Customization)**

Adds custom section components to the template registry, design tokens/theme, and dedicated routes for pages that need special treatment.

#### SEO / GEO Requirements

Every generated site MUST include:

1. **`buildMetadata(slug)` from renderer toolkit** — Fetches `PageSeoConfig` per slug, returns Next.js `Metadata` object. SEO data lives in `PageSeoConfig` entities, NOT on content entry fields. Editors control SEO without deploys.
2. **`buildSitemap()` from renderer toolkit** — Queries CMS for all published slugs. New pages appear automatically. See Phase 9 for implementation.
3. **`buildRobots()` from renderer toolkit** — CMS-controlled robots rules with AI crawler directives (GPTBot, ClaudeBot, PerplexityBot, Google-Extended). See Phase 9.
4. **`serveLlmsTxt()` from renderer toolkit** — Generates `llms.txt` for AI crawlers describing site structure and content.
5. **`<JsonLd>` + `buildJsonLd()` from renderer toolkit** — Structured data driven by `BusinessProfile` and content type. Schema.org mapping (e.g., `LocalBusiness`, `Restaurant`, `Hotel`, `Article`, `FAQPage`, `BreadcrumbList`).
6. **Canonical URLs** — CMS-controlled per `PageSeoConfig` entry. Prevents duplicate content across properties.
7. **GEO readiness** — Semantic HTML, structured data, llms.txt, and CMS-driven content blocks make content extractable by AI crawlers.

#### Security Requirements

Every generated site MUST include:

1. **Slug validation** — Regex `/^[a-z0-9]+(?:[-/][a-z0-9]+)*$/` in the catch-all route. Rejects path traversal (`../../admin`) and injection (`<script>`).
2. **CMS output sanitization** — Sanitize HTML fields at fetch time using DOMPurify or equivalent. Applied in the content layer (`src/lib/content/index.ts`).
3. **Content-type allowlist** — Only render known content types. Prevents arbitrary CMS types from rendering if an editor creates them.
4. **Published-only filter** — Content-fetching layer only returns published entries. Draft content never leaks to the public site.
5. **No credentials in client code** — CMS org ID may be public, but auth tokens, API keys, and passwords must never appear in client bundles.

#### Performance Requirements

Every generated site MUST include:

1. **Dynamic imports for templates** — Section registry uses `next/dynamic` to avoid loading all templates in one bundle.
2. **ISR with on-demand revalidation** — `api/revalidate` webhook endpoint. CMS triggers revalidation on content publish. Falls back to `force-dynamic` during development.
3. **Image optimization** — All images use `next/image`. CDN URLs in `next.config.js` `images.remotePatterns`.
4. **Font preloading** — Design tokens extracted in Phase 3 include font loading strategy.
5. **Edge caching headers** — `Cache-Control` headers set per route. Catch-all pages cache for configurable duration with `stale-while-revalidate`.

---

## EXECUTION PHASES

Execute phases in order within each Phase Group. After completing a Phase Group:
1. Write the continuity file
2. Commit to git
3. Notify user of `/clear` breakpoint
4. Wait for user to clear context before proceeding

---

### PHASE GROUP A: FOUNDATION (Phases 1-6)

**Context requirement:** LOW — all outputs persisted to files and CMS

| Phase | Name | Reference File |
|-------|------|----------------|
| 1 | Pre-flight Validation & Authentication | [PHASE-1-PREFLIGHT.md](PHASE-1-PREFLIGHT.md) |
| 2 | Project Initialization & CMS Setup | [PHASE-2-INIT.md](PHASE-2-INIT.md) |
| 3 | Design Token Extraction | [PHASE-3-TOKENS.md](PHASE-3-TOKENS.md) |
| 4 | Asset Extraction & CDN Upload | [PHASE-4-ASSETS.md](PHASE-4-ASSETS.md) |
| 5 | CMS Content Type Setup | [PHASE-5-CONTENT-TYPES.md](PHASE-5-CONTENT-TYPES.md) |
| 5.5 | Media Extraction & R2 Upload | [PHASE-5.5-MEDIA-EXTRACTION.md](PHASE-5.5-MEDIA-EXTRACTION.md) |
| 6 | Global Content Extraction | [PHASE-6-GLOBAL-CONTENT.md](PHASE-6-GLOBAL-CONTENT.md) |

**After Phase 6, BEFORE proceeding:**

```markdown
## BREAKPOINT 1 INSTRUCTIONS

1. Write continuity file:
   - Create `/project-config/continuity-group-a.md`
   - Include all CMS IDs, content type slugs, entry slugs
   - List all files created
   - Document any decisions or deviations

2. Git commit:
   ```bash
   git add -A
   git commit -m "Phase Group A complete: Foundation

   - Project initialized with Next.js + TypeScript + Tailwind
   - Design tokens extracted from Figma
   - CMS content types created
   - Global content entries populated
   - Media assets uploaded to CDN

   Co-Authored-By: Claude <noreply@anthropic.com>"
   ```

3. Notify user:
   ```
   ══════════════════════════════════════════════════════════════
   ✓ PHASE GROUP A COMPLETE: FOUNDATION

   Completed:
   - Phases 1-6 (Preflight through Global Content)
   - All CMS schema and global content created
   - Design tokens and assets ready

   Continuity saved to: /project-config/continuity-group-a.md

   ▶ BREAKPOINT: Please run /clear to free context before Phase Group B

   After clearing, say: "Continue with Phase Group B (Build)"
   ══════════════════════════════════════════════════════════════
   ```

4. STOP and wait for user to run /clear
```

---

### PHASE GROUP B: BUILD (Phases 7-9)

**Context requirement:** HIGH — component library must stay in context during page assembly

**Before starting (after /clear):**
1. Read `/project-config/continuity-group-a.md`
2. Read `/project-config/config.md`
3. Read `/project-config/components.md`
4. Read `/src/styles/design-tokens.css`

| Phase | Name | Reference File |
|-------|------|----------------|
| 7 | Component Generation | [PHASE-7-COMPONENTS.md](PHASE-7-COMPONENTS.md) |
| 8 | Page Content Extraction | [PHASE-8-PAGE-CONTENT.md](PHASE-8-PAGE-CONTENT.md) |
| 9 | Page Assembly | [PHASE-9-PAGES.md](PHASE-9-PAGES.md) |

**After Phase 9, BEFORE proceeding:**

```markdown
## BREAKPOINT 2 INSTRUCTIONS

1. Write continuity file:
   - Create `/project-config/continuity-group-b.md`
   - List all components created with file paths
   - List all pages created with their templates
   - Document page-to-component mapping
   - Note any complex patterns used

2. Git commit:
   ```bash
   git add -A
   git commit -m "Phase Group B complete: Build

   - All React components generated
   - All page content extracted to CMS
   - Dynamic routing implemented
   - Site renders all pages from CMS

   Co-Authored-By: Claude <noreply@anthropic.com>"
   ```

3. Notify user:
   ```
   ══════════════════════════════════════════════════════════════
   ✓ PHASE GROUP B COMPLETE: BUILD

   Completed:
   - Phases 7-9 (Components through Page Assembly)
   - All components built with CMS data attributes
   - All pages rendering from CMS content

   Continuity saved to: /project-config/continuity-group-b.md

   ▶ BREAKPOINT: Please run /clear to free context before Phase Group C

   After clearing, say: "Continue with Phase Group C (Audit)"
   ══════════════════════════════════════════════════════════════
   ```

4. STOP and wait for user to run /clear
```

---

### PHASE GROUP C: AUDIT (Phases 10-12)

**Context requirement:** LOW — reads existing code, fixes issues

**Before starting (after /clear):**
1. Read `/project-config/continuity-group-b.md`
2. Scan component and page files for audit

| Phase | Name | Reference File |
|-------|------|----------------|
| 10 | Zero Hardcoded Content Audit | [PHASE-10-AUDIT.md](PHASE-10-AUDIT.md) |
| 11 | Inline Editor Audit | [PHASE-11-EDITOR-AUDIT.md](PHASE-11-EDITOR-AUDIT.md) |
| 12 | Automated QA | [PHASE-12-QA.md](PHASE-12-QA.md) |

**After Phase 12, BEFORE proceeding:**

```markdown
## BREAKPOINT 3 INSTRUCTIONS

1. Write continuity file:
   - Create `/project-config/continuity-group-c.md`
   - List all issues found and fixed
   - List any remaining issues (with severity)
   - Document test results

2. Git commit:
   ```bash
   git add -A
   git commit -m "Phase Group C complete: Audit

   - Zero hardcoded content verified
   - All inline editor attributes validated
   - Automated QA passed
   - [X] issues found and fixed

   Co-Authored-By: Claude <noreply@anthropic.com>"
   ```

3. Notify user:
   ```
   ══════════════════════════════════════════════════════════════
   ✓ PHASE GROUP C COMPLETE: AUDIT

   Completed:
   - Phases 10-12 (Content Audit through QA)
   - Zero hardcoded content verified
   - Inline editor fully functional
   - All automated tests passing

   Continuity saved to: /project-config/continuity-group-c.md

   ▶ BREAKPOINT: Please run /clear to free context before Phase Group D

   After clearing, say: "Continue with Phase Group D (Deploy)"
   ══════════════════════════════════════════════════════════════
   ```

4. STOP and wait for user to run /clear
```

---

### PHASE GROUP D: DEPLOY (Phases 13-14)

**Context requirement:** LOW — final checks and deployment

**Before starting (after /clear):**
1. Read `/project-config/continuity-group-c.md`
2. Read deployment configuration

| Phase | Name | Reference File |
|-------|------|----------------|
| 13 | Production Audit | [PHASE-13-PRODUCTION.md](PHASE-13-PRODUCTION.md) |
| 14 | Deployment | [PHASE-14-DEPLOY.md](PHASE-14-DEPLOY.md) |

**After Phase 14:**

```markdown
## BUILD COMPLETE

1. Final commit:
   ```bash
   git add -A
   git commit -m "Phase Group D complete: Deployed to production

   - Production audit passed
   - Deployed to [platform]
   - Live URL: [url]

   Co-Authored-By: Claude <noreply@anthropic.com>"
   ```

2. Notify user:
   ```
   ══════════════════════════════════════════════════════════════
   ✓ BUILD COMPLETE

   🎉 Site deployed successfully!

   Production URL: [url]
   Admin URL: [admin-url]

   All 14 phases completed:
   ✓ Phase Group A: Foundation (1-6)
   ✓ Phase Group B: Build (7-9)
   ✓ Phase Group C: Audit (10-12)
   ✓ Phase Group D: Deploy (13-14)

   Continuity files preserved in /project-config/ for future reference.
   ══════════════════════════════════════════════════════════════
   ```
```

---

### Reference Documents

| Document | Purpose |
|----------|---------|
| [CMS-FIELD-DEFINITIONS.md](CMS-FIELD-DEFINITIONS.md) | Field types, validation, JSON schemas |
| [CMS-REFERENCE.md](CMS-REFERENCE.md) | CMS API overview |
| [CMS-FALLBACK.md](CMS-FALLBACK.md) | Resilience: serve cached content if CMS down |
| [JSON-FIELD-SCHEMA.md](JSON-FIELD-SCHEMA.md) | JSON Schema patterns for structured editing |
| [GRAPHQL-EXAMPLES.md](GRAPHQL-EXAMPLES.md) | GraphQL query/mutation examples |
| [COMPONENT-PATTERNS.md](COMPONENT-PATTERNS.md) | React component patterns |
| [NESTED-COMPONENTS.md](NESTED-COMPONENTS.md) | Nested component handling (composition vs embedding) |
| [LEARNINGS.md](LEARNINGS.md) | Lessons learned, common issues |

---

## PROJECT CONFIGURATION

### Configuration Input & Storage

**CRITICAL: Save the user's configuration to a file for project reference.**

When the user invokes `/figma-to-code` and pastes their configuration:

1. **Create the project-config directory** (if it doesn't exist):
   ```bash
   mkdir -p project-config
   ```

2. **Save the configuration to `/project-config/config.md`**:
   - Write the user's pasted configuration verbatim to this file
   - This becomes the project's reference document for all phases
   - Commit this file to version control

3. **Confirm the save to the user**:
   ```
   ✓ Configuration saved to /project-config/config.md

   This file will be used as reference throughout the build.
   You can edit it to update configuration values.
   ```

**Why save the config?**
- Provides a reference document checked into version control
- Allows re-running phases without re-pasting configuration
- Documents project settings for other developers
- Enables easy updates to configuration values

### Reading Configuration

All phases read configuration from `/project-config/config.md`.

**Expected format (simplified):**

```markdown
# Project Configuration

## Project Info
**Project Name:** My Project Name
**Brand Name:** My Brand
**Project Slug:** my-project-name

## Figma
**Main File URL:** https://www.figma.com/design/xxx/Project-Name

## CMS Configuration
**CMS Type:** Halo CMS
**CMS GraphQL URL:** https://backend-production-162b.up.railway.app/graphql
**CMS Organization ID:** org_xxx
**CMS Organization Slug:** my-organization
**CMS Property ID:** prop_xxx (or "CREATE_NEW" to create during preflight)
**CMS Property Slug:** my-property-slug (required if CREATE_NEW)
**CMS Property Name:** My Property Name (required if CREATE_NEW)
**CMS Admin Email:** admin@example.com
**CMS WebsiteProject ID:** wp_xxx (or "CREATE_NEW" to create during preflight)

## Deployment
**Deployment Platform:** Vercel
```

### Auto-Discovery (No Manual Entry Required)

The following are **automatically discovered** from the Figma file during Phase 1:
- **Pages** — All website pages with desktop/mobile frame URLs
- **Global Components** — Navigation, Footer, Menu Overlay frames
- **Page Slugs** — Inferred from frame names (editable in `/project-config/pages.md`)
- **Component Library** — All reusable components from the Components/Design System page

See [PHASE-1-PREFLIGHT.md](PHASE-1-PREFLIGHT.md) for the discovery process.

**Outputs:**
- `/project-config/figma-structure.md` — 4-page Figma structure detection
- `/project-config/pages.md` — Page inventory (review and edit if needed)
- `/project-config/components.md` — Component library inventory with nested relationships
- `/project-config/component-coverage.md` — Coverage analysis and build order
- `/project-config/dynamic-breakpoints.md` — Dynamic breakpoint assessment based on complexity

### Sensitive Credentials

These are **NOT** stored in config.md:

| Credential | Storage |
|------------|---------|
| CMS Admin Password | Prompted at runtime |
| Figma Access Token | Figma MCP (OAuth) |

### Required `.env.local` File

Create `.env.local` in your project root before starting:

```bash
# CMS Configuration
NEXT_PUBLIC_CMS_GRAPHQL_URL=https://backend-production-162b.up.railway.app/graphql
CMS_ORGANIZATION_ID=org_xxx
CMS_PROPERTY_ID=prop_xxx
WEBSITE_PROJECT_ID=wp_xxx
```

**Note:** Media uploads go through the CMS API (`uploadMedia` mutation). The CMS handles CDN storage internally — no direct CDN credentials needed.

### Property Requirement

**CRITICAL:** Every website MUST be associated with a Property in the CMS.

- **Organization** = Parent entity (e.g., "Acme Hotels")
- **Property** = Individual website/brand (e.g., "NoMad Wynwood", "Run House")

Content entries are scoped to a Property. This allows:
- Multiple websites under one Organization
- Property-specific content that doesn't leak between sites
- Proper content isolation in the admin interface

If the Property doesn't exist, Phase 1 will create it using the configuration values.

---

## QUICK REFERENCE

### Data Attribute Patterns

```tsx
// Text
<h1 data-cms-entry={entry} data-cms-field="hero.title">{title}</h1>

// Image
<img data-cms-entry={entry} data-cms-field="hero.image" data-cms-type="image" src={src} />

// Array item
<p data-cms-entry={entry} data-cms-field="team[0].name">{name}</p>

// Rich text
<div data-cms-entry={entry} data-cms-field="intro.body" data-cms-type="richtext" />

// Global content
<span data-cms-entry="global-footer" data-cms-field="copyrightText">{copyright}</span>
```

### Checkpoint Command

```bash
osascript -e 'display notification "Checkpoint: {{PHASE_NAME}} complete" with title "Claude Code" sound name "Ping"'
```

---

## REFERENCE FILES

- [COMPONENT-PATTERNS.md](COMPONENT-PATTERNS.md) - Component code examples
- [CMS-REFERENCE.md](CMS-REFERENCE.md) - Content type and entry slug reference
- [CMS-FIELD-DEFINITIONS.md](CMS-FIELD-DEFINITIONS.md) - Field definition rules and patterns
- [GRAPHQL-EXAMPLES.md](GRAPHQL-EXAMPLES.md) - GraphQL query/mutation examples

### CMS API Reference

For all CMS GraphQL operations (authentication, content types, entries, media uploads), use the `Halo-API` skill. This skill is listed as a dependency and provides complete query/mutation documentation.

**Key operations from Halo-API:**
- `login` — Authenticate and get JWT token
- `createContentType` — Create content type schemas
- `createContentEntry` — Create content entries
- `uploadMedia` — Upload images (CMS handles CDN internally)
- `contentEntryBySlug` — Fetch content for rendering

---

## Common Pitfalls to Avoid

❌ **Don't:**
- Hardcode ANY text that displays to users (buttons, labels, titles, etc.)
- Invent content that doesn't exist in the Figma design
- Skip the data-cms attributes on editable elements
- Approximate colors or spacing — extract exact values from Figma
- Use parallel agents for page builds (causes context issues)
- Forget to upload images to CDN before referencing them in CMS
- Create components without TypeScript interfaces for CMS data
- Skip pages because they "look similar" to others
- Create nested `data.data` structures in CMS entries
- Deploy to production with temporary Figma MCP image URLs
- Skip the `transformImageUrls()` function in the content layer — inline editor images won't display
- Assume images from CMS are always strings — inline editor saves them as objects
- Forget to create `middleware.ts` — inline editor saves won't reflect after page refresh
- **Continue past a breakpoint without writing continuity file**
- **Skip reading continuity file after /clear**

✅ **Do:**
- Extract EVERY piece of text verbatim from Figma
- Add `data-cms-entry`, `data-cms-field` to all editable elements
- Use exact hex values, font sizes, and spacing from Figma
- Build pages sequentially with full attention to each
- Upload images to CDN first, then reference URLs in CMS entries
- Create typed interfaces that match your CMS content structure
- Treat the 5th page with the same rigor as the 1st page
- Commit and checkpoint after each phase
- Keep CMS data structure flat (sections at top level of `data`)
- Run image migration script before production deployment
- **ALWAYS include `transformImageUrls()` in your content layer** — this handles inline editor image saves
- **ALWAYS create `middleware.ts`** — this enables edit mode cache bypass so changes reflect after save
- Test inline editing by saving an image and refreshing the page to verify it persists
- **Write continuity file at each breakpoint with ALL context needed to resume**
- **Read continuity file first thing after /clear**

---

## Version History

**v6.0.0** (February 2026)
- **MAJOR: DB-Driven Composition & SEO Module Reconciliation** — Reconciled F2C skill with new Halo architecture
- Phase 1: WebsiteProject awareness — validate/create WebsiteProject during preflight
- Phase 2: Renderer toolkit wiring — `@halo/renderer-toolkit` installed, env vars updated
- Phase 3: DesignTokenSet sync — F2C syncs Figma-extracted tokens to DB `DesignTokenSet` entity
- Phase 5: Content type upsert — `createOrUpdateContentType()` pattern for wizard coexistence; SEO fields removed from `page` content type (now in `PageSeoConfig`)
- Phase 6: Global content upsert — `upsertGlobalEntry()` handles wizard-created entries
- Phase 7: DB composition — `ComponentDefinition` + `ComponentVariant` registration in DB; component registry
- Phase 8: SEO module integration — `savePageSeo` / `bulkSavePageSeo` instead of content entry fields
- Phase 9: DB page composition — `PageTemplate` + `PageComposition` entities; universal `PageRenderer`; `buildMetadata()`, `buildSitemap()`, `buildRobots()`, `serveLlmsTxt()`, `<JsonLd>` from toolkit
- Phase 13: Production audit — renderer toolkit verification, SEO module checks, llms.txt, JSON-LD, GEO readiness
- Phase 14: Build pipeline integration — `BuildTask` progress reporting, `WebsiteProject` update after deployment
- Updated Site Framework Architecture to reference renderer toolkit and `PageSeoConfig` model

**v5.0.0** (January 2026)
- **MAJOR: Dynamic Breakpoints** — Claude Code now assesses project complexity after Phase 1 and inserts additional `/clear` breakpoints as needed
- New Step 7 in Phase 1: Dynamic Breakpoint Assessment
- Complexity factors: component count, page count, nesting depth, coverage percentage, content type count
- Threshold-based decision matrix for inserting breakpoints
- Phase Group B most likely to benefit from splitting (after Phase 7 and/or Phase 8)
- Dynamic continuity files: `/project-config/continuity-dynamic-phase-[N].md`
- New output: `/project-config/dynamic-breakpoints.md` — documents recommended breakpoints
- User notification format for dynamic breakpoints with complexity reasoning
- Updated PHASE-1-PREFLIGHT.md with Step 7 (Dynamic Breakpoint Assessment)

**v4.0.0** (January 2026)
- **MAJOR: Component-Driven Build Architecture** — Build global component library first, reuse everywhere
- Phase 1 now includes Figma structure detection (4-page pattern: Wireframes, Components, Design System, Assets)
- Phase 1 now includes Component Coverage Analysis (comparing instances used vs components defined)
- New outputs: `/project-config/figma-structure.md`, `/project-config/component-coverage.md`
- Enhanced `/project-config/components.md` with nested component relationships
- Component build order follows Atomic Design: Atomic → Molecules → Organisms → Templates
- Exception handling for components used in Wireframes but not defined in Component Library
- Phase 5 updated with Nested Component Content Modeling (Composition vs Embedding decision tree)
- Phase 7 completely rewritten with 4-level component build strategy
- New directory structure: `ui/`, `molecules/`, `organisms/`, `templates/`, `exceptions/`
- Added COMPONENT-DRIVEN BUILD ARCHITECTURE section to main docs

**v3.0.0** (January 2026)
- **MAJOR: Phase Groups with /clear Breakpoints** — Reorganized 14 phases into 4 Phase Groups
- Added continuity file system for preserving context across /clear boundaries
- Group A (Foundation): Phases 1-6 — setup, tokens, CMS schema
- Group B (Build): Phases 7-9 — components, content, pages (HIGH context dependency)
- Group C (Audit): Phases 10-12 — verification and fixes
- Group D (Deploy): Phases 13-14 — production and deployment
- Added explicit breakpoint instructions with continuity file format
- Added resume instructions for reading continuity after /clear
- Improved output quality by managing context window effectively

**v2.0.0** (January 2026)
- **MAJOR: Site Framework Architecture** — Added comprehensive hybrid routing model
- SEO/GEO requirements: `generateMetadata()`, `sitemap.ts`, `robots.ts`, structured data (JSON-LD), canonical URLs, GEO readiness for AI crawlers
- Security requirements: slug validation regex, CMS output sanitization, content-type allowlist, published-only filter
- Performance requirements: dynamic imports for templates, ISR with on-demand revalidation, edge caching headers
- Layer responsibilities: f2c (first filter) → Halo backend (data contract) → Per-site (customization)
- Updated PHASE-9-PAGES.md with sitemap.ts, robots.ts, and slug validation
- Updated PHASE-13-PRODUCTION.md with SEO/security/performance audit checklists
- Added Architecture learning to LEARNINGS.md

**v1.9.0** (January 2026)
- **Added CONTENT STRUCTURE GUIDELINES** to critical rules section
- Naming conventions: prefix UI concepts (nav-, header-, footer-) to avoid domain collisions
- Granularity rules: when to create separate entries vs embed as fields
- Entry slug patterns for uniqueness (property-concept, hero-page, section-page-purpose)
- Updated PHASE-5-CONTENT-TYPES.md with detailed naming/granularity guidance
- Added "The Granularity Test" heuristic for decision-making

**v1.8.0** (January 2026)
- **MAJOR: Dynamic routing only** — Removed static page mode entirely
- All content pages now use catch-all route (`[...slug]/page.tsx`)
- Added template system: pages specify `template` field for rendering
- New pages created in CMS are immediately accessible without code changes
- Added DYNAMIC ROUTING ARCHITECTURE section to main docs
- Rewrote PHASE-9-PAGES.md with template-based dynamic routing
- Updated PHASE-5-CONTENT-TYPES.md with required `template` field on page type

**v1.7.0** (January 2026)
- **Component Library Discovery** — Phase 1 now auto-discovers all components from the Figma Components page
- New output: `/project-config/components.md` — Component inventory with node IDs and variants
- Phase 7 updated to reference component inventory before building (prevents duplicate work)
- Helps maintain consistency across build phases

**v1.6.0** (January 2026)
- **MAJOR: Component-based content model** — Content Types now map to Figma components, not pages
- Added Three-Tier Content Model (Global → Components → Pages)
- Completely rewrote PHASE-5-CONTENT-TYPES.md with new architecture
- Added Step 1: Analyze Figma Components before creating content types
- Tier 2 component types: hero-section, content-section, gallery, faq-item, event, team-member, menu-category, award
- Tier 3 page type references component entries for composition
- Content model enables reuse (e.g., FAQs shared across pages)

**v1.5.0** (January 2026)
- **Configuration persistence** — User's pasted configuration is now saved to `/project-config/config.md`
- Added "Step 0: Save Configuration to Project" in PHASE-1-PREFLIGHT.md
- Configuration file serves as project reference document
- Enables re-running skill phases without re-pasting configuration
- Updated workflow documentation with config file creation flow

**v1.4.0** (January 2026)
- **MAJOR: Auto-discovery of Figma pages** — Pages and global components are now automatically discovered from the Figma file using the Figma MCP
- Simplified config.md format — removed manual page table and global component URLs
- Updated PHASE-1-PREFLIGHT.md with new Step 3: Figma Page Discovery
- Pages inventory saved to `/project-config/pages.md` for review/editing
- Configuration form simplified (removed 6 redundant fields)

**v1.3.0** (January 2026)
- Added CMS FIELD DEFINITION RULES to critical rules section
- Created [CMS-FIELD-DEFINITIONS.md](CMS-FIELD-DEFINITIONS.md) comprehensive guide
- Updated PHASE-5-CONTENT-TYPES.md with flat field naming patterns
- Added field type reference (text, richText, media, json, reference)
- Added array handling patterns (JSON vs separate content types)
- Added frontend transformation layer documentation

**v1.2.0** (January 2026)
- Added edit mode cache bypass via middleware + cookie pattern
- Updated PHASE-2-INIT.md with complete middleware.ts and apollo-client.ts code
- Added middleware requirement to technology stack and pitfalls

**v1.1.0** (January 2026)
- Added CONTENT LAYER REQUIREMENT section with `transformImageUrls()` pattern
- Added inline editor image object handling documentation
- Updated pitfalls to include image transformation requirements
- Cross-referenced LEARNINGS.md for full implementation patterns

**v1.0.0** (January 2025)
- Initial skill release
- 14-phase workflow from design to deployment
- Zero hardcoded content enforcement
- Inline editor data attributes support
- Modular reference files for progressive loading

---
> Source: [gautam-lulla/figma-to-code](https://github.com/gautam-lulla/figma-to-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
