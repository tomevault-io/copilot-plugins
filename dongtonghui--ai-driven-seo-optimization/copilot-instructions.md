## ai-driven-seo-optimization

> >


# AI-Driven SEO Optimization & Content Generation

## 1. Core Philosophy

This skill combines two authoritative sources into one actionable workflow:
- **Google Search Central**: E-E-A-T, helpful people-first content, technical SEO basics, and generative AI guidance.
- **QuickCreator Agentic Pipeline**: Topic intelligence → deep research → structured outline → knowledge-base writing → EEAT optimization → contextual conversion.

**Golden Rule**: Content must be created to benefit people, not to manipulate search rankings. AI is a tool for research, structure, and scale—but originality, accuracy, and human value are mandatory.

### 1.1 Ideal User Persona (理想用户画像)

Before creating content, define a detailed persona that represents the target audience. A persona is not just demographics—it is a decision-making prototype.

**Core Dimensions:**

| Dimension | What to Capture | SEO Implication |
|-----------|-----------------|-----------------|
| **Demographics** | Age, gender, income, occupation, education | Tone, examples, pricing transparency |
| **Geography** | City, district, neighborhood landmarks | Local keyword modifiers, landing pages, NAP consistency |
| **Psychographics** | Interests, values, purchase motivations | Content angles, emotional hooks, trust signals |
| **Behavior** | Online channels, search devices, purchase habits | Content format (mobile-first vs. desktop), channel mix |
| **Pain Points & Goals** | Problems they want solved; desired outcomes | Long-tail queries, FAQ topics, CTA design |

**Localization Rule**: For local SEO, go beyond "Shanghai" to specific districts or landmarks (e.g., "静安区搬家服务", "EV charging stations near Times Square"). For multi-language audiences, build a separate persona per language/region and adapt cultural references, tone, and examples.

**Deliverable**: `01-audit-reports/user-persona-<locale>.md`

## 2. Optimization Framework: The 5-Pillar SEO System

| Pillar | Google Principle | QuickCreator Mechanism | Key Actions |
|--------|------------------|------------------------|-------------|
| **P1: Topical Authority** | Helpful, comprehensive, original content | Topic Strategy Agent (semantic clusters, buyer-journey mapping) | Build topic matrices, cover sub-questions, create internal linking architecture |
| **P2: Content Quality (EEAT)** | Experience, Expertise, Authoritativeness, Trust | Content Optimization Agent (EEAT + Main Content Quality scoring) | Add author bios, real examples, source citations, methodology disclosures |
| **P3: Technical Foundation** | Crawlability, structured data, performance | SEO metadata & schema automation | Optimize titles, descriptions, alt text, URLs, schema markup, Core Web Vitals, hreflang, NAP |
| **P4: Intent Alignment** | People-first, answer-ready content | ICP-based research & SERP/PAA analysis | Match content to search intent (Informational, Commercial, Transactional) across 5 journey stages |
| **P5: Conversion Layer** | Great page experience, clear next steps | Conversion Agent (context-aware CTAs) | Embed natural CTAs by journey stage (Awareness → Advocacy), track lead attribution |

## 2.1 User Lifecycle Journey & Search Intent

Map every content piece to one of five journey stages. Each stage has distinct search intent, content type, and distribution channel.

| Stage | User Mindset | Search Intent | Content Types | Channels | CTA Intensity |
|-------|-------------|---------------|---------------|----------|---------------|
| **Awareness** | "I have a problem" | Informational, broad education | Industry guides, news, brand stories, trend reports | SEO, social media, paid display, WeChat articles | Soft (subscribe, read more) |
| **Consideration** | "What are my options?" | Commercial investigation | Comparison posts, case studies, deep guides, reviews | SEO, email nurture, retargeting, WeChat/Weibo | Medium (download, watch demo) |
| **Decision** | "Why should I choose you?" | Transactional, local intent | Product pages, pricing, local landing pages, FAQ | SEO (local pack), direct sales, mini-programs | Hard (buy, get quote, call) |
| **Retention** | "How do I get more value?" | Support, advanced usage | Tutorials, upgrade guides, customer stories, member content | Email, community, in-app, WeChat groups | Engagement (renew, upgrade) |
| **Advocacy** | "I want to tell others" | Branded, social proof | Testimonials, referral programs, UGC campaigns | Social, community, referral, KOL shares | Viral (share, recommend) |

