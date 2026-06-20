## gemini-image-skill

> Generate images using Gemini CLI with the nanobanana extension. Supports icons, patterns, diagrams, illustrations, UI assets, image editing, and presentation slide decks. Use when user needs visuals, backgrounds, app icons, hero images, patterns, flowcharts, presentation slides, or wants to edit/restore existing images.


# Google Image Generation Skill

Generate high-quality images using Gemini CLI with the **nanobanana extension** and **Nano Banana Pro** best practices.

---

## Structured Prompt Schema

For best results with Nano Banana Pro, use this structured schema instead of keyword lists:

| Field | Purpose | Example |
|-------|---------|---------|
| **SUBJECT** | Main focus | "cyberpunk hacker in a hood" |
| **CONTEXT** | Environment, mood | "dimly lit server room, tense atmosphere" |
| **COMPOSITION** | Framing, camera | "medium shot, 3/4 angle, subject centered" |
| **STYLE** | Art style, medium | "photorealistic 3D render, cinematic" |
| **LIGHTING** | Light source, palette | "neon blue backlighting, warm orange accents" |
| **TEXT** | Exact words, placement | "headline 'SECURE' top-center, sans-serif bold" |
| **TECHNICAL** | Resolution, aspect | "16:9 widescreen, high detail" |
| **CONSTRAINTS** | What to avoid | "no watermarks, no distorted hands" |

### Schema Example
```bash
~/.claude/skills/google-image/google-img '/generate "SUBJECT: minimalist chat bubble icon. STYLE: modern flat design, iOS aesthetic. LIGHTING: soft ambient, clean white background. TECHNICAL: square 1:1, crisp at 512px. CONSTRAINTS: no text, single blue accent (#0066FF), no shadows"'
```

---

## Three Modes: Choose Based on Task

### Mode A: Production Assets
**For:** Logos, banners, UI elements, thumbnails, marketing materials

- Use **FULL schema** (all 8 fields)
- Emphasize: exact text, brand colors, clean layout
- Keywords: "consistent design system", "grid-based", "print-safe typography"

```bash
# Production mode example
~/.claude/skills/google-image/google-img '/generate "SUBJECT: PlotCraft logo with quill pen. STYLE: modern flat, vector-style. LIGHTING: none (flat design). TEXT: wordmark PlotCraft below icon, sans-serif. TECHNICAL: square, scalable. CONSTRAINTS: max 3 colors, no gradients, works on light/dark backgrounds"'
```

### Mode B: Concept Art & Creative
**For:** Hero images, moodboards, backgrounds, illustrations, artistic exploration

- Use **ABBREVIATED schema** (Subject + Context + Style + Lighting)
- Emphasize: atmosphere, style fusion, cinematic quality
- Keywords: "rich detail", "depth and atmosphere", "blend [style A] with [style B]"

```bash
# Creative mode example
~/.claude/skills/google-image/google-img '/generate "SUBJECT: lone figure in rain. CONTEXT: cyberpunk megacity, neon-lit streets. STYLE: Blade Runner meets anime, cinematic. LIGHTING: neon reflections, volumetric fog" --count=3'
```

### Mode C: Diagrams & Technical
**For:** Flowcharts, architecture diagrams, infographics, wireframes, educational content

- Use **TECHNICAL schema** (Subject + Composition + Text + Constraints)
- Emphasize: clarity, legibility, proper labels
- Keywords: "clear labels", "high contrast", "no overlapping text", "simple geometric shapes"

```bash
# Diagram mode example
~/.claude/skills/google-image/google-img '/diagram "SUBJECT: user auth flow with OAuth. COMPOSITION: left-to-right, decision diamonds, process rectangles. TEXT: clear labels on each step. CONSTRAINTS: no overlapping lines, standard flowchart conventions"'
```

---

## Iterative Refinement Workflow

**Don't expect perfection on the first try.** Treat image generation as a conversation.

### Step 1: Clarify Vague Requests
If the user's prompt is vague ("cool logo", "nice background"), ask:
- What is the **use case**? (app icon, social media, print, web background)
- What **style**? (minimal, detailed, photorealistic, illustrated, abstract)
- Any **text** to include? (exact wording matters)
- **Aspect ratio** preference? (square, 16:9, portrait)
- **Brand colors** or palette?

