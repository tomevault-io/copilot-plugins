## design-md-generator

> Generate a comprehensive DESIGN.md design system document for web and mobile app projects, from project context, website URL, design screenshot, or by guiding a human step-by-step. Use this skill whenever the user asks for creating a design system, generating DESIGN.md, analyzing a website's visual design, extracting design elements from a screenshot or mockup, or producing visual/interaction specs for web or mobile apps — even if they don't explicitly use the word "DESIGN.md".


## Step 1: Input Detection & Mode Routing

Automatically detect the user's input type and route to the appropriate mode. **Do not ask the user which mode they want if the input type is unambiguous.**

### Input Type Decision Tree

```
User Input
├── URL (starts with http:// or https://)
│   └── → URL Analysis Mode (see "URL Analysis Mode" below)
├── Image file (uploaded .png/.jpg/.jpeg/.webp/.gif or provided path)
│   └── → Image Analysis Mode (see "Image Analysis Mode" below)
├── Existing project files (README.md, package.json, code files detected)
│   └── → Autonomous Mode (see "Autonomous Mode" below)
└── Text description only / no specific input
    └── → Guided Mode (see "Guided Mode" below)
```

### Ambiguous Input Handling

If the user provides **multiple types** (e.g., a URL + a screenshot), prioritize in this order:
1. **Image** (most specific visual source)
2. **URL** (live, interactive source)
3. **Project files** (contextual source)

Then ask:

> "I can work with multiple inputs here. Should I prioritize the [image/URL] for visual extraction, or the project files for context?"

---

## Mode Summaries

### Autonomous Mode

Analyze existing project files (README, manifest, styles) to infer design direction. Pick a design personality from the archetypes below based on product type and tech stack. **Avoid "AI flavor"** — do not default to Inter + `#3B82F6`. See "Autonomous Mode" section below for the full procedure.

### URL Analysis Mode

Extract colors, typography, components, layout, shadows, and border radius from a live website using WebFetch. Infer product type, audience, and emotional metaphor from the visual traits. Mark unextractable fields (deep narrative, "unreasonable choice", brand taboos) as `[MISSING]`. See "URL Analysis Mode" section below for the full extraction checklist.

### Image Analysis Mode

Analyze a design screenshot or mockup image. Extract color palette, font categories, component styles, layout density, depth/elevation, and overall atmosphere. Mark unidentifiable fields (exact font names, precise pixel values, transition timing) as `[MISSING]`. See "Image Analysis Mode" section below for the full visual analysis checklist.

### Guided Mode

Two sub-modes:
- **Full Interview** (standalone): Ask 8 structured questions one at a time (product, metaphor, unreasonable choice, platform, color, typography, density, motion).
- **Gap-Filling** (post-analysis): After URL/Image analysis, ask only the missing fields. Do not repeat already-extracted information.

See "Guided Mode" section below for the full question bank and gap-filling templates.

---

## Step 2: Data Sufficiency Check & Auto-Guided Gap-Filling

### Sufficiency Checklist

Before generating the final DESIGN.md, verify that all critical fields are filled (not placeholder, not `[MISSING]`):

| Priority | Check Item | Threshold |
|----------|------------|-----------|
| **P0** | Product name | Must have |
| **P0** | At least 1 primary color with hex | Must have |
| **P0** | Background color + text color | Must have |
| **P0** | Font family (heading + body) | Must have |
| **P0** | Emotional metaphor (2-3 sentences) | Must have |
| **P1** | "Unreasonable but right" choice | Must have |
| **P1** | Button style (padding, radius, colors) | Must have |
| **P1** | Card/container style | Must have |
| **P2** | Spacing system | Can infer |
| **P2** | Shadow system | Can infer |
| **P2** | Border radius scale | Can infer |
| **P2** | Breakpoints | Can infer |
| **P2** | Semantic colors | Can infer |
| **P2** | Do's and Don'ts | Can infer from metaphor |

### Decision Matrix

| Missing P0 | Missing P1 | Missing P2 | Action |
|------------|------------|------------|--------|
| 0 | 0 | 0-3 | **Generate immediately** with inferred P2 values |
| 0 | 1-2 | any | **Generate draft** + ask 1-2 targeted questions |
| 0 | 3+ | any | **Enter Gap-Filling** (see "Guided Mode" below) |
| 1+ | any | any | **Enter Gap-Filling**, must resolve P0 |

### Inference Rules for Missing Fields

When auto-generating values for insufficient data, follow these rules:

**If spacing system is missing:**
- Default to 4px base unit with scale: 4, 8, 12, 16, 24, 32, 48, 64, 96
- Adjust for density: dense products use 4/8/12/16; airy products use 8/16/24/32/48

**If shadow system is missing:**
- Vercel/Linear style (minimal shadows, borders instead) → use subtle 1px borders
- Material/Figma style → use layered shadows: `0 1px 3px rgba(0,0,0,0.1)`, `0 4px 12px rgba(0,0,0,0.08)`, `0 12px 32px rgba(0,0,0,0.12)`
- Apple/Notion style → use very subtle shadows or none