**Localization Note**: In the Decision stage, "near me" and "[City] + service" queries dominate. Build dedicated city/region landing pages with local schema, map embeds, and localized NAP. For Chinese markets, optimize for Baidu Maps and local directory listings (大众点评, 58同城).

## 2.2 Keyword Research Methodology

Keyword research is the bridge between user intent and content. Use the following tool stack and workflow:

**Tool Comparison:**

| Tool | Purpose | Key Feature |
|------|---------|-------------|
| Google Keyword Planner | Keyword discovery, traffic estimates | Free (requires Google Ads account) |
| Google Trends | Trend analysis, seasonal demand | Regional & time-based trend views |
| Ahrefs | Competitive research, backlink analysis | Free site health check; deep paid data |
| SEMrush | Full-suite keyword & SERP analysis | Global database, competitor gap reports |
| AnswerThePublic | Question-based long-tail mining | Visualizes PAA-style queries |
| Baidu Index | Chinese market trend analysis | Region/interest segmentation for Baidu SEO |

**Research Workflow:**
1. **Seed List**: Extract 5-10 core business pillars and their Chinese/English equivalents.
2. **Long-tail Expansion**: Use SEMrush / Keyword Planner to generate long-tail variants; record monthly volume and trend.
3. **Competitor Mapping**: In Ahrefs/SEMrush, identify 3-5 competitors' top-ranking keywords and content depth.
4. **Local Habit Mining**: Collect regional phrasing (e.g., "附近搬家公司", "北京SEO优化", "New York moving service", " dentist near me").
5. **Multi-channel Validation**: Cross-check with GSC internal data, Google Trends/Baidu Index, and autocomplete suggestions.
6. **Intent Tagging**: Label every keyword by journey stage and localization modifier.

**Multi-language Rule**: Conduct separate keyword research for each target language. Mark pages with `hreflang` annotations to signal language/region variants to search engines.

**Deliverable**: `02-keyword-research/keyword-research-report.md`

## 3. Workflow: From Audit to Publish

### Phase 0: Site Audit & Baseline
Before creating new content, audit the existing site:
1. **Technical SEO scan**: Use `technical-seo-audit` skill.
2. **Baseline metrics capture**: TTFB, LCP, CLS, total requests, HTML size.
3. **Content inventory**: List all pages/posts, identify thin content (<300 words), duplicate titles, missing meta descriptions.
4. **Competitor gap analysis**: Identify 3-5 competitors ranking for target keywords; note their content depth, structure, and schema usage.

### Phase 1: Topic Intelligence & Strategy

**Goal**: Build a decision-aware content architecture, not a flat keyword list.

1. **Persona Alignment**: Reference the `user-persona` deliverable. Ensure topics map to real pain points, geographies, and search behaviors.
2. **Seed Input**: Define 3-5 core product/service categories or business pillars.
3. **Demand Analysis**:
   - Extract real user questions from SERP "People Also Ask", related searches, and AnswerThePublic.
   - Analyze competitor content gaps via Ahrefs/SEMrush.
   - Mine local phrasing habits (e.g., city+district+service combinations).
4. **Journey Tagging**: Categorize each topic into one of five stages:
   - **Awareness (TOFU)**: Problem discovery, education
   - **Consideration (MOFU)**: Comparison, vendor evaluation, trade-offs
   - **Decision (BOFU)**: Validation, implementation, pricing, local intent
   - **Retention**: Support, advanced usage, loyalty building
   - **Advocacy**: Social proof, referral, community building
5. **Topic Matrix**: Cross-reference buyer personas with journey stages. Prioritize underrepresented stages.
6. **Semantic Expansion**: For each core topic, fan out 5-10 sub-questions to build topical depth.
7. **Localization Layer**: Identify geo-modified keywords for each core topic. Plan dedicated local landing pages (one per target city/region).

**Deliverable**: `02-keyword-research/topic-matrix.md`

```markdown
### Topic Matrix Template
| Core Topic | Sub-Topic | Journey Stage | Target Keyword | Search Intent | Localization | Priority | Internal Links To |
|------------|-----------|---------------|----------------|---------------|--------------|----------|-------------------|
```

### Phase 2: Content Creation (The Agentic Writing Pipeline)

