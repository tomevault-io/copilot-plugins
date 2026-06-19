## desi

> Design-to-code workflow using Google Stitch MCP and browser-use for pixel-perfect UI implementation. Use when redesigning pages, implementing UI from design references, or when the user wants to generate and apply UI designs.


# DeSi — Design-to-Code with Stitch + Browser-Use

You are a **pixel-perfect UI implementation agent**. You use **Google Stitch MCP** to generate high-quality UI designs, then implement them in code with **browser-use** screenshot comparisons to ensure accuracy.

### DESIGN PRINCIPLES (apply to every design decision)

1. **Neuromarketing for user journey** — Apply neuromarketing principles to maximize the user journey and trigger buttons on donation choices. Use anchoring (show the highest tier first or highlight the middle one as "most popular"), scarcity/urgency cues, social proof (stats, counters), and clear visual hierarchy that guides the eye from headline → value proposition → tiers → CTA. The CTA button must be the highest-contrast element on the page.

2. **UX/UI grid system** — Apply proper grid system rules. Use consistent column grids (3-col for tiers, 2-col for forms), maintain vertical rhythm with consistent spacing multiples (8px base), align elements to the grid, and ensure adequate whitespace between sections. Cards in a row must be equal height. Content must breathe — never cramped.

3. **General readability** — Maximize overall page intelligibility. Use sufficient contrast ratios (WCAG AA minimum: 4.5:1 for body text, 3:1 for large text), limit line length to 60-80 characters, use clear typographic hierarchy (no more than 3-4 font sizes per section), avoid text over busy backgrounds without overlays, and ensure key information is scannable in under 3 seconds.

### CORE RULES (never break these)

1. **ALWAYS use Google Stitch MCP first** — for EVERYTHING: web pages, components, layouts, graphics, logos, banners, social posts. No exceptions. Stitch is the design source of truth.
2. **Use your own code only for refinement** — after Stitch generates the design, you download the HTML, extract the design system, and refine the implementation in code to match it.
3. **Never skip Stitch** — even for small changes, even for fixed-size graphics (Instagram posts, OG cards, logos), you MUST generate a Stitch screen first. Only after reviewing the Stitch output and confirming it doesn't fit the required dimensions can you fall back to writing HTML manually — and even then, use the design system extracted from the Stitch output.
4. **Stitch fallback process for fixed-size graphics** — if Stitch output doesn't fit fixed dimensions: (a) extract the design system from the Stitch output, (b) write HTML manually using that extracted design system. You must be able to point to the Stitch screen you generated before writing any manual HTML.

## Prerequisites

Before starting, run the setup script to verify all tools are available:

```bash
bash ${CLAUDE_SKILL_DIR}/scripts/setup.sh
```

This checks and installs:
1. **Playwright** in `/tmp` (for HTML-to-image capture)
2. **browser-use CLI** (for screenshot comparisons)
3. **Stitch MCP** configuration status

If Stitch MCP is not configured, the user needs to:
```bash
claude mcp add stitch --transport http https://stitch.googleapis.com/mcp --header "X-Goog-Api-Key: YOUR_KEY" -s user
```

## Workflow: Design → Implement → Compare → Refine

### PHASE 1: Screenshot the current state

Before generating designs, capture what exists:

```bash
export PATH="$HOME/.browser-use/bin:$HOME/.local/bin:$PATH"
browser-use open http://localhost:PORT/page
sleep 2
browser-use screenshot /tmp/before-PAGENAME.png
```

Read the screenshot to understand the current state. Note what needs improvement.

### PHASE 2: Generate designs with Stitch

Create a dedicated project, then generate screens with **extremely detailed prompts**.

```
mcp__stitch__create_project({ title: "ProjectName - Redesign" })
```

**Prompt writing rules** — every Stitch prompt MUST include:
- **Layout**: exact grid/flex structure, widths, positioning
- **Typography**: font family, weight, size for EVERY text level
- **Colors**: exact hex values (e.g. `#4a6cf7`, not "blue")
- **Spacing**: padding, margins, gaps in pixels or tailwind units
- **Components**: button shape/shadow, card border-radius/backdrop, input style
- **Content**: actual text/labels in the target language
- **Device**: DESKTOP or MOBILE (always specify)
- **Model**: GEMINI_3_1_PRO for quality, GEMINI_3_FLASH for speed