### Step 2: Generate with Structured Prompt
Transform the user's request into a schema-based prompt, then generate 2-3 variations:
```bash
~/.claude/skills/google-image/google-img '/generate "[structured prompt]" --count=3'
```

### Step 3: Propose Targeted Follow-ups
After generating, suggest 2-3 specific refinements:
- "Tighter crop on the main subject"
- "More minimal layout, remove background elements"
- "Swap color palette to [specific colors]"
- "Keep everything but change expression to [X]"
- "Remove the [element], keep the rest identical"

### Editing Existing Images
Use conversational language for edits:
```bash
~/.claude/skills/google-image/google-img '/edit image.png "keep the composition but change the sky to sunset colors"'
```

---

## Quick Reference

See **[prompt-templates.md](./prompt-templates.md)** for 50+ ready-to-use templates organized by category.

---

## Prerequisites

```bash
# Install Gemini CLI and nanobanana extension
npm install -g @google/gemini-cli
gemini extensions install https://github.com/gemini-cli-extensions/nanobanana

# Ensure API key is set
export GEMINI_API_KEY=<your-key>
```

## Quick Start (Recommended)

Use the `google-img` wrapper script which handles everything automatically:

```bash
~/.claude/skills/google-image/google-img '/generate "description"'
```

**The wrapper automatically:**
- Sets `gemini-3-pro-image-preview` model for best quality
- Fixes JPEG files incorrectly saved as `.png`
- Flattens nested output directories
- Reports final file paths

### Custom Output Options

Control filenames and output location with wrapper flags:

```bash
# Custom filename (extension auto-detected from content)
~/.claude/skills/google-image/google-img --name="app-icon" '/generate "SUBJECT: chat bubble..."'
# Output: nanobanana-output/app-icon.jpg

# Custom output directory
~/.claude/skills/google-image/google-img --output="./assets/images" '/generate "..."'
# Output: ./assets/images/subject_xyz.jpg

# Both together
~/.claude/skills/google-image/google-img --name="hero" --output="./public" '/generate "..."'
# Output: ./public/hero.jpg

# Multiple images with custom name (auto-numbered)
~/.claude/skills/google-image/google-img --name="variant" '/generate "..." --count=3'
# Output: variant.jpg, variant_1.jpg, variant_2.jpg
```

### File Context for Prompts

Provide markdown or PDF files as context to inform image generation. The context is wrapped into the prompt using the format: "Based on this context: [file contents]. Create: [your prompt]"

**Prerequisites (for PDFs):**
```bash
# macOS
brew install poppler

# Linux
apt install poppler-utils
```

**Usage:**
```bash
# Single markdown file
~/.claude/skills/google-image/google-img --context="./brand-guide.md" '/generate "company logo"'

# Multiple files (comma-separated, max 10)
~/.claude/skills/google-image/google-img --context="./style.md,./colors.md" '/generate "hero banner"'

# PDF context
~/.claude/skills/google-image/google-img --context="./design-spec.pdf" '/generate "product mockup"'

# Combined with other flags
~/.claude/skills/google-image/google-img --context="./brand.md" --preset=landing --name="hero" \
  '/generate "SaaS dashboard preview"'
```

**Supported File Types:**
- `.md`, `.markdown` - Markdown files
- `.txt` - Plain text files
- `.pdf` - PDF documents (requires pdftotext)

**Limits:**
- Maximum 10 context files per request
- ~20MB total inline content limit
- Large files may exceed model context window (~65K tokens for image models)

| Flag | Purpose | Example |
|------|---------|---------|
| `--name=NAME` | Custom filename (no extension) | `--name="logo"` → `logo.jpg` |
| `--output=DIR` | Custom output directory | `--output="./assets"` |
| `--context=FILES` | Context files (comma-separated) | `--context="./brand.md,./style.md"` |

---

## Animation Generation

Generate animated sequences from AI-generated frames using `/story` + FFmpeg conversion.

### Basic Usage