**Goal**: Produce original, researched, SEO-optimized content that passes Google's helpful-content standards.

**Step 1 — Deep Research**
- Define the Ideal Customer Profile (ICP) for this piece (reference persona deliverable).
- Analyze SERP top 3 results: What do they cover? What do they miss?
- Gather authoritative sources and statistics.
- Map the piece to a specific journey stage and corresponding search intent.

**Step 2 — Outline**
- Create a unique angle (point of view) tied to the business.
- Structure with clear H2s and H3s.
- Plan internal links to product/service pages.
- For local landing pages: plan location-specific sections (local team, service area map, local testimonials, NAP block).

**Step 3 — Writing with Knowledge Base**
- Reference private knowledge (product docs, case studies, internal guides).
- Include real-world examples, data, and expert quotes.
- Avoid generic fluff. Every paragraph should add value.
- For multi-language content: adapt cultural references; do not translate literally.

**Step 4 — SEO Packaging**
- **Title**: 50-60 chars, keyword-forward, compelling. Include city name for local pages.
- **Meta Description**: 120-160 chars, include CTA. Add location modifiers for local pages.
- **URL Slug**: Short, descriptive, hyphenated, keyword-rich. Use pinyin or English for Chinese slugs.
- **H1**: Exactly one per page, descriptive, includes primary keyword.
- **Images**: High-quality, near relevant text, descriptive alt text. Include local imagery for landing pages.
- **Internal Links**: 3-5 contextual links to related pages.
- **Schema**: Add Article/Product/FAQ/LocalBusiness schema as appropriate.
- **FAQ Section**: Add an FAQ block using FAQPage schema to capture PAA/featured snippet opportunities.

**Content Types by Stage:**

| Stage | Primary Content | Supporting Content | Schema / Markup |
|-------|----------------|--------------------|----------------|
| **Awareness** | Blog posts, guides, trend reports | Social snippets, newsletter teasers | Article, BreadcrumbList |
| **Consideration** | Comparison pages, case studies, deep guides | Downloadable PDFs, demo videos | Article, Product, Review |
| **Decision** | Product/service pages, local landing pages | Pricing tables, live chat, map embeds | Product, LocalBusiness, Review, FAQPage |
| **Retention** | Tutorials, knowledge base, customer stories | Email sequences, community posts | Article, FAQPage |
| **Advocacy** | Testimonials, referral pages, UGC galleries | Social share cards, reward dashboards | Review, Organization |

**Deliverable**: `03-content-optimization/article-drafts/`

### Phase 3: EEAT & Quality Optimization

**Goal**: Ensure content demonstrates credibility and originality.

Evaluate every draft against these dimensions:

**EEAT Checklist:**
- [ ] **Experience**: Does it include real-world insight, case studies, or first-hand data?
- [ ] **Expertise**: Does it show subject-matter depth beyond surface-level explanations?
- [ ] **Authoritativeness**: Is the author or brand clearly credited? (author byline, bio, credentials)
- [ ] **Trust**: Are sources cited? Are disclosures (affiliates, AI use) transparent?

**Main Content Quality Checklist:**
- [ ] **Originality**: Does it add a unique perspective or new analysis?
- [ ] **Accuracy**: Are facts precise and supported by credible sources?
- [ ] **Effort & Depth**: Is it comprehensive (typically 800-2500 words for pillar content)?
- [ ] **Clarity & Structure**: Are headings logical? Is it scannable? Are lists and tables used?

**Red flags (fix before publishing):**
- Generic AI fluff ("In today's fast-paced world...")
- Repetitive information without new insight
- Missing author or publication date
- Thin content (<300 words on key pages)
- Keyword stuffing or unnatural phrasing

**Deliverable**: `03-content-optimization/eeat-review-checklist.md`

### Phase 4: Technical On-Page Optimization

**Goal**: Make the content discoverable, properly rendered, and locally relevant by search engines.

Execute this checklist for every page:

1. **Title & Meta**
   - Title: 50-60 chars
   - Meta Description: 120-160 chars
   - OG Title/Description/Image filled
   - Twitter Card configured

2. **Headings**
   - Exactly one H1
   - H2s and H3s form a logical hierarchy
   - Keywords naturally distributed in headings