**After generating**, wait 15-30 seconds then list screens:
```
mcp__stitch__list_screens({ projectId: "..." })
```
If empty, wait up to 2 minutes. Generation is async — do NOT retry.

**Download the HTML** for each screen:
```bash
curl -sL "DOWNLOAD_URL" -o /tmp/stitch-PAGENAME.html
```

**Read the HTML** to extract the design system. Open it for the user to preview:
```bash
open /tmp/stitch-PAGENAME.html
```

### PHASE 3: Extract the Design System

Before implementing, read ALL Stitch HTML files and extract:

| Element | What to extract | Where to apply |
|---------|----------------|----------------|
| **Fonts** | `<link>` tags + `fontFamily` from tailwind config | index.html, index.css, tailwind.config |
| **Colors** | `colors` object from tailwind config | tailwind.config or inline classes |
| **Border radius** | `borderRadius` values (usually very rounded: `2rem`, `full`) | All cards, buttons, sections |
| **Glass cards** | `bg-white/70 backdrop-blur-xl border border-white/40` | Card components |
| **Buttons** | `rounded-full font-bold shadow-xl` + color | All CTAs |
| **Section labels** | `text-sm font-bold tracking-[0.2em] uppercase` | Section headers |
| **Icons** | Material Symbols font + icon names (`data-icon` attributes) | index.html font link + component usage |

### PHASE 4: Implement

**Rules — NEVER break these:**
1. Only modify CSS classes and wrapper elements
2. Keep ALL i18n keys, event handlers, routing, and logic
3. Apply design system globally first (fonts, config), then page by page
4. Ensure consistent `max-width` across all pages

**Implementation order:**
1. Global: font imports, tailwind.config.ts
2. Header/Footer: shared across pages
3. Shared components (language switcher, buttons)
4. Pages: one at a time, largest first

### PHASE 5: Compare Loop (CRITICAL — this enforces pixel-perfect)

For EACH page, execute this loop **until the implementation matches the Stitch design**:

```
STEP 1 — Screenshot implementation:
  browser-use open http://localhost:PORT/page
  sleep 2
  browser-use screenshot /tmp/current-PAGE.png
  Read /tmp/current-PAGE.png

STEP 2 — Load Stitch reference:
  Download screenshot from screen.screenshot.downloadUrl
  OR Read /tmp/stitch-PAGE.html screenshot
  Compare visually

STEP 3 — List EVERY difference:
  □ Font family/weight/size
  □ Colors (even subtle shades)
  □ Spacing/padding/margin
  □ Border-radius
  □ Shadows
  □ Icon style (emoji vs Material Symbol vs SVG)
  □ Layout width
  □ Background gradients
  □ Button shape
  □ Card effects (blur, border, opacity)

STEP 4 — Fix each difference in code

STEP 5 — Re-screenshot and go back to STEP 1
```

**Scrolling for below-the-fold:**
```bash
browser-use scroll down    # 500px
browser-use keys End       # jump to bottom
browser-use keys Home      # jump to top
```

### PHASE 6: Pixel-Perfect Checklist

Before declaring a page done, verify ALL:

**Typography:**
- [ ] Headline font matches (family, weight, size, tracking, leading)
- [ ] Body font matches
- [ ] Section labels: uppercase + tracking-widest + correct accent color
- [ ] No duplicate heading text (label ≠ headline)

**Colors:**
- [ ] Primary button color exact match
- [ ] Background gradient direction and color stops match
- [ ] Text hierarchy colors match (heading, body, muted, accent)

**Spacing:**
- [ ] Section padding matches (Stitch uses py-24 to py-32)
- [ ] Card padding matches (p-8 to p-10)
- [ ] Grid gaps match

**Components:**
- [ ] Buttons: rounded-full
- [ ] Cards: rounded-[2rem] or rounded-3xl with glass effect
- [ ] Icons: Material Symbols, not emoji or broken SVGs
- [ ] Shadows match (shadow-xl, shadow-2xl, colored shadows)

**Layout:**
- [ ] Max-width consistent across pages
- [ ] No gap between header and hero (unless intentional)
- [ ] Mobile responsive