**If border radius is missing:**
- Engineer/developer tool → 4-6px (sharp, precise)
- Consumer/warm brand → 8-12px (friendly)
- Playful/creative → 16-24px (soft)
- Medical/finance → 2-4px (serious)

**If breakpoints are missing:**
- Default: Mobile 640px, Tablet 768px, Desktop 1024px, Wide 1280px
- Adjust for app-only projects to use device-specific breakpoints

**If semantic colors are missing:**
- Error: `#EF4444` (red-500) or darker `#DC2626`
- Warning: `#F59E0B` (amber-500)
- Success: `#10B981` (emerald-500) or `#22C55E`
- Info: `#3B82F6` (blue-500)
- **Exception**: If the brand color is red/orange/green/blue, shift semantic colors to avoid collision:
  - Red brand → Error becomes `#991B1B` (darker), or use `#E11D48` (rose)
  - Blue brand → Info becomes `#6366F1` (indigo)

---

## Step 3: Generate DESIGN.md

Use the Master Template below to fill the DESIGN.md. Fill every section with **specific values** (hex codes, px/rem values, font names). Never leave placeholders.

Key principles while filling:
- Inject at least one "unreasonable but right" choice (see Anti-Pattern Checklist)
- Include a specific emotional metaphor in Chapter 1
- Use exact values, no placeholders like "[value]" in final output

---

## Step 4: Anti-Pattern Checklist

Before finalizing the DESIGN.md, run this checklist. If any item is true, revise.

- **AI Flavor Check**: Does it default to Inter + `#3B82F6` + "clean modern"? If yes, change at least one.
- **Narrative Check**: Is there a specific emotional metaphor in Chapter 1? If no, add one.
- **Unreasonable Choice Check**: Is there at least one design rule that breaks industry convention? If no, add one.
- **Specificity Check**: Are all colors, sizes, and fonts given as exact values? No placeholders allowed.
- **App Gesture Check**: If mobile is included, are gestures mapped to physical metaphors?
- **Do's and Don'ts Check**: Are there at least 3 items in each list, and are they actionable?
- **URL Extraction Check** (if URL mode): Did you verify extracted colors against the actual rendered page? CSS variables may differ from computed styles.
- **Image Analysis Check** (if Image mode): Did you double-check inferred hex values against the image? Screen brightness and compression can shift perceived colors.
- **Semantic Collision Check**: Do semantic colors (error/warning/success/info) clash with brand colors? If brand is blue, info should be shifted to indigo.

---

## Step 5: Output Generation

By default, generate **all 4 files** in a `[project-root]/design-system/` directory (or project root if user prefers). Do not ask permission for each file — generate them as a complete package.

### Mode-Specific Notes

**Autonomous Mode:**
- Write to `[project-root]/design-system/`
- Use project context to name the design system

**URL Analysis Mode:**
- Write to `[project-root]/design-system/` or a directory named after the extracted brand/URL domain
- Prefix the design system name with the domain (e.g., "Design System: example.com Inspired")
- Include a note: "Based on visual analysis of [URL] as of [date]"

**Image Analysis Mode:**
- Write to `[project-root]/design-system/` or a directory named after the image file (without extension)
- Prefix with the inferred brand or "Unnamed Project"
- Include a note: "Based on visual analysis of [image filename]"

**Guided Mode:**
- Write to `[project-root]/design-system/`
- Use the product name provided by the user

### Output Files

1. **`DESIGN.md`** — Use the Master Template (+ Mobile Extension if applicable)
2. **`README.md`** — Brand-facing introduction. See "Output File Specifications" below for the template.
3. **`preview.html`** — Light mode visual catalog with color swatches, typography scale, button gallery, card/input examples, spacing scale, and micro-interactions. See "Output File Specifications" below for detailed requirements.
4. **`preview-dark.html`** — Dark mode variant. See "Output File Specifications" below for requirements.

Optional File 5: **Native Preview Snippets** (`Preview.swift` / `Preview.kt`) if building a native app. See "Output File Specifications" below.

### Output Instructions

1. Write all files to the designated directory.
2. Inform the user where they were saved and what mode was used.
3. Summarize the "unreasonable choice" and the emotional metaphor in one sentence.
4. Offer to generate additional files: `AGENTS.md`, `tailwind.config.js`, `tokens.json`, Figma variables JSON.

For URL/Image modes, also provide:
- **Extraction Summary**: What was extracted vs. inferred vs. asked
- **Confidence Rating**: High / Medium / Low per section
- **Suggested Iterations**: 2-3 follow-up prompts to refine the design system

See "Output File Specifications" below for full details.

---

## Personality Archetypes (Reference)

Use these as starting points, but always customize.