```bash
# Simple animation (4 frames, 12fps, WebP+GIF output)
~/.claude/skills/google-image/google-img --animate "flower blooming from bud to full bloom"

# Custom settings
~/.claude/skills/google-image/google-img --animate --fps=8 --steps=6 --name="loading" \
  "circular spinner rotating smoothly"
```

### Animation Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--animate` | - | Enable animation mode |
| `--fps=N` | 12 | Frames per second (1-60) |
| `--steps=N` | 4 | Number of frames to generate (2-8) |
| `--loop=N` | 0 | Loop count (0=infinite) |
| `--formats=X` | webp,gif | Output formats (webp, gif, mp4) |
| `--ping-pong` | - | Play forward then reverse for seamless loops |

### Output Formats

| Format | Size | Quality | Best For |
|--------|------|---------|----------|
| **WebP** | Small (~170KB) | Excellent | Modern web (96% browser support) |
| **GIF** | Large (~1.7MB) | Good | Universal compatibility |
| **MP4** | Smallest | Excellent | Long animations, video players |

### Animation Examples

```bash
# Pixel art animation with ping-pong loop
~/.claude/skills/google-image/google-img --animate --fps=6 --steps=4 --ping-pong \
  --name="pixel-fire" --formats="gif" \
  "pixel art campfire flickering, 16-bit retro style"

# Loading spinner for web
~/.claude/skills/google-image/google-img --animate --fps=12 --steps=8 \
  --name="spinner" --output="./assets" \
  "minimal circular loading indicator rotating"

# Background effect with all formats
~/.claude/skills/google-image/google-img --animate --fps=8 --steps=6 \
  --formats="webp,gif,mp4" --name="ambient" \
  "floating particles drifting upward, dark background"

# Character idle animation
~/.claude/skills/google-image/google-img --animate --fps=4 --steps=4 --ping-pong \
  --name="idle" \
  "pixel character breathing idle animation, RPG style"
```

### Tips

- **Use `--ping-pong`** for smooth loops (e.g., pulsing, breathing)
- **Lower FPS (4-8)** works well for pixel art and stylized animations
- **Label removal is automatic** - the wrapper injects instructions to prevent "Step 1 of 4:" text overlays
- **WebP is 10x smaller** than GIF with better quality - prefer for web
- Source frames are preserved in `nanobanana-output/` for manual editing

---

## Web Design Mockups

Generate website mockups with proper structure using web design presets.

### Preset Reference

| Preset | Aspect | Best For |
|--------|--------|----------|
| `--preset=landing` | 1:1 | Single-page landing (one viewport) |
| `--preset=landing-scroll` | 9:16 | Full scrolling landing pages |
| `--preset=mobile` | 9:16 | Mobile landing pages |
| `--preset=hero` | 16:9 | Hero banner sections |
| `--preset=dashboard` | 16:9 | Admin dashboards, SaaS apps |
| `--preset=app-landing` | 9:16 | App/product landing pages |
| `--preset=portfolio` | 9:16 | Portfolio/creative sites |
| `--preset=ecommerce` | 9:16 | E-commerce/shop pages |

### Aspect Ratio Guide

**When to use 1:1 (square/single-page):**
- Single-viewport landing pages (no scrolling)
- Focused hero + CTA designs
- Simple product announcements

**When to use 9:16 (portrait/scrolling):**
- Full page layouts showing nav → hero → features → footer
- Landing pages, portfolios, e-commerce sites
- Any page where you need to see the complete structure

**When to use 16:9 (landscape/viewport):**
- Hero banner sections only
- Dashboard/app interfaces (desktop viewport)
- Above-fold screenshots

### Usage Examples

```bash
# Desktop SaaS landing page
~/.claude/skills/google-image/google-img --preset=landing --name="saas-landing" \
  '/generate "project management tool landing page, purple gradient, minimalist"'

# Mobile landing page (full scroll)
~/.claude/skills/google-image/google-img --preset=mobile --name="mobile-landing" \
  '/generate "fitness app landing page, energetic orange theme"'

# Dashboard mockup
~/.claude/skills/google-image/google-img --preset=dashboard --name="analytics" \
  '/generate "analytics dashboard, dark mode, charts and KPIs"'

# E-commerce storefront
~/.claude/skills/google-image/google-img --preset=ecommerce --name="shop" \
  '/generate "fashion boutique, minimal aesthetic, product grid"'
```