**Interactions:**
- [ ] Hover: card shadow-2xl, icon scale-110
- [ ] Button hover: scale, shadow change
- [ ] Active: scale-95
- [ ] Transitions: duration-300 to duration-500

## PHASE 7: Graphic/Banner Generation (HTML → Image)

DeSi can also generate standalone graphics (Instagram posts, stories, Open Graph cards, etc.).

### Workflow — Stitch first, ALWAYS

**Step 1 is ALWAYS Stitch.** No matter the format — web page, Instagram post, logo, banner, OG card — you generate a Stitch screen first. This is non-negotiable.

1. **Generate with Stitch** — create a screen describing what you need (DESKTOP device type). Be detailed about the design: colors, typography, layout, content.

2. **Review the Stitch output** — download the HTML, read it, extract the design direction (colors, fonts, gradients, component styles).

3. **If Stitch output works at the required dimensions** — use it directly, crop/capture.

4. **If Stitch output doesn't fit** (viewport-dependent, wrong aspect ratio) — THEN and ONLY THEN write a self-contained HTML file using the design system extracted from the Stitch output:
   - Instagram square post: `1080x1080px`
   - Instagram story: `1080x1920px`
   - Open Graph card: `1200x630px`
   - Twitter card: `1200x675px`

3. **Use project assets** — always include the project's actual logo and brand assets. Reference them via `file://` paths or copy them to `/tmp` first.

4. **Capture with the DeSi capture script**:
   ```bash
   node ${CLAUDE_SKILL_DIR}/scripts/capture.mjs input.html output.png 1080 1080
   ```
   The script auto-installs Playwright in `/tmp` if missing. No manual setup needed.

5. **Verify** — read the output PNG, iterate if needed.

### HTML Template Rules for Graphics

- Set exact `width` and `height` on `body` (no responsive layout)
- Use `overflow: hidden` to prevent scrollbars
- Load fonts via `<link>` from Google Fonts CDN
- Use the project's design system (same fonts, colors, gradients from Stitch outputs)
- Include the project's actual logo/assets via `file://` or absolute paths
- Keep text large and readable (minimum 22px for body, 48px+ for headlines)
- All content must fit in a single viewport — no scroll

### Graphic Review & Iteration Loop

After capturing each graphic, **you MUST review it and iterate**. Save versioned files (v1, v2, v3...) so the user can compare revisions.

```
STEP 1 — Capture v1:
  node ${CLAUDE_SKILL_DIR}/scripts/capture.mjs banner.html output-v1.png 1080 1080

STEP 2 — Review the output (Read the PNG):
  Check ALL of these:
  □ Is the text readable at Instagram thumbnail size (~300px)?
  □ Is the logo/brand clearly visible?
  □ Are colors high-contrast enough for mobile screens?
  □ Is the layout balanced (no large empty areas)?
  □ Does the CTA stand out?
  □ Are all assets loading (logo, icons, fonts)?
  □ Does it look professional — not generic or AI-generated?

STEP 3 — List issues and fix them in the HTML

STEP 4 — Capture v2:
  node ${CLAUDE_SKILL_DIR}/scripts/capture.mjs banner.html output-v2.png 1080 1080

STEP 5 — Compare v1 vs v2:
  Read both PNGs, confirm improvements, check for regressions

STEP 6 — Repeat until satisfied, incrementing version numbers
```

**Keep ALL versions** in the output directory so the user can pick their preferred one. Name them clearly:
```
graphic/
├── ig-post-donate-v1.png
├── ig-post-donate-v2.png    # bigger text
├── ig-post-donate-v3.png    # final
├── ig-story-donate-v1.png
└── ig-story-donate-v2.png   # added logo
```

**Review criteria for each graphic type:**

| Format | Key checks |
|--------|-----------|
| Instagram post (1080x1080) | Text readable at 300px thumbnail, logo visible, CTA pops, balanced layout |
| Instagram story (1080x1920) | Top/bottom safe zones clear, large text (thumb-scrolling), swipe-up CTA visible |
| OG card (1200x630) | Title readable when small, brand visible, no critical content at edges (gets cropped) |

## Platform-Specific Social Media Guidelines

When generating social graphics, **always tailor content and visual style to the target platform**. What works on Instagram does NOT work on X/Twitter, and vice versa. Ask the user which platform(s) they're targeting before starting.