| Archetype | Emotional Metaphor | Typical Colors | Typical Typography | Representative Brand |
|-----------|-------------------|----------------|-------------------|---------------------|
| **Dark Engineer** | A high-end recording studio at midnight | Near-black bg, single saturated accent | Geometric sans, tight tracking | Linear, Vercel |
| **Warm Salon** | A letter from a thoughtful friend | Parchment, terracotta, warm grays | Serif headings, sans UI | Claude |
| **Minimalist Gallery** | A white cube art space | Pure white, subtle warm gray, one earthy accent | Humanist sans | Apple, Notion |
| **Vibrant Workshop** | A creative studio with paint everywhere | Bold saturated palette, high contrast | Expressive sans or mixed | Figma, Framer |
| **Deep Terminal** | A spaceship control panel | Carbon black, neon green/blue accent | Monospace + geometric sans | VoltAgent, Warp |

---

# Mode Instructions

Reference section for the detailed instructions of each generation mode. Read the section corresponding to the mode selected in Step 1.

---

## Autonomous Mode

1. **Inspect project context**:
  - Read `README.md`, `package.json`, `Cargo.toml`, or equivalent manifest files.
  - Look for any existing UI screenshots, Figma links, or style files (`globals.css`, `theme.ts`, `tailwind.config.js`).
  - Identify the product type: SaaS dashboard, AI chat, e-commerce, developer tool, content app, etc.
2. **Infer design direction**:
  - Based on the product type and tech stack, pick a "design personality" (see Personality Archetypes above).
  - If the project is B2B developer tool → lean toward dark engineering aesthetic (Linear/Vercel inspired).
  - If the project is consumer AI companion → lean toward warm literary salon (Claude inspired).
  - If the project is creative tool → lean toward vibrant, expressive system (Figma/Framer inspired).
3. **Generate DESIGN.md** using the **Master Template** below.
  - Fill in all sections with specific hex colors, font families, spacing scales, and component rules.
  - **Avoid "AI flavor"**: Do not default to Inter + `#3B82F6` + "clean modern interface". Inject at least one "unreasonable but right" choice.
4. **Execute Data Sufficiency Check** (Step 2 above):
  - If critical fields are missing, enter **Guided Gap-Filling** before final generation.
5. **Output**: Follow Step 5 above.

---

## URL Analysis Mode

Use this mode when the user provides a website URL. Extract the design system directly from the live site.

### Fetch Website Content

Use WebFetch to retrieve the target URL. Fetch both:
- The main page HTML (for structure and content)
- The rendered visual output (for accurate color and layout analysis)

If the site uses client-side rendering (React/Vue/SPA), note that some dynamic styles may not be fully captured. In that case, infer missing values from visible elements.

### Extract Visual Elements

Analyze the fetched content systematically. Document your findings in a structured extraction note.

**Color Extraction:**
- **Primary/Brand**: Extract from `--primary`, `--brand`, `--color-primary` CSS variables; CTA button backgrounds; active navigation links.
- **Accent**: Extract from secondary CTAs, hover states, highlight badges, links.
- **Background**: Extract from `body`, `main`, `.container`, card backgrounds. Note both light and dark mode if detected.
- **Text**: Extract from `body`, `h1-h6`, `.text-primary`, `.text-secondary`.
- **Surface/Border**: Extract from card backgrounds, input borders, dividers.
- **Semantic**: Extract from error messages (red), success states (green), warnings (yellow/amber), info (blue).
- **Neutrals**: Extract gray scale from text, borders, disabled states.

**Typography Extraction:**
- **Heading font**: From `h1-h6` `font-family`, weight, and size hierarchy.
- **Body font**: From `body`/`p` `font-family`, size, line-height.
- **Code font**: From `code`, `pre` elements.
- **Hierarchy**: Document the actual pixel/rem sizes for H1, H2, H3, Body, Caption as used on the site.
- **Letter-spacing**: Note any tight or loose tracking (e.g., uppercase labels with wide tracking).

**Component Style Extraction:**
- **Buttons**: padding, border-radius, background color, text color, hover/active states, font-weight.
- **Cards/Containers**: background, border (width, color, style), border-radius, box-shadow, padding.
- **Inputs/Forms**: background, border, focus ring style, border-radius, padding, placeholder color.
- **Navigation**: background color, link color, active/hover link treatment, height.
- **Links**: default color, hover color, underline behavior.
- **Badges/Tags**: background, text color, border-radius, padding.

**Layout Extraction:**
- **Container max-width**: From `.container`, `.wrapper`, `.max-w-*` classes.
- **Grid system**: Columns, gutters, gap values.
- **Spacing**: Base unit and scale (e.g., 4px, 8px, 16px, 24px, 32px, 48px, 64px).
- **Breakpoints**: From `@media` queries in CSS.

**Shadow & Depth Extraction:**
- **Box-shadow values**: From cards, dropdowns, modals, buttons.
- **Depth philosophy**: Does the site use shadows for elevation (Material) or borders for structure (Vercel/Linear)?

**Border Radius Extraction:**
- Small (inputs, badges)
- Medium (cards, buttons)
- Large (modals, featured sections)
- Full (pills, avatars)