### Web Design Constraints (Auto-Injected)

All web presets automatically inject these constraints:
- Navigation bar positioned at TOP of page
- Readable English text only (no gibberish)
- Proper visual hierarchy (nav → hero → content → footer)
- Professional typography and layout

---

## Presentation Slides (Mode D)

Generate professional slide decks as PPTX files with AI-generated visuals, consistent themes, and speaker notes.

### The Process — Why This Matters

The quality of AI-generated slides depends almost entirely on the **thinking that happens before image generation**. The tools automate the mechanical parts (consistent theme injection, image generation, PPTX assembly), but the plan is where the real work happens.

**Great slides come from a great plan. The plan comes from understanding the content.**

### Step 1: Understand the Content (The Most Important Step)

Before touching any tools, you need to understand:
- **What is the presentation about?** Not just the topic — the argument, the narrative arc.
- **Who is the audience?** What do they already know? What do they care about?
- **How long is the presentation?** This determines slide count. Rule of thumb: ~40 seconds per slide.
- **What is the one thing the audience should remember?** This shapes your closing slide.

If you're working with a user:
- Have a conversation. Ask them to explain the idea as if talking to a friend.
- Listen for what they emphasize, what they repeat, what gets them excited.
- That emphasis becomes the structure.

If you're working from a document:
- Read it fully. Identify the thesis, supporting points, and conclusion.
- Cut ruthlessly. A 5-minute presentation can only make 5-6 points well.

### Step 2: Create the Plan JSON

The plan is a JSON file that defines the theme (applied to ALL slides) and each slide's content and visual direction.

```json
{
  "title": "My Presentation",
  "output_dir": "./presentation",
  "theme": {
    "background": "dark charcoal/navy",
    "accent": "gold/amber",
    "text": "white",
    "style": "clean modern corporate presentation",
    "motif": "subtle geometric patterns along bottom edge",
    "font": "bold sans-serif",
    "constraints": "no photos of real people, no watermarks, no clutter, text must be perfectly legible"
  },
  "slides": [
    {
      "number": 1,
      "name": "title",
      "title": "COMPANY NAME",
      "content": "Tagline or subtitle text here",
      "notes": "What you say while this slide is showing.",
      "visual": "Large centered title, motif border at bottom, warm glow behind text"
    },
    {
      "number": 2,
      "name": "problem",
      "title": "THE PROBLEM",
      "content": "Bullet 1. Bullet 2. Bullet 3.",
      "notes": "Explain the problem in your own words. This is your script.",
      "visual": "Left side shows icon or illustration, right side has title and bullet points"
    }
  ]
}
```

#### Plan JSON Schema

| Field | Required | Description |
|-------|----------|-------------|
| `title` | Yes | Presentation title (used for PPTX filename) |
| `output_dir` | Yes | Where to save images and PPTX |
| `theme` | Yes | Visual theme applied to every slide (see below) |
| `slides` | Yes | Array of slide definitions |

#### Theme Object

| Field | Description | Example |
|-------|-------------|---------|
| `background` | Background color/style | `"dark charcoal/navy"` |
| `accent` | Accent color for icons, borders, highlights | `"gold/amber"` |
| `text` | Primary text color | `"white"` |
| `style` | Overall visual style | `"clean modern corporate presentation"` |
| `motif` | Recurring visual element for brand consistency | `"kente-inspired geometric patterns"` |
| `font` | Typography direction | `"bold sans-serif"` |
| `constraints` | What to avoid across all slides | `"no photos, no watermarks"` |

#### Slide Object

| Field | Required | Description |
|-------|----------|-------------|
| `number` | Yes | Slide order (1-indexed) |
| `name` | Yes | Short name (used in filename: `01_title.jpg`) |
| `title` | No | Headline text on the slide |
| `content` | No | Body text, bullet points, or labels |
| `notes` | No | Speaker notes (what you say out loud) |
| `visual` | Yes | Visual layout direction — what the slide looks like |