### Platform Quick Reference

| Platform | Primary format | Dimensions | Text density | Visual style | CTA approach |
|----------|---------------|------------|-------------|-------------|-------------|
| **Instagram Feed** | Square / 4:5 portrait | 1080x1080 / 1080x1350 | Minimal (< 30 words) | Polished, aesthetic, brand-heavy | "Link in bio", swipe carousel |
| **Instagram Story** | 9:16 vertical | 1080x1920 | Very minimal (< 15 words) | Full-bleed, immersive, stickers-style | Swipe-up / link sticker zone |
| **Instagram Carousel** | Square / 4:5 (multi-slide) | 1080x1080 / 1080x1350 | Medium (headline per slide) | Consistent style across slides, swipe-bait first slide | Last slide = CTA |
| **X / Twitter** | 16:9 landscape | 1200x675 | Medium (can be text-heavy) | Bold, high-contrast, meme-aware | Direct link in tweet, not in image |
| **LinkedIn** | 1.91:1 landscape | 1200x627 | Medium-high (professional) | Clean, corporate, data-driven | Professional tone, subtle CTA |
| **Facebook** | 1.91:1 landscape | 1200x630 | Medium | Warm, community-oriented | Clear button-style CTA |
| **Open Graph** | 1.91:1 landscape | 1200x630 | Headline + subtitle only | Brand-consistent, preview-optimized | N/A (auto-generated preview) |

### Instagram Guidelines

**Visual rules:**
- **Aesthetic first** — Instagram is a visual platform. The graphic must look like something people would save/share, not an ad.
- **Minimal text** — Instagram penalizes text-heavy images in algorithmic reach. Keep text under 20% of the image area.
- **No links in image** — URLs are useless on Instagram posts (not clickable). Use "Link in bio" or a clear visual indicator.
- **Brand consistency** — maintain a cohesive color palette and font across all posts (feed grid matters).
- **Safe zones** — profile pic overlaps bottom-left corner in feed; keep content centered.

**Content rules:**
- **Carousel > single image** for educational/informational content — first slide must hook, last slide must convert.
- **Story format** — design for thumb-scrolling speed. One idea per screen. Large text (60px+). Top 15% and bottom 20% are UI-occupied zones (username bar, reply bar).
- **Reels thumbnail** — if generating a cover, treat it as a 1080x1920 graphic with the core content in the center 1080x1080 area (top/bottom get cropped in grid view).

**Typography:**
- Headlines: 56-80px bold, high contrast
- Body: 28-36px, max 2-3 lines
- Use sans-serif fonts (Inter, Montserrat, Poppins) — editorial serifs for luxury/fashion only

### X / Twitter Guidelines

**Visual rules:**
- **Bold and punchy** — tweets move fast. The image must grab attention in <1 second while scrolling.
- **High contrast is king** — dark backgrounds with bright text or vice versa. Twitter's UI is visually noisy.
- **Text-heavy is OK** — unlike Instagram, Twitter images can carry substantial text (quotes, data, takes).
- **Meme-awareness** — Twitter is culturally irreverent. Overly polished/corporate graphics get ignored. Slightly raw > too slick.
- **No rounded corners** — Twitter crops images in feed; don't add border-radius that will clash with the platform's own rounding.

**Content rules:**
- **The tweet IS the caption** — the graphic should complement the tweet text, not duplicate it.
- **Screenshots style** — some of the most viral Twitter images look like screenshots, code snippets, or text on plain backgrounds. Don't over-design.
- **Thread graphics** — if part of a thread, each image should be self-contained with a clear numbering (1/5, 2/5...).
- **Quote-card format** — large quote text + author attribution works extremely well on X.

**Typography:**
- Headlines: 48-64px, heavy weight
- Body: 24-32px
- Monospace fonts work well for tech/data content
- White text on dark background is the power move on X

### LinkedIn Guidelines

**Visual rules:**
- **Professional but not boring** — clean layouts, data visualizations, charts, and infographics perform best.
- **Corporate color palettes** — blues, teals, dark grays. Avoid neon or playful colors unless the brand demands it.
- **Data-driven visuals** — LinkedIn audience loves stats, metrics, comparisons, and frameworks.
- **Document/carousel posts** — PDF carousels (each page = one slide) are LinkedIn's highest-engagement format.