### Infer Design Personality

Based on extracted elements, infer:
- **Product type**: SaaS, e-commerce, developer tool, content platform, social app, etc.
- **Target audience**: Developers, consumers, enterprise, creatives.
- **Emotional metaphor**: Map visual traits to a place/feeling:
  - Dark + neon accents → "spaceship control panel" (Deep Terminal)
  - Warm + serif fonts → "a letter from a thoughtful friend" (Warm Salon)
  - White + minimal → "white cube art gallery" (Minimalist Gallery)
  - Bold + saturated → "creative studio with paint everywhere" (Vibrant Workshop)
  - Near-black + single accent → "high-end recording studio at midnight" (Dark Engineer)
- **Density preference**: Information-dense (Bloomberg Terminal) vs. breathable (Aesop website).
- **Motion language**: From any visible animations or transitions in CSS.

### Fill the Master Template

Populate the Master Template below with extracted values. Use `[MISSING: description]` placeholders for fields that cannot be extracted.

**Common unextractable fields (mark as MISSING):**
- Deep emotional narrative in Chapter 1 (beyond surface-level visual inference)
- The "unreasonable but right" choice (requires user intent)
- Specific brand-origin color meanings (e.g., "studio wall color")
- Mobile-specific gesture mappings (unless the URL is a mobile app web view)
- Explicit Do's and Don'ts rooted in brand values

### Execute Data Sufficiency Check

Run Step 2 above. If gaps are found, enter **Guided Gap-Filling** before generating the final DESIGN.md.

---

## Image Analysis Mode

Use this mode when the user provides an image file (screenshot, Figma export, design mockup, mood board).

### Read the Image

Use the Read tool to load the image file. The image may be:
- A full-page website screenshot
- A mobile app screen
- A Figma/Sketch design export
- A mood board or style reference
- A partial component collage

### Visual Analysis Checklist

Analyze the image comprehensively. Document findings in a structured extraction note.

**Color Palette Analysis:**
- **Dominant color**: The most visually prominent color (usually brand/primary).
- **Background colors**: Page background, card surfaces, section backgrounds.
- **Text colors**: Primary text, secondary text (labels, metadata), inverted text on dark backgrounds.
- **Accent colors**: CTA buttons, links, highlights, status indicators.
- **Neutral palette**: Grays used for borders, dividers, disabled states.
- **Dark mode colors**: If the image shows dark mode, document those separately.
- For each color, provide the hex value if clearly identifiable, or a close approximation.

**Typography Analysis:**
- **Font style category**:
  - Sans-serif (geometric, humanist, grotesque)
  - Serif (modern, transitional, slab)
  - Monospace
  - Display/experimental
- **Heading characteristics**: Size relative to body, weight (light/regular/bold), letter-spacing, casing (sentence/uppercase/title).
- **Body characteristics**: Size, weight, line-height perception (tight/normal/loose).
- **Hierarchy levels**: Count distinct text sizes visible (e.g., 5 levels: hero, heading, subheading, body, caption).
- **Code/technical text**: If present, note the style.

**Component Recognition:**
- **Buttons**: Identify all button styles (primary, secondary, ghost, danger). Note shape (pill, rounded, square), shadow, and size.
- **Cards/Containers**: Note background treatment (solid, translucent, gradient), border, shadow, corner radius.
- **Input fields**: Border style, focus indication, background, placeholder visibility.
- **Navigation**: Top bar, sidebar, tab bar, bottom nav — note style, height, background treatment.
- **Tags/Badges/Chips**: Small label elements, note shape, color, and size.
- **Tables/Lists**: If present, note row separators, hover states, selection states.
- **Modals/Overlays**: Note backdrop treatment, shadow, and border.

**Layout & Spacing Analysis:**
- **Density**: Crowded vs. airy. Count approximate elements per screen area.
- **Alignment**: Left-aligned, center-aligned, or mixed.
- **Whitespace**: Generous margins/padding vs. edge-to-edge content.
- **Grid**: Visible columns, card arrangements, or asymmetric layouts.
- **Safe area**: If mobile, note top/bottom spacing for system bars.

**Depth & Elevation:**
- **Shadow usage**: Soft diffused shadows, sharp drop shadows, or no shadows (flat).
- **Border usage**: 1px hairlines, thick borders, or borderless.
- **Layering**: Overlapping elements, floating action buttons, sticky headers.

**Motion & Interaction Clues (if visible):**
- Hover states captured in the image (e.g., button with different background).
- Active/selected states (tab underlines, list selections).
- Loading states or skeletons if visible.

**Overall Atmosphere:**
- Describe the mood in 3-5 adjectives (e.g., "clinical, precise, premium, subdued").
- Match to the closest **Personality Archetype** (see above).
- Note any reference brand the design resembles.

### Fill the Master Template

Same as URL Analysis. Use `[MISSING: description]` for unidentifiable fields.

