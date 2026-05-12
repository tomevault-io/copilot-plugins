## seo-agent-stack

> > **Instructions**: Replace all `{{...}}` values with your own information.

# {{PROJECT_NAME}} — SEO Project

## Configuration (FILL IN)

> **Instructions**: Replace all `{{...}}` values with your own information.
> Once filled, this file is the single source of truth for all agents.
> The `/setup` command can fill this automatically.

| Variable | Value |
|----------|-------|
| **Site URL** | `{{SITE_URL}}` |
| **Site Name** | `{{SITE_NAME}}` |
| **Sector** | `{{SECTOR}}` (e.g., medical imaging, e-commerce, SaaS, restaurant) |
| **Content Language** | `{{LANGUAGE}}` (e.g., fr, en, es) |
| **CSS Prefix** | `ss-` (default — SEO Stack) |

## Credentials & Connections (FILL IN)

| Resource | Identifier |
|----------|-----------|
| **Webflow Site ID** | `{{WEBFLOW_SITE_ID}}` |
| **Webflow Workspace ID** | `{{WEBFLOW_WORKSPACE_ID}}` |
| **GA4 Property ID** | `{{GA4_PROPERTY_ID}}` |
| **GSC Property** | `{{GSC_PROPERTY}}` (e.g., `sc-domain:example.com`) |

## Webflow CMS Collections (FILL IN)

> List your Webflow collections. The `/setup` command auto-discovers these.
> Agents use these IDs to publish content.

| Collection | ID | Slug | Items |
|------------|----|------|-------|
| **Articles** (blog) | `{{COLLECTION_ARTICLES_ID}}` | `/{{ARTICLES_SLUG}}` | — |
| **Service Pages** | `{{COLLECTION_SERVICES_ID}}` | `/{{SERVICES_SLUG}}` | — |
| **Sub-pages** | `{{COLLECTION_SUBPAGES_ID}}` | `/{{SUBPAGES_SLUG}}` | — |

> **Note**: You may have fewer or more collections. Add/remove rows as needed.

### CMS Schema — Articles (main blog collection)

> Adapt this schema to your actual Webflow fields. The `/setup` command auto-detects these.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `name` | PlainText | yes | Article title |
| `slug` | PlainText | yes | URL slug (max 256 chars, alphanum) |
| `meta-description` | PlainText | yes | Max 150 characters |
| `short-info` | PlainText | yes | Short description |
| `category` | PlainText | yes | Category (free text) |
| `thumb-image` | Image | yes | Thumbnail — {{THUMB_DIMENSIONS}} px |
| `main-image` | Image | yes | Hero image — {{MAIN_IMAGE_DIMENSIONS}} px |
| `details` | RichText | yes | Article content (HTML) |

### CMS Schema — Service Pages

> Document your service/product collection fields here.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `name` | PlainText | yes | Service name |
| `slug` | PlainText | yes | URL slug |
| `meta-title` | PlainText | yes | SEO meta title |
| `meta-description` | PlainText | yes | SEO meta description |
| `overview` | RichText | no | Main content |
| `thumb-image` | Image | no | Thumbnail |
| `main-image` | Image | no | Hero image |

## Existing Content (FILL IN)

> List pages and articles already published on your site.
> Agents use this for internal linking and to avoid cannibalization.
> The `/setup` command auto-discovers content from Webflow.

### Published Service Pages

- [Service Name 1] (`/{{SERVICES_SLUG}}/[slug]`)
- [Service Name 2] (`/{{SERVICES_SLUG}}/[slug]`)

### Published Blog Articles

- [Article Title 1] (`/{{ARTICLES_SLUG}}/[slug]`)
- [Article Title 2] (`/{{ARTICLES_SLUG}}/[slug]`)

### Static Pages

| Page | URL |
|------|-----|
| Home | `/` |
| Blog | `/{{ARTICLES_SLUG}}` |
| Contact | `/contact` |
| About | `/about` |

## URL Structure (FILL IN)

- Blog articles: `{{SITE_URL}}/{{ARTICLES_SLUG}}/[slug]`
- Service pages: `{{SITE_URL}}/{{SERVICES_SLUG}}/[slug]`
- Sub-pages: `{{SITE_URL}}/{{SUBPAGES_SLUG}}/[slug]`

## Business Context (FILL IN)

> Filled automatically by `/setup`. This context drives all SEO strategy decisions.

### Business Model

| Aspect | Details |
|--------|---------|
| **Revenue model** | `{{REVENUE_MODEL}}` (e.g., service sales, e-commerce, SaaS, lead gen) |
| **Flagship offer** | `{{FLAGSHIP_OFFER}}` |
| **Average deal size** | `{{DEAL_SIZE}}` (e.g., 50-200€, 5k-20k€) |
| **Conversion funnel** | `{{CONVERSION_FUNNEL}}` (e.g., visit → contact → call → sale) |