3. **Images**
   - All images have descriptive alt text (>95% coverage)
   - File names are descriptive (not IMG_001.jpg)
   - Images compressed for web (WebP preferred)

4. **Links**
   - Canonical tag present
   - Internal links use descriptive anchor text
   - No broken outbound links
   - Nofollow used appropriately on sponsored/UGC links

5. **Structured Data**
   - JSON-LD schema validated
   - Article schema for blogs, Product schema for product pages, FAQ schema for Q&A sections
   - **LocalBusiness / Organization schema** for local SEO (name, address, phone, geo, opening hours)
   - **Service schema** for service-oriented pages
   - **Review schema** for testimonials and ratings

6. **Mobile & Performance**
   - Page passes mobile-friendly test
   - LCP < 2.5s, CLS < 0.1, TTFB < 800ms
   - Responsive design or dedicated mobile site with proper viewport meta tag
   - CDN, image compression, Gzip/Brotli enabled

7. **Multi-language & Localization**
   - `hreflang` tags in `<head>` or HTTP headers for every language/region variant
   - HTML `lang` attribute set correctly per page
   - NAP (Name, Address, Phone) consistent across site footer, contact page, and third-party directories
   - Local landing pages include embedded map and district-specific imagery

8. **Page Structure & Indexing**
   - Each page targets one core keyword cluster
   - Title and first paragraph include primary keyword
   - Body uses related terms naturally with clear heading hierarchy
   - XML sitemap submitted and 404 errors fixed

**Deliverable**: `04-technical-fixes/on-page-seo-checklist.md`

### Phase 5: Conversion Optimization

**Goal**: Turn traffic into measurable business outcomes across the full lifecycle.

1. **Journey-Stage CTA Mapping**:
   - **Awareness (TOFU)**: Soft CTAs (subscribe, download guide, read related article)
   - **Consideration (MOFU)**: Medium CTAs (compare solutions, watch demo, request consultation)
   - **Decision (BOFU)**: Hard CTAs (get quote, free trial, contact sales, book appointment)
   - **Retention**: Engagement CTAs (join community, access premium content, schedule onboarding)
   - **Advocacy**: Viral CTAs (share story, leave review, refer a friend, join ambassador program)

2. **Placement Rules**:
   - Place a CTA after the value proposition is established (usually within first 30% for BOFU).
   - Add a mid-content CTA after a key insight.
   - End with a clear next-step CTA.
   - For local pages: place "Call Now" and "Get Directions" above the fold.

3. **Lead Attribution**:
   - Tag all forms/CTAs with UTM parameters or event tracking.
   - Ensure lead data flows to CRM/email tool.
   - Track local conversions separately (phone calls, direction requests, store visits).

4. **Trust Signals by Stage**:
   - **Decision**: Client logos, review counts, guarantees, secure payment badges
   - **Retention**: Onboarding videos, dedicated support contacts
   - **Advocacy**: Referral rewards, social proof counters, share buttons

**Deliverable**: `05-backlinks-outreach/conversion-map.md`

### Phase 6: Automated Execution on WordPress + Elementor

**Goal**: Apply SEO fixes directly on the live site without manual GUI clicking for every page.

When the target site runs on WordPress (especially with Elementor + Yoast SEO), use a **hybrid browser + injected-JavaScript** approach. Navigate in the browser to establish an authenticated session, then execute updates via the WordPress REST API or the page's exposed JS objects.

#### 6.1 Core Execution Principles

1. **Authenticate first**: Navigate to `/wp-login.php`, log in, then go to any `/wp-admin/` page.
2. **Extract the nonce** from the authenticated page context:
   ```javascript
   const nonce = window.wpApiSettings?.nonce ||
     document.querySelector('script#wp-api-request-js')?.innerText?.match(/"nonce":"([^"]+)"/)?.[1];
   ```
3. **Prefer REST API for bulk updates** (fastest, most reliable for text/meta fields).
4. **Use Elementor's internal JS API** (`window.elementor.elementsModel`) for layout-level changes inside the editor.
5. **Verify after saving**: Reload the live page and re-run the audit script to confirm persistence.

> **Pitfall**: The REST API requires a valid session cookie + nonce. POSTing from an external script with username/password Basic Auth usually returns **401 Unauthorized** unless an Application Password plugin is installed. Always inject `fetch()` calls into the authenticated browser page.