**Common unextractable fields from images:**
- Exact font names (unless clearly identifiable from letterforms)
- Exact pixel values for spacing (estimate with "approx. Xpx")
- Hover/active transition timing and easing curves
- Responsive breakpoint behavior
- Deep brand narrative and emotional metaphor
- "Unreasonable but right" design choices
- Brand constraints and taboos

### Execute Data Sufficiency Check

Run Step 2 above. If gaps are found, enter **Guided Gap-Filling**.

---

## Guided Mode

### Full Interview (standalone mode)

Use this when the user starts with no specific input, or explicitly asks for guided mode.

Conduct a structured interview. Ask **one question at a time** and build upon previous answers.

#### Questionnaire (ask in order)

**Q1. Product & Audience**

> "用一句話描述你的產品是什麼，以及核心用戶是誰？"
> (e.g., "A note-taking app for nighttime programmers")

**Q2. Emotional Metaphor**

> 「如果你的產品是一個『場所』，它會是什麼？請用具體的類比描述氛圍。」
> (e.g., "a 24-hour convenience store at 3 AM — lonely but warm, with neon glow")

**Q3. One Unreasonable Choice**

> 「市面上同類產品都在用什麼設計慣例？你想故意打破哪一條？」
> (e.g., "All dev tools are dark mode. I want mine to be aggressively light.")

**Q4. Platform Scope**

> 「這個設計系統要覆蓋哪些平台？」
>
> - Web only
> - Mobile App only (iOS/Android)
> - Both Web + Mobile App

**Q5. Color Constraint**

> 「有沒有一個對你有特殊意義的顏色？（如工作室牆面顏色、家鄉土壤顏色）如果沒有，給我一個情緒詞，我來幫你轉譯。」

**Q6. Typography Personality**

> 「你的產品『說話』的聲音是什麼樣的？選一個最接近的：
>
> - 冷靜的工程師（等寬/幾何無襯線）
> - 溫暖的編輯（襯線體）
> - 前衛的藝術家（實驗字體）
> - 可靠的管家（經典人文主義無襯線）」

**Q7. Density Preference**

> 「介面應該感覺『擁擠但強大』還是『寬鬆但從容』？」
> (Information-dense like Bloomberg Terminal vs. breathable like Aesop website)

**Q8. Motion & Interaction**

> 「當用戶完成一個重要操作時，你希望產品給出什麼樣的反饋？選一個：
>
> - 精確的機械咔噠聲
> - 溫柔的彈性回饋
> - 華麗的慶祝動效
> - 幾乎無聲的沈默確認」

After Q8, synthesize all answers and generate the `DESIGN.md` using the Master Template below.

### Guided Gap-Filling (post-analysis mode)

Use this after URL or Image analysis when the Data Sufficiency Check detects missing fields. **Do not repeat questions for fields already extracted.** Only ask about missing information.

**Opening statement:**

> "我已經從 [URL/圖片] 分析了你的設計方向。以下是提取到的關鍵元素：
>
> - 主色：[顏色]（#[hex]）
> - 字體風格：[推斷的字體類型]
> - 整體氛圍：[推斷的氣氛描述]
> - 產品類型：[推斷的類型]
>
> 為了讓設計系統更完整，我還需要確認幾個細節："

**Missing-field Question Bank** (ask only for fields marked MISSING):

**If Product Name/Audience is missing:**
> "這個設計是為什麼產品服務的？核心用戶是誰？"

**If Emotional Metaphor is missing or weak:**
> "從 [提取的顏色/字體/布局] 來看，這個設計給人的感覺是 [推斷的情感]。你會怎麼用具體的場所或場景來描述這個產品的氛圍？"

**If "Unreasonable Choice" is missing:**
> 「市面上同類產品通常 [行業慣例觀察]。你想故意打破哪一條規則來讓自己的產品與眾不同？」

**If Color Meaning is missing:**
> "主色 [hex] 對你的品牌有什麼特殊意義嗎？（如來自 logo、團隊文化、或純粹的美學選擇）"

**If Typography is unconfirmed:**
> "從圖片中我推斷標題使用 [推斷字體類型]，正文使用 [推斷字體類型]。這個判斷正確嗎？有沒有指定的字體名稱？"

**If Brand Taboos/Constraints are unknown:**
> "有沒有什麼顏色、字體或設計元素是這個品牌絕對不能用的？"

**If Mobile App scope is unclear:**
> 「這個設計系統需要覆蓋手機 App（iOS/Android）嗎？還是僅限 Web？」

**If Density is unclear:**
> 「你希望介面感覺『資訊密集但強大』還是『寬鬆但從容』？」

After collecting answers, fill the remaining `[MISSING]` placeholders and proceed to generation.

---

# Master Template

Use this exact structure when generating DESIGN.md. Fill every section with **specific values** (hex codes, px/rem values, font names). Never leave placeholders.