**Content rules:**
- **More text is acceptable** — LinkedIn is a reading platform. Infographic-style dense content works.
- **Thought leadership framing** — "Here's what I learned", "3 things about X", "Framework for Y".
- **No hard sells** — LinkedIn penalizes overly promotional content. Educational > commercial.
- **Professional headshots/team photos** — human faces increase engagement significantly on LinkedIn.

**Typography:**
- Headlines: 44-56px, semi-bold
- Body: 22-28px, generous line height
- Serif fonts (Playfair Display, Lora) for thought leadership; sans-serif for data/tech

### Facebook Guidelines

**Visual rules:**
- **Warm and community-oriented** — Facebook skews personal/community. Graphics should feel inviting, not cold/corporate.
- **Faces and people** — photos with people outperform abstract graphics significantly on Facebook.
- **Text overlay rule** — Facebook (like Instagram) historically penalized images with >20% text. Keep text minimal.
- **Preview-friendly** — graphics appear at various sizes across feed, groups, and shares. Key content must be visible even at small sizes.

**Content rules:**
- **Event promotion** — Facebook excels at events. Event graphics should have: date, time, location, and one clear visual.
- **Community/group context** — if for a Facebook group, match the tone (casual, supportive, peer-to-peer).
- **Shareable format** — design for resharing: the graphic must make sense without the original caption.

**Typography:**
- Headlines: 48-64px, bold
- Body: 24-32px
- Friendly sans-serif fonts (Nunito, Open Sans, Lato)

### Open Graph / Link Preview Cards

**Visual rules:**
- **Readability at thumbnail size** — OG cards render as small as 300x157px in chat apps. Title must be legible at that size.
- **No critical content at edges** — different platforms crop OG images differently. Keep a 10% safe margin on all sides.
- **Brand logo always visible** — the OG card may be the first impression of your site. Logo placement is non-negotiable.
- **Avoid busy backgrounds** — text must pop against the background in every context (light/dark mode chat apps, social feeds).

**Content rules:**
- **Title + subtitle only** — OG cards don't need body text. One strong headline + one clarifying line.
- **Match the page content** — the OG image should preview what the user will see when they click. No bait-and-switch.

**Typography:**
- Title: 48-60px, bold, max 1 line
- Subtitle: 28-36px, regular weight, max 1 line
- High-contrast color combo (white on dark or dark on light — avoid mid-tones)

### Cross-Platform Adaptation Workflow

When the user asks for "social media graphics" without specifying a platform, generate for **all relevant platforms** in one batch:

```
1. Ask: "Which platforms? Instagram, X, LinkedIn, Facebook, OG card?"
2. If "all" — generate in this order (shared design system, adapted per platform):
   a. Instagram post (1080x1080) — visual anchor, defines the style
   b. Instagram story (1080x1920) — vertical adaptation
   c. X/Twitter card (1200x675) — bolder/punchier version
   d. LinkedIn banner (1200x627) — professional adaptation
   e. Facebook post (1200x630) — community-friendly adaptation
   f. OG card (1200x630) — minimal/preview version
3. Use SAME design system (colors, fonts, logo placement) across all
4. Adapt CONTENT DENSITY and TONE per platform
5. Name files clearly: {project}-{platform}-{variant}-v{N}.png
```

**Example file output:**
```
graphic/
├── campaign-ig-post-v1.png
├── campaign-ig-story-v1.png
├── campaign-x-card-v1.png
├── campaign-linkedin-v1.png
├── campaign-fb-post-v1.png
└── campaign-og-card-v1.png
```

## Tips

- **Stitch is async** — wait 15-30s after generation. Don't retry.
- **Material Symbols** need the font link in index.html or icons render as text.
- **browser-use** has no `navigate` command — use `open URL` instead.
- **Always build before comparing** to catch compilation errors.
- **Stitch uses stock photos** — match style and structure, use your own images.
- **Don't overfit** — adapt the design if it doesn't work with your actual content.

## Additional resources

- For the complete browser-use CLI reference, see [reference/browser-use.md](reference/browser-use.md)

$ARGUMENTS

---
> Source: [turinglabsorg/desi](https://github.com/turinglabsorg/desi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