#### 6.2 Bulk-Update Yoast SEO Meta (REST API)

The fastest way to fix missing focus keywords and meta descriptions across pages/posts:

```javascript
const nonce = '...'; // extracted above
const updates = [
  { id: 123, type: 'pages', focuskw: 'EV charging station', metadesc: 'High-quality EV charging stations for commercial and residential use. Explore FBK POWER solutions today.' },
  { id: 124, type: 'pages', focuskw: 'DC fast charger', metadesc: 'Reliable DC fast chargers designed for fleet operators. View specifications and request a quote.' },
];

(async () => {
  for (const item of updates) {
    const res = await fetch(`/wp-json/wp/v2/${item.type}/${item.id}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json', 'X-WP-Nonce': nonce },
      body: JSON.stringify({
        meta: {
          _yoast_wpseo_focuskw: item.focuskw,
          _yoast_wpseo_metadesc: item.metadesc
        }
      })
    });
    console.log(item.id, res.status);
  }
})();
```

If the REST API silently fails to persist Yoast meta on a specific site configuration, fall back to the **Gutenberg data layer** in the block editor:

```javascript
wp.data.dispatch('core/editor').editPost({
  meta: {
    _yoast_wpseo_focuskw: 'your-keyword',
    _yoast_wpseo_metadesc: 'Your meta description.'
  }
});
wp.data.dispatch('core/editor').savePost();
```

Or, as a tertiary fallback, manipulate the Yoast inputs directly in the DOM and click the WordPress Save button.

#### 6.3 Fix Image Alt Text in Bulk

**Media Library (REST API)** — for images that appear in many places:

```javascript
const nonce = '...';
const alts = [
  { id: 2401, alt_text: 'FBK POWER Wall-Mounted AC Charging Station' },
  { id: 2402, alt_text: 'Split-Type DC Charging Cabinet in warehouse' },
];