```markdown
# Design System: [Product Name]

## 1. Visual Theme & Atmosphere

[2-3 paragraphs describing the emotional and philosophical core of the design.]

**Key Characteristics:**
- [Distinctive trait 1 with specific value]
- [Distinctive trait 2 with specific value]
- [Distinctive trait 3 with specific value]
- [At least one "unreasonable but right" choice]

## 2. Color Palette & Roles

### Primary
- **[Color Name]** (`#HEXCODE`): [Role and emotional meaning]

### Secondary & Accent
- **[Color Name]** (`#HEXCODE`): [Role]

### Surface & Background
- **[Color Name]** (`#HEXCODE`): [Role]

### Neutrals & Text
- **[Color Name]** (`#HEXCODE`): [Role]

### Semantic Colors (Error, Warning, Success, Info)
- **Error** (`#HEXCODE`)
- **Warning** (`#HEXCODE`)
- **Success** (`#HEXCODE`)
- **Info** (`#HEXCODE`)

## 3. Typography Rules

### Font Family
- **Headings**: [Font name, fallback stack]
- **Body**: [Font name, fallback stack]
- **Code/Mono**: [Font name, fallback stack]

### Hierarchy
| Level | Size | Weight | Line-Height | Letter-Spacing | Use Case |
|-------|------|--------|-------------|----------------|----------|
| Display | [px/rem] | [weight] | [ratio] | [px] | Hero headlines |
| H1 | [px/rem] | [weight] | [ratio] | [px] | Page titles |
| H2 | [px/rem] | [weight] | [ratio] | [px] | Section headers |
| H3 | [px/rem] | [weight] | [ratio] | [px] | Sub-sections |
| Body | [px/rem] | [weight] | [ratio] | [px] | Paragraphs |
| Caption | [px/rem] | [weight] | [ratio] | [px] | Labels, metadata |
| Code | [px/rem] | [weight] | [ratio] | [px] | Inline code |

### Principles
- [Principle 1]
- [Principle 2]
- [Principle 3]

## 4. Component Stylings

### Buttons

**Primary Button**
- Background: `[color]` (`#HEX`)
- Text: `[color]` (`#HEX`)
- Padding: `[value]`
- Radius: `[value]`
- Hover/Active: [specific state rules]

**Secondary Button**
- Background: `[color]` (`#HEX`)
- Text: `[color]` (`#HEX`)
- Padding: `[value]`
- Radius: `[value]`
- Hover/Active: [specific state rules]

**Danger Button**
- Background: `[color]` (`#HEX`)
- Text: `[color]` (`#HEX`)
- Padding: `[value]`
- Radius: `[value]`

### Cards & Containers
- Background: `[color]` (`#HEX`)
- Border: `[value] solid [color]`
- Radius: `[value]`
- Shadow: `[shadow values]`

### Inputs & Forms
- Background: `[color]`
- Text: `[color]`
- Border: `[value] solid [color]`
- Focus ring: `[value] [color]`
- Radius: `[value]`
- Padding: `[value]`

### Navigation
- Background: `[color]`
- Link color: `[color]`
- Active link: `[color]`
- CTA treatment: [button style reference]

### Links
- Default: `[color]`
- Hover: `[color]`
- Visited: `[color]` (if applicable)

### Distinctive Components
[Describe 1-3 components unique to this product.]

## 5. Layout Principles

### Spacing System
- Base unit: `[px/rem]`
- Scale: `[list of values]`

### Grid & Container
- Max container width: `[value]`
- Gutter: `[value]`
- Columns: `[number]`

### Whitespace Philosophy
[1-2 sentences on how space is used meaningfully.]

### Border Radius Scale
- Small: `[value]` — inputs, badges
- Medium: `[value]` — cards, buttons
- Large: `[value]` — modals, featured sections
- Full: `[value]` — pills, avatars

## 6. Depth & Elevation

### Shadow System
| Level | Shadow Value | Use Case |
|-------|--------------|----------|
| Flat | `none` | Base surfaces |
| Subtle | `[value]` | Cards, inputs |
| Elevated | `[value]` | Dropdowns, popovers |
| Floating | `[value]` | Modals, toasts |