### Acquisition & Competition

| Aspect | Details |
|--------|---------|
| **Primary acquisition channel** | `{{PRIMARY_CHANNEL}}` (e.g., word of mouth, Google Ads, organic, social) |
| **Current organic traffic** | `{{ORGANIC_TRAFFIC_LEVEL}}` (e.g., none, ~500/mo, 10k+/mo) |
| **Main competitors** | `{{COMPETITORS}}` (URLs if available) |
| **Key differentiator** | `{{DIFFERENTIATOR}}` |

### Market & Geography

| Aspect | Details |
|--------|---------|
| **Geographic scope** | `{{GEO_SCOPE}}` (local, regional, national, international) |
| **Target areas** | `{{TARGET_AREAS}}` (cities, regions — if local/regional) |
| **Seasonal patterns** | `{{SEASONALITY}}` (e.g., summer peak, Q4 slow) |

### SEO Maturity

| Aspect | Details |
|--------|---------|
| **SEO history** | `{{SEO_HISTORY}}` (e.g., never done, tried and stopped, active agency) |
| **Publishing frequency** | `{{PUBLISH_FREQUENCY}}` (e.g., never, 1/month, 2/week) |
| **Business goals (6-12 months)** | `{{BUSINESS_GOALS}}` |

### SEO Strategy (generated by `/setup` analysis)

> This section is filled by Claude's analysis of the business context above.

| Element | Details |
|---------|---------|
| **Priority strategy** | `{{PRIORITY_STRATEGY}}` (e.g., local SEO + service pages, content marketing, technical SEO) |
| **Content priorities** | `{{CONTENT_PRIORITIES}}` (ranked by business impact) |
| **Intent mapping** | `{{INTENT_MAPPING}}` (which intent types match the funnel: transactional, informational, navigational) |

## Priority Target Keywords (FILL IN)

> List your 10-20 main keywords. Agents use these for content strategy.
> `/setup` expands these with long-tail variants, local modifiers, and funnel-stage keywords.

- keyword 1, keyword 1 + location
- keyword 2, keyword 2 variant
- keyword 3 long tail

## Stack & Tools

| Tool | Usage |
|------|-------|
| **Webflow MCP** | CMS management, article/page publication, on-page SEO |
| **Google Search Console MCP** | Positions, impressions, CTR, search queries |
| **Google Analytics 4 MCP** | Traffic, engagement, user behavior, events |
| **Agent seo-planner** | GSC + GA4 analysis, opportunity identification, editorial calendar |
| **Agent seo-article-writer** | SEO article/page writing, saved locally in `articles/` |
| **Agent seo-publisher** | Quality control (accents, HTML, SEO, linking) + Webflow CMS publication |
| **Agent google-analytics-insight** | Dedicated GA4 analysis: traffic, user behavior, KPIs |
| **Agent gsc-insight** | Dedicated GSC analysis: positions, queries, CTR, indexation, sitemaps |

> **Note**: No database. All content is managed via Webflow CMS only.

## Available Commands

| Command | Description |
|---------|-------------|
| `/setup` | Interactive setup wizard — configures everything |
| `/health` | Diagnostic check — tests all connections and config |
| `/seo-planner` | Strategic GSC + GA4 cross-analysis, data-driven editorial plan |
| `/seo-article-writer` | Write an SEO-optimized article or page |
| `/seo-publisher` | Quality control + publish to Webflow CMS |
| `/google-analytics-insight` | Dedicated traffic and user behavior analysis via GA4 |
| `/gsc-insight` | Dedicated Search Console performance analysis |
| `/seo-pipeline` | Full pipeline: analysis → writing → QC → images → publication |
| `/report-seo` | Generate a quarterly SEO performance report |

## Typical Workflow

1. **Quick analysis**: `/gsc-insight` and `/google-analytics-insight` in parallel for a status check
2. **Strategy**: `/seo-planner` to cross GSC + GA4 and plan content
3. **Writing**: `/seo-article-writer` to create content (saved in `articles/`)
4. **Quality control**: `/seo-publisher` to check accents, HTML, SEO, internal linking
5. **Publication**: `/seo-publisher` publishes to Webflow CMS after your approval
6. **Tracking**: `/gsc-insight` for positions, `/google-analytics-insight` for traffic

---

## Webflow CMS Rules (MANDATORY)

These rules apply to ALL agents that create or publish content on Webflow.

### Content Quality BEFORE Push