(async () => {
  for (const item of alts) {
    await fetch(`/wp-json/wp/v2/media/${item.id}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json', 'X-WP-Nonce': nonce },
      body: JSON.stringify({ alt_text: item.alt_text })
    });
  }
})();
```

**Elementor Pages (JS API)** — for images embedded in Elementor-built pages:

> **Critical**: Elementor `image-box` widgets render `img alt` from the `alt_text` setting field, not from `image.alt`. You must set **both**.

```javascript
function walk(model, callback) {
  const attrs = model.attributes || {};
  callback({ id: attrs.id, widgetType: attrs.widgetType || attrs.elType, settings: attrs.settings });
  if (attrs.elements?.models?.length) attrs.elements.models.forEach(m => walk(m, callback));
}

walk(window.elementor.elementsModel, ({ widgetType, settings }) => {
  if (widgetType === 'image' && settings) {
    const img = settings.get('image') || {};
    img.alt = 'Descriptive alt text';
    settings.set('image', { ...img });
  }
  if (widgetType === 'image-box' && settings) {
    const img = settings.get('image') || {};
    const alt = 'Descriptive alt text for image box';
    img.alt = alt;
    settings.set('image', { ...img });
    settings.set('alt_text', alt); // required for frontend output
  }
});

window.elementor.saver.saveEditor({ status: 'publish' });
```

If an image appears on multiple pages but is absent from the current page's `elementsModel`, it likely lives in a **global template** (Header, Footer, Loop Item, or Single template). Edit it via *Elementor > Templates* or *Theme Builder*.

#### 6.4 Modify Elementor Headings & Buttons via JS API

Avoid clicking the brittle sidebar UI. Traverse the Backbone model directly:

```javascript
walk(window.elementor.elementsModel, ({ id, widgetType, settings }) => {
  // Heading: change H5 → H3
  if (widgetType === 'heading' && settings) {
    settings.set('header_size', 'h3'); // required
    settings.set('html_tag', 'h3');    // also set for consistency
  }
  // Button: update text
  if (widgetType === 'button' && settings) {
    settings.set('text', 'Request a Quote');
  }
});
window.elementor.saver.saveEditor({ status: 'publish' });
```

> **Warning**: Direct DOM manipulation inside `#elementor-preview-iframe` (e.g., changing tags or button text via `document.querySelector`) may look correct in the preview but will **not** sync to Elementor's `_elementor_data` JSON. Only edits through `elementsModel` persist reliably.

#### 6.5 Bulk-Create Posts/Pages via REST API

When populating a blog or product page set, create items directly:

```javascript
const nonce = '...';
const posts = [
  {
    title: 'How to Choose the Right AC Charging Station',
    content: `<p>For compact spaces, the <a href="https://example.com/product/feva12021d/">Wall-Mounted AC Charging Station-FEVA12021D</a> is ideal...</p>`,
    status: 'publish',
    categories: [7]
  }
];

(async () => {
  for (const post of posts) {
    const res = await fetch('/wp-json/wp/v2/posts', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json', 'X-WP-Nonce': nonce },
      body: JSON.stringify(post)
    });
    const data = await res.json();
    console.log(data.id, data.link);
  }
})();
```

> **Tip**: Resolve valid category IDs first via `/wp-json/wp/v2/categories?per_page=100`. Invalid IDs are silently dropped and replaced with "Uncategorized."

#### 6.6 Widget Blocks (Footer/Sidebar Alt Fixes)

Images in footers or sidebars may live in Widgets rather than pages. Update them via the widgets endpoint:

```javascript
fetch('/wp-json/wp/v2/widgets')
  .then(r => r.json())
  .then(widgets => widgets.map(w => ({ id: w.id, sidebar: w.sidebar, content: w.instance?.raw?.content })))
```

Then POST the updated block HTML back, preserving the block comment JSON (`<!-- wp:image {...} -->`).

#### 6.7 Execution Checklist

- [ ] Extracted a fresh `X-WP-Nonce` from an authenticated `/wp-admin/` page
- [ ] Verified REST API response is 200 (not 403 "Cookie check failed")
- [ ] For Elementor changes: used `window.elementor.elementsModel`, not iframe DOM selectors
- [ ] For image-box widgets: set both `image.alt` and `alt_text`
- [ ] For heading widgets: changed both `header_size` and `html_tag`
- [ ] Saved via `window.elementor.saver.saveEditor({status: 'publish'})` or Gutenberg `savePost()`
- [ ] Re-loaded the live (incognito) page and confirmed changes with `browser_console`
- [ ] Cleared server/CDN cache if applicable

## 4. AI Content Compliance (Google Guidelines)

When using AI to generate or assist content:

1. **Accuracy First**: Fact-check all AI-generated claims. Hallucinations destroy trust and rankings.
2. **Value Addition**: AI should structure and research; human expertise should add unique insights, examples, and judgments.
3. **Transparency**: Disclose AI use where appropriate (especially in YMYL niches).
4. **Avoid Scaled Content Abuse**: Do not mass-produce low-value pages. Every page must serve a distinct user need.
5. **Metadata Quality**: AI-generated titles, descriptions, and alt text must be accurate and relevant.

## 4.1 Localization & Multi-language SEO Strategy

For sites targeting multiple languages or regions, apply these additional rules:

1. **hreflang Implementation**: Add `<link rel="alternate" hreflang="x" href="URL" />` tags in the `<head>` of every page. Include a self-referencing hreflang and an `x-default` fallback. Example:
   ```html
   <link rel="alternate" hreflang="en-us" href="https://example.com/en-us/service" />
   <link rel="alternate" hreflang="zh-cn" href="https://example.com/zh-cn/service" />
   <link rel="alternate" hreflang="x-default" href="https://example.com/service" />
   ```

2. **HTML lang Attribute**: Each page must declare its language via `<html lang="zh-CN">` or `<html lang="en-US">`.

3. **NAP Consistency**: Ensure Name, Address, Phone are identical across:
   - Website footer and contact page
   - Google Business Profile / Baidu Maps / 百度地图
   - Industry directories (大众点评, 58同城, Yelp, Yellow Pages)
   - Even minor formatting differences (e.g., "Suite" vs "Ste") can dilute local ranking signals.

4. **Local Landing Pages**: Create one dedicated page per target city/region. Each page must include:
   - City name in title, H1, URL slug, and meta description
   - Local service description and team intro
   - Embedded map and accurate NAP block
   - Local customer testimonials with Review schema
   - Internal links to broader service pages

5. **Maps & Citations**: Claim and optimize Google Business Profile (or Baidu Maps for China). Post updates, respond to reviews, and ensure category selection matches target keywords.

6. **Multimedia Localization**: Translate/subtitle videos, adapt image alt text, and ensure infographics use culturally appropriate examples.

## 5. Deliverables Structure

For every SEO engagement, store outputs in this structure:

```
fbkpower-seo/
├── 01-audit-reports/
│   ├── technical-seo-audit-<date>.md
│   ├── competitor-gap-analysis.md
│   └── user-persona-<locale>.md
├── 02-keyword-research/
│   ├── topic-matrix.md
│   ├── keyword-research-report.md
│   └── keyword-opportunities.md
├── 03-content-optimization/
│   ├── article-drafts/
│   │   ├── <slug>.md
│   │   └── ...
│   ├── eeat-review-checklist.md
│   ├── content-calendar.md
│   └── local-landing-pages/
│       ├── <city>-<service>.md
│       └── ...
├── 04-technical-fixes/
│   ├── on-page-seo-checklist.md
│   ├── schema-implementations.json
│   ├── performance-baseline.md
│   └── hreflang-mapping.md
├── 05-backlinks-outreach/
│   ├── conversion-map.md
│   ├── outreach-targets.md
│   └── local-citations-list.md
└── 06-deliverables/
    └── seo-strategy-summary-<date>.md
```

## 6. Pitfalls

- **Topic islands**: Writing one-off posts without internal linking clusters. Always build topic authority.
- **AI-first, human-second**: Let AI handle research and structure; never publish raw AI output without expert review.
- **Ignoring EEAT signals**: Anonymous content without sources or author bios underperforms in competitive SERPs.
- **Skipping baseline metrics**: Without before/after data, you cannot prove ROI.
- **CTA mismatch**: Asking for a demo in an awareness-stage article creates friction. Match CTA intensity to intent.
- **Neglecting local schema**: Missing LocalBusiness or Review schema reduces visibility in local pack results.
- **Inconsistent NAP**: Any variation in name, address, or phone across directories weakens local SEO trust signals.
- **Missing hreflang**: Without proper language/region annotations, search engines may serve the wrong version to users.
- **Thin local pages**: City landing pages with only a title change and no local-specific content are treated as doorway pages.
- **WordPress + Elementor-specific**: Bulk DOM edits in the preview iframe do NOT persist. Use `window.elementor.elementsModel` or the WordPress REST API for reliable updates. See Phase 6 and `wordpress-browser-api-automation` skill.
- **Yoast "Search Appearance" modal trap**: In the block editor, clicking Yoast's "Search Appearance" often opens a full-screen modal instead of expanding the sidebar. Use `browser_vision` to confirm UI state before typing.
- **Nonce expiration**: `X-WP-Nonce` expires on page reload or timeout. If a call returns 403, re-extract from an active `/wp-admin/` page.
- **Global templates hide images**: If an image appears on the frontend but not in the current page's `elementsModel`, it lives in an Elementor global template (Header/Footer/Theme Builder). Edit the template, not the page.
- **Yoast React settings do NOT persist via raw DOM mutation**: Yoast settings pages (e.g. *SEO → Settings → Breadcrumbs*) use React-managed inputs. Injecting `document.querySelector('#input-wpseo_titles-breadcrumbs-home').value = 'Home'` and dispatching input/change events will appear to work on screen but will **not** persist after clicking Save. Use actual browser interactions (`browser_type` into the focused input, then `browser_click` the Save button) so React state is updated, or edit the option directly via `wp-cli` / custom PHP if server access is available.
- **Widget block JSON fragility**: When updating widget HTML via REST API, preserve the `<!-- wp:image {...} -->` block comments. Removing them breaks the block editor rendering.

## 7. Quick Reference: Content Quality Questions (Google)

Before publishing any page, ask:
- Does the content provide original information, reporting, research, or analysis?
- Does it provide a substantial, complete, or comprehensive description of the topic?
- Does it provide insightful analysis or interesting information that is beyond the obvious?
- Does the main heading or page title provide a descriptive, helpful summary of the content?
- Would you expect to see this content in or referenced by a printed magazine, encyclopedia, or book?

If the answer to any of these is "no," revise before publishing.

---
> Source: [dongtonghui/ai-driven-seo-optimization](https://github.com/dongtonghui/ai-driven-seo-optimization) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