### Elevation Philosophy
[Describe whether the product uses shadows for depth, borders for structure, or a hybrid approach like Vercel's "shadow-as-border" technique.]

## 7. Do's and Don'ts

### Do
- [Specific, actionable rule 1]
- [Specific, actionable rule 2]
- [Specific, actionable rule 3]
- [At least one rule that enforces the "unreasonable choice"]

### Don't
- [Specific anti-pattern 1]
- [Specific anti-pattern 2]
- [Specific anti-pattern 3]
- [At least one rule that explicitly forbids a common cliché]

## 8. Responsive Behavior

### Breakpoints
- Mobile: `[px]`
- Tablet: `[px]`
- Desktop: `[px]`
- Wide: `[px]`

### Touch Targets
- Web minimum: `[px] x [px]`
- App minimum (iOS): `44px x 44px`
- App minimum (Android): `48px x 48px`

### Collapsing Strategy
- Navigation: [how it transforms]
- Grids: [how columns collapse]
- Typography: [how sizes scale]
- Spacing: [how padding adjusts]

### Image Behavior
[How media scales, aspect ratios, art direction if any.]

## 9. Agent Prompt Guide

### Quick Color Reference
- Brand CTA: "[Color Name] (#HEX)"
- Page Background: "[Color Name] (#HEX)"
- Card Surface: "[Color Name] (#HEX)"
- Primary Text: "[Color Name] (#HEX)"
- Secondary Text: "[Color Name] (#HEX)"
- Border: "[Color Name] (#HEX)"

### Example Component Prompts
- "Create a hero section on [Background] (#HEX) with a [size] [font] headline..."
- "Design a feature card on [Surface] (#HEX) with a [border] border and [radius] radius..."
- "Build a [state] button in [color] (#HEX) with [text color] text..."

### Iteration Guide
1. Focus on ONE component at a time
2. Reference specific color names with hex values
3. Always specify the background surface for context
4. Use exact typography specs (font, size, weight, line-height)
5. Follow the Do's and Don'ts strictly
```

---

## Mobile App Extension

If the project includes **mobile app**, append these three chapters to the DESIGN.md **after** Chapter 9.

```markdown
## 10. Platform Adaptation

### iOS
- Safe Area compliance: all content respects top/bottom safe area insets
- System font: SF Pro (do not use Android fonts on iOS)
- Navigation pattern: [Tab Bar / Navigation Stack / Modal Sheets]
- Haptic feedback: [light / medium / heavy] for [specific actions]

### Android
- System font: Roboto / Noto Sans CJK
- Navigation pattern: [Bottom Navigation / Navigation Drawer / Floating Action Button]
- Dynamic color: [enabled / disabled]
- Back button behavior: [specific rule]

### Cross-Platform Taboos
- Never use hamburger menu as primary navigation on iOS
- Never use iOS-style back chevrons on Android
- Never use pure black `#000000` as dark mode background (use `#0f0f11` or `#121212`)

## 11. Gesture Semantics

| Intent | Gesture | Physical Metaphor |
|--------|---------|-------------------|
| Delete | Swipe left + haptic | Tearing a paper note |
| Reorder | Long-press + drag | Picking up a Lego block |
| Go back | Edge swipe right | Turning a book page |
| Refresh | Pull down + elastic snap | Pulling a rubber band |
| Quick actions | Long-press bubble | Peeling a sticker |
| Dismiss modal | Swipe down | Sliding a card off the table |

## 12. System Component Mapping

| Web Component | iOS Native | Android Native |
|---------------|------------|----------------|
| Primary Button | UIButton (filled) | Material Button (filled) |
| Modal Dialog | UIAlertController / UISheetPresentationController | AlertDialog / BottomSheet |
| Dropdown | UIPickerView / UIActionSheet | Exposed Dropdown Menu |
| Toast / Snackbar | Custom UIKit overlay | Snackbar |
| Toggle | UISwitch | Material Switch |
| Text Field | UITextField | TextInputLayout |
| Card | UICollectionViewCell | Material Card |
```

---

# Output File Specifications

Reference section for the detailed specifications of each output file in the design system package. Read this when you are ready to generate the output files (Step 5).

## File 1: DESIGN.md

Use the Master Template above (+ Mobile App Extension if applicable).

---

## File 2: README.md (Brand Introduction)

A concise brand-facing document that introduces the design system without technical implementation details.

```markdown
# [Product Name] Design System

## Design Philosophy

[1-paragraph summary of the emotional metaphor and core idea from DESIGN.md Chapter 1.]

## Key Principles

- **[Principle 1]**: [One-sentence explanation]
- **[Principle 2]**: [One-sentence explanation]
- **[Principle 3]**: [One-sentence explanation]

## Color Essence

| Role | Color | Hex | Meaning |
|------|-------|-----|---------|
| Primary | [Name] | `#HEX` | [Emotional meaning] |
| Background | [Name] | `#HEX` | [Atmosphere role] |
| Accent | [Name] | `#HEX` | [Highlight role] |

## Typography Voice

- **Headings**: [Font] — [personality description]
- **Body**: [Font] — [personality description]

## What's Included

- `DESIGN.md` — Complete design system specification for AI agents and developers
- `preview.html` — Light mode visual component catalog
- `preview-dark.html` — Dark mode visual component catalog

## Usage

Drop `DESIGN.md` into your project root and reference it when prompting AI coding agents.
```

---

## File 3: preview.html (Light Mode Visual Catalog)

A self-contained HTML page that renders the design system as a visual catalog. Must include:

1. **Color Swatches**: All primary, secondary, surface, neutral, and semantic colors with hex labels
2. **Typography Scale**: Display, H1, H2, H3, Body, Caption samples with actual font stack
3. **Button Gallery**: Primary, Secondary, Danger, Disabled states
4. **Card Example**: A sample card using the actual shadow, border, radius, and background values
5. **Input Example**: Form input with focus state
6. **Spacing Scale**: Visual bars showing spacing increments
7. **Micro-Interactions Section**: A dedicated area demonstrating motion language

### Micro-Interactions Requirements

The preview must include live, interactive examples of:

- **Button States**: Hover transitions (scale 1.02, shadow lift, color shift), active press (scale 0.98), and loading spinner state
- **Input Focus**: Soft glow/focus ring fade-in (200-300ms ease-out), placeholder fade behavior
- **Card Hover**: Elevation lift with shadow deepening and subtle Y-axis translate (-2px to -4px)
- **Toggle/Switch**: Elastic spring animation when toggled (use CSS `transition` with `cubic-bezier(0.34, 1.56, 0.64, 1)`)
- **Feedback Pulse**: A success/checkmark micro-animation or gentle pulse on a CTA element
- **Stagger Reveal**: If showing a list or group, optional stagger fade-in on load (0.1s delay increments)

### App Simulation Mode (if project includes mobile app)

Within the same `preview.html`, add a **"Mobile Simulation"** section that mimics app interactions using CSS/JS:

- **Phone Frame Container**: A ~375px x 812px rounded container representing an iPhone/Android screen
- **Touch Target Rings**: Visual 44px/48px radius circles appearing briefly on tap to demonstrate compliance
- **Gesture Hints**: Swipe arrows and instructional overlays for edge-swipe-back, pull-to-refresh, long-press menus
- **Bottom Navigation Bar**: Rendered inside the phone frame using the app-safe-area bottom padding
- **Haptic Visualizer**: A small ripple or screen flash on tap to simulate haptic feedback visually
- **Page Transition Simulation**: A push/pop slide animation between two dummy screens when tapping navigation items

Technical requirements:

- Single-file, no external JS dependencies (pure CSS transitions + optional lightweight inline JS for click states)
- Use CSS custom properties (variables) for colors to keep `DESIGN.md` and `preview.html` in sync
- Embed all CSS in `<style>` and any JS in `<script>`
- Use the exact hex colors, fonts, easing curves, and spacing from `DESIGN.md`
- Load web fonts via Google Fonts or jsDelivr CDN if needed
- Responsive container (~900px max-width, centered)

---

## File 4: preview-dark.html (Dark Mode Visual Catalog)

Same structure as `preview.html`, but:

- Background uses the dark mode surface color from `DESIGN.md`
- Text uses the corresponding dark mode text colors
- Components render their dark variants (dark cards, dark inputs, dark buttons)
- Micro-interactions adapt to dark mode (e.g., focus glow should use the accent color with lower opacity, shadows may become inner-glows or border-glows)
- App simulation phone frame uses dark mode surfaces and OLED-safe background colors
- If `DESIGN.md` does not define explicit dark mode colors, derive them by inverting the background/text pairs and keeping accent colors unchanged

---

## File 5: Native Preview Snippets (Optional, if project includes mobile app)

If the user confirms they are building a native app, also generate lightweight preview snippets that developers can paste into Xcode or Android Studio to see real native micro-interactions:

**`Preview.swift`** (SwiftUI Preview for iOS)

```swift
import SwiftUI

struct DesignSystemPreview: View {
    var body: some View {
        // Renders primary button, card, toggle, and input
        // with the exact colors and corner radius from DESIGN.md
    }
}

#Preview {
    DesignSystemPreview()
}
```

**`Preview.kt`** (Jetpack Compose Preview for Android)

```kotlin
@Preview
@Composable
fun DesignSystemPreview() {
    // Renders primary button, card, switch, and text field
    // with the exact colors and shapes from DESIGN.md
}
```

Requirements:

- Use hardcoded hex values from `DESIGN.md` (do not use Material dynamic theming unless explicitly requested)
- Include a basic `@State` toggle demo to show the switch/check animation
- Include a `Button` with ripple enabled/disabled per the design system's preference
- Show a `Card` with the exact elevation/shadow values

Note: Native previews are optional quick-starts, not production code. They help bridge the gap between `DESIGN.md` specs and actual platform implementation.

---

## URL/Image Mode Additional Output

When generating from a URL or image, also provide:

1. **Extraction Summary**: A brief summary of what was extracted vs. what was inferred:
  > "From [URL/image], I extracted: primary color `#3B82F6`, heading font 'Inter', button radius 8px. I inferred: spacing scale (4px base), shadow system (subtle layered). I asked you about: emotional metaphor, unreasonable choice."

2. **Confidence Rating**: For each major section, indicate confidence:
  - **High**: Directly extracted from the source (e.g., exact hex from CSS)
  - **Medium**: Inferred from visual analysis (e.g., font category from letterforms)
  - **Low**: Defaulted or heavily estimated (e.g., motion language from static image)

3. **Suggested Iterations**: List 2-3 specific follow-up prompts the user can use to refine the design system:
  > "To refine: try 'make the buttons more rounded' or 'add a secondary accent color in coral'"

---
> Source: [JobYu/design-md-generator](https://github.com/JobYu/design-md-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