- **Correct diacritics/accents** (if applicable): ALWAYS verify content has correct accents BEFORE pushing to Webflow. Never publish content with missing accents.
- **Accurate sector terminology**: Use correct domain terms while keeping language accessible.
- **Non-breaking spaces**: Use `&nbsp;` in prices (e.g., `50&nbsp;&euro;`)
- **Quotation marks**: Use the appropriate style for the content language (`&laquo;`/`&raquo;` for French, `&ldquo;`/`&rdquo;` for English)

### Tables = ALWAYS in Embeds

Raw HTML tables (`<table>`) do NOT display correctly in Webflow RichText. **Always** wrap them in an embed:

```html
<div data-rt-embed-type='true'>
<style>/* inline CSS here */</style>
<div class="ss-table-wrap">
  <table class="ss-table">...</table>
</div>
</div>
```

Table styling rules:
- Dark header: `background:#1a1a2e; color:#fff`
- Zebra striping: `tr:nth-child(even){background:#f7f7fb}`
- Hover: `tr:hover{background:#eef0ff}`
- Responsive: `overflow-x:auto` on wrapper + mobile media query
- Prefix CSS classes with `ss-` (SEO Stack default)

### Custom HTML Embeds

All custom HTML (CTAs, widgets, info blocks) must be wrapped in:
```html
<div data-rt-embed-type='true'>
  <!-- HTML content here -->
</div>
```

### Webflow CMS API — Required Fields

When creating a CMS item (`create_collection_items`):
- `cmsLocaleIds: []` — MANDATORY even for a non-localized site
- `isArchived: false` — MANDATORY
- **Images**: format `{url: "...", alt: "..."}`, NOT `{fileId: "..."}`

### Webflow API Limitations

- **No scheduling via API**: Publication scheduling must be done in Webflow Designer
- **RichText**: Supports standard HTML + embeds via `data-rt-embed-type`

### Webflow RichText Limitations (CRITICAL)

Webflow RichText (via CMS API) **automatically strips** the following elements from **static HTML** in embeds:
- `<input>`, `<select>`, `<option>`, `<textarea>` — form elements
- `<button>` — action buttons
- `<form>` — form containers

**Preserved elements**: `<script>`, `<style>`, `<div>`, `<span>`, `<a>`, `<table>`, `<ul>/<li>`, `<details>/<summary>`, `<img>`

## Writing Conventions

- Articles in the **configured language** with **correct accents/diacritics** (MANDATORY)
- Tone adapted to the sector and target audience (configure below)
- Structure: H1 > H2 > H3 with clear hierarchy
- Internal linking to other articles and service pages (use URLs from the "URL Structure" section)
- Meta title < 60 characters, meta description < 155 characters
- Natural keyword insertion (no keyword stuffing)
- Comparison tables as embedded HTML (see Webflow rules above)

### Tone & Style (CONFIGURE)

> Describe the tone agents should use when writing.

- **Main tone**: `{{TONE}}` (e.g., professional and reassuring, casual and expert, corporate and premium)
- **Target audience**: `{{TARGET_AUDIENCE}}` (e.g., patients, B2C buyers, B2B decision makers)
- **E-E-A-T strategy**: `{{EEAT_STRATEGY}}` (e.g., cite medical sources HAS/SFR, cite client case studies, cite ISO standards)

### Component Labels (CONFIGURE)

> Set the display text for visual components in your content language.

| Component | Label |
|-----------|-------|
| Key Takeaways | `{{LABEL_TAKEAWAY}}` (e.g., "Key Takeaways", "L'essentiel a retenir", "Lo esencial") |
| Good to Know | `{{LABEL_INFO}}` (e.g., "Good to Know", "Bon a savoir", "Dato importante") |
| CTA Button | `{{LABEL_CTA_BUTTON}}` (e.g., "Book Now", "Prendre rendez-vous", "Reservar") |
| CTA URL | `{{CTA_URL}}` (e.g., `/contact`, `/book`, `/rendez-vous`) |

## SEO Topic Clusters (FILL IN)

> Organize your topics into clusters (content silos).

### 1. Main Cluster

- Pillar topic: [description]
- Sub-topics: [list]

### 2. Secondary Cluster

- Pillar topic: [description]
- Sub-topics: [list]

### 3. Blog / Long-tail Cluster

- Educational articles
- Comparisons and guides
- FAQ and practical tips

## Project Files

```
{{PROJECT_NAME}}/
  CLAUDE.md              # This file — project config
  .claude/
    agents/              # Agent definitions
    commands/            # Slash commands
    skills/              # Technical skills
  articles/              # Draft articles
  analyses/              # SEO analysis reports
  planning/              # Editorial calendar and planning
  rapport/               # Client quarterly reports
```

> **Important**: Document your strategy in `planning/strategie.md` before writing articles. This document guides agents in topic selection and internal linking.

---
> Source: [jadedagher/seo-agent-stack](https://github.com/jadedagher/seo-agent-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