#### Writing Good Visual Directions

The `visual` field is the most important field per slide. It tells the image generator HOW to compose the slide. Be specific about:
- **Layout**: "split layout, icon left, text right" vs "centered text, full bleed"
- **Elements**: "three connected icons in a horizontal row" vs "circular diagram with four nodes"
- **Text placement**: "title bold top-left, bullets below" vs "large centered quote"

**Good visual directions:**
```
"Left side: glowing outline map of Africa in gold. Right side: title and five bullet points in white text"
"Circular flywheel diagram with four nodes connected by curved gold arrows. Center text reads: Each cycle compounds"
"Large inspirational quote centered, three short gold taglines below, kente border at bottom matching title slide"
```

**Bad visual directions:**
```
"Make it look nice"
"Professional slide"
"Something about our product"
```

### Step 3: Generate Slide Images

```bash
# Generate all slides from the plan
~/.claude/skills/google-image/google-img --slides-generate=plan.json

# Regenerate just one slide that came out wrong
~/.claude/skills/google-image/google-img --slides-generate=plan.json --slide=3
```

The tool reads the plan, composes a full prompt for each slide by combining the theme with the slide's visual direction, and generates the images. Files are named `01_title.jpg`, `02_problem.jpg`, etc.

### Step 4: Review and Iterate

Look at each generated image. Common issues and fixes:
- **Text is garbled**: Simplify the text content in the plan, use fewer words
- **Layout is wrong**: Be more specific in the `visual` field about positioning
- **Inconsistent style**: Check that the theme fields aren't contradicting the visual direction
- **One slide looks off**: Regenerate just that slide with `--slide=N`

### Step 5: Build PPTX

```bash
# Package images + speaker notes into a PowerPoint file
~/.claude/skills/google-image/google-img --slides-build=plan.json
```

This creates a 16:9 PPTX with each slide image as a full-bleed background and speaker notes populated from the plan.

### Slides Quick Reference

| Command | Description |
|---------|-------------|
| `--slides-generate=plan.json` | Generate all slide images from plan |
| `--slides-generate=plan.json --slide=3` | Regenerate slide 3 only |
| `--slides-build=plan.json` | Package into PPTX with speaker notes |

### Presentation Planning Tips

**Slide count by duration:**
| Duration | Slides | Seconds/slide |
|----------|--------|---------------|
| 3 min | 5-6 | ~35s |
| 5 min | 7-8 | ~40s |
| 10 min | 12-15 | ~45s |
| 20 min | 20-25 | ~50s |

**Common slide types and their visual patterns:**

| Type | Visual Pattern |
|------|---------------|
| **Title** | Large centered text, decorative motif, minimal |
| **Problem/Opportunity** | Split layout: visual on one side, text on other |
| **How it works** | Icons in a row with labels, connected by lines |
| **Timeline/Journey** | Left-to-right flow with stages |
| **Flywheel/Cycle** | Circular diagram with nodes and arrows |
| **Comparison** | Two columns side by side |
| **Quote/Vision** | Large centered text, aspirational feel |
| **Data** | Chart, graph, or key numbers highlighted |

### Full Example

Here's a complete plan for a 5-minute startup pitch:

```json
{
  "title": "Acme Labs Vision",
  "output_dir": "./pitch-deck",
  "theme": {
    "background": "dark charcoal/navy",
    "accent": "electric blue",
    "text": "white",
    "style": "clean modern tech presentation, minimalist",
    "motif": "subtle circuit-board pattern along bottom edge",
    "font": "bold sans-serif",
    "constraints": "no stock photos, no watermarks, text must be legible, clean minimal layout"
  },
  "slides": [
    {
      "number": 1,
      "name": "title",
      "title": "ACME LABS",
      "content": "Making developers 10x faster",
      "notes": "Hi, I'm [Name], founder of Acme Labs. We make developers 10x faster.",
      "visual": "Large centered title in white, subtitle below in lighter weight, circuit motif at bottom, blue glow behind title"
    },
    {
      "number": 2,
      "name": "problem",
      "title": "THE PROBLEM",
      "content": "60% of dev time is spent reading code. Code review takes 4-8 hours per PR. Context switching kills flow state.",
      "notes": "Developers spend most of their time reading, not writing. Code review alone eats 4 to 8 hours per pull request.",
      "visual": "Left side: hourglass icon in blue outline draining. Right side: title and three bullet points with small clock icons"
    },
    {
      "number": 3,
      "name": "solution",
      "title": "THE SOLUTION",
      "content": "AI that understands your entire codebase. Instant code review. Automatic context loading.",
      "notes": "We built an AI that reads your entire codebase so you don't have to. Instant reviews, automatic context.",
      "visual": "Center: glowing brain/chip icon connected to three feature cards arranged in a triangle below it"
    },
    {
      "number": 4,
      "name": "traction",
      "title": "TRACTION",
      "content": "500 teams. $2M ARR. 95% retention. Growing 20% month over month.",
      "notes": "500 teams use us daily. Two million in recurring revenue. 95 percent retention. Growing 20 percent monthly.",
      "visual": "Four large numbers arranged in a 2x2 grid, each with a label below. Subtle upward arrow behind the grid"
    },
    {
      "number": 5,
      "name": "ask",
      "title": "THE ASK",
      "content": "Raising $5M Series A to expand to 5,000 teams",
      "notes": "We're raising 5 million to go from 500 to 5,000 teams. Here's how we'll do it.",
      "visual": "Clean centered text, large number $5M prominent, growth arrow from 500 to 5000, circuit motif at bottom matching title"
    }
  ]
}
```

### Requirements

- `jq` - for reading plan JSON (`brew install jq`)
- `python-pptx` - for PPTX assembly (`pip install python-pptx`)

---

### Examples

```bash
# Hero image
~/.claude/skills/google-image/google-img '/generate "a cyberpunk city at night" --count=2'

# App icon
~/.claude/skills/google-image/google-img '/icon "chat bubble" --style=modern --type=app-icon'

# Pattern/texture
~/.claude/skills/google-image/google-img '/pattern "geometric waves" --type=seamless'

# Diagram
~/.claude/skills/google-image/google-img '/diagram "user auth flow" --type=flowchart'
```

## Available Commands

| Command | Purpose | Key Options |
|---------|---------|-------------|
| `/generate` | Text-to-image | `--count=N`, `--styles="style1,style2"` |
| `/icon` | App icons & UI | `--type=app-icon`, `--style=modern`, `--sizes="64,128"` |
| `/pattern` | Seamless textures | `--type=seamless`, `--size="512x512"` |
| `/diagram` | Technical diagrams | `--type=flowchart\|sequence\|mindmap` |
| `/edit` | Modify images | `/edit image.png "make darker"` |
| `/restore` | Enhance photos | `/restore old.jpg "remove scratches"` |
| `/story` | Image sequences | `--steps=4` |
| `/nanobanana` | Natural language | Flexible prompting |

## Output

- Images saved to `./nanobanana-output/`
- Add `--preview` to auto-open generated images
- Use `--count=N` (1-8) for multiple variations

## Manual Method (Alternative)

If not using the wrapper, run these two commands:

```bash
# 1. Generate
NANOBANANA_MODEL=gemini-3-pro-image-preview gemini -y '/generate "prompt"'

# 2. Fix extensions (required - some JPEGs saved as .png)
find ./nanobanana-output -name "*.png" -exec sh -c 'file "$1" | grep -q JPEG && mv "$1" "${1%.png}.jpg" && echo "Fixed: $1"' _ {} \;
```

### Why Post-Processing?

The Gemini API returns JPEG or PNG dynamically, but nanobanana always saves as `.png`. Claude Code validates mime types strictly—a JPEG with `.png` extension will fail to display. The wrapper/fix script ensures extensions match actual content.

## Model Notes

- `gemini-3-pro-image-preview` produces **1408x768 widescreen JPEGs**
- Default model produces **1024x1024 square PNGs**
- The wrapper always uses the pro model for best results

---
> Source: [japhet19/gemini-image-skill](https://github.com/japhet19/gemini-image-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
