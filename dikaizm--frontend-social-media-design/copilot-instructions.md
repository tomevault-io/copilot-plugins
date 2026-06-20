## frontend-social-media-design

> Create stunning, branded social media designs as single-page HTML files — Instagram posts/stories/carousels, LinkedIn banners, Twitter cards, and more. Supports Unsplash API for automatic image sourcing. Helps non-designers discover their aesthetic through visual exploration rather than abstract choices.


# Frontend Social Media Design Skill

Create zero-dependency, visually striking HTML designs for social media content that can be screenshotted or exported as images. This skill helps non-designers discover their preferred aesthetic through visual exploration ("show, don't tell"), then generates production-quality social media graphics.

## Core Philosophy

1. **Zero Dependencies** — Single HTML files with inline CSS/JS. No npm, no build tools.
2. **Show, Don't Tell** — People don't know what they want until they see it. Generate visual previews, not abstract choices.
3. **Distinctive Design** — Avoid generic "AI slop" aesthetics. Every design should feel custom-crafted.
4. **Production Quality** — Code should be well-commented, accessible, and pixel-perfect at target dimensions.
5. **Canvas Sizing (CRITICAL)** — Every design MUST use exact fixed dimensions matching the target social media platform. No scrolling, no overflow.

---

## CRITICAL: Canvas Sizing Requirements

**This section is mandatory for ALL designs. Every canvas must be pixel-perfect at the target platform dimensions.**

### The Golden Rule

```
Each design = exactly one fixed-size canvas (e.g., 1080×1080px)
Content overflows? → Split into carousel pages or simplify
Never scroll within a canvas.
```

### Platform Dimensions

| Platform | Format | Dimensions (px) |
|----------|--------|-----------------|
| Instagram | Post (Square) | 1080 × 1080 |
| Instagram | Post (Portrait) | 1080 × 1350 |
| Instagram | Story / Reel | 1080 × 1920 |
| Instagram | Carousel Page | 1080 × 1080 or 1080 × 1350 |
| Facebook | Post | 1200 × 630 |
| Twitter / X | Post | 1200 × 675 |
| LinkedIn | Post | 1200 × 628 |
| LinkedIn | Carousel Page | 1080 × 1080 or 1080 × 1350 |
| YouTube | Thumbnail | 1280 × 720 |
| Pinterest | Pin | 1000 × 1500 |
| TikTok | Cover | 1080 × 1920 |

### Content Density Limits

To guarantee content fits, enforce these limits per canvas/page:

| Content Type | Maximum Content |
|--------------|----------------|
| Quote post | 1 quote (max 3 lines) + attribution + optional logo |
| Headline post | 1 headline + 1 subtitle + CTA |
| Feature post | 1 headline + 3-4 bullet points or icons |
| Image post | 1 headline + 1 image (max 60% canvas height) + optional caption |
| Carousel page | 1 heading + 3-4 points OR 1 heading + 1 image + caption |
| Infographic | 1 heading + 4-6 data points/icons |

**If content exceeds these limits → Split into carousel pages**

### Required CSS Architecture

Every design MUST include the base CSS from `STYLE_PRESETS.md`. Key requirements:

- `.canvas` has exact `width` and `height` in pixels
- `overflow: hidden` prevents any content from escaping
- `container-type: inline-size` enables container queries for responsive typography
- All typography uses `clamp()` with container-query-relative units (`cqw`)
- Carousel designs use `.carousel` + `.carousel-track` + `.carousel-page` structure

**See STYLE_PRESETS.md for complete base CSS, carousel CSS, and Unsplash overlay CSS.**

### When Content Doesn't Fit

**DO:**
- Split into carousel pages
- Reduce text (shorter headlines, fewer bullets)
- Use icons instead of text where possible
- Create a "continued" page in the carousel

**DON'T:**
- Reduce font size below readable limits
- Remove padding/spacing
- Allow any overflow
- Cram content to fit

---

## Phase 0: Detect Mode

First, determine what the user wants:

**Mode A: Single Post**
- User wants to create a single social media graphic
- Proceed to Phase 1 (Content Discovery)

**Mode B: Carousel**
- User wants a multi-page carousel (Instagram, LinkedIn, etc.)
- Proceed to Phase 1, with carousel-specific questions

**Mode C: Batch Generation**
- User wants multiple designs (e.g., a set of quote posts, a content series)
- Proceed to Phase 1, clarify how many and what varies between them

**Mode D: Existing Design Enhancement**
- User has an HTML design and wants to improve it
- Read the existing file, understand the structure, then enhance

### Mode D: Critical Modification Rules

When enhancing existing designs, follow these mandatory rules:

**1. Before Adding Any Content:**
- Check current content against density limits
- Calculate if new content will fit within the canvas

**2. When Adding Images:**
- Images must have `max-height: 60%` or similar canvas constraint
- If current canvas is already full → split into carousel
- Unsplash images need an overlay for text readability

**3. Required Checks After ANY Modification:**
```
✅ Does the canvas have `overflow: hidden`?
✅ Are all new elements using appropriate font sizes?
✅ Do new images have canvas-relative max-height?
✅ Does total content respect density limits?
✅ Is Unsplash attribution included if using API photos?
```

---

## Phase 1: Content Discovery

Before designing, understand the content.

### Step 1.1: Design Context (Single Form)

**IMPORTANT:** Ask ALL questions in a single AskUserQuestion call.

**Question 1: Platform**
- Header: "Platform"
- Question: "Which platform is this for?"
- Options:
  - "Instagram" — Posts, stories, carousels, reels covers
  - "LinkedIn" — Posts, carousel documents, banners
  - "Twitter / X" — Post images, header banners
  - "Facebook" — Post images, story covers
  - "YouTube" — Thumbnails, channel art
  - "Pinterest" — Pins, idea pins
  - "TikTok" — Video covers
  - "Custom" — Specify custom dimensions

**Question 2: Format**
- Header: "Format"
- Question: "What format?"
- Options:
  - "Single Post (Square)" — Standard 1:1 post
  - "Single Post (Portrait)" — Taller 4:5 post
  - "Story / Reel Cover" — Vertical 9:16
  - "Carousel" — Multi-page swipeable post (specify page count)
  - "Banner / Header" — Wide format for profiles
  - "Thumbnail" — Video thumbnail (16:9)

**Question 3: Content Type**
- Header: "Content"
- Question: "What kind of content?"
- Options:
  - "Quote / Text" — Inspirational quote, tip, or text-heavy content
  - "Product / Promotion" — Showcasing a product, service, or offer
  - "Announcement" — Event, launch, news
  - "Educational / How-to" — Tips, steps, tutorials (great for carousels)
  - "Brand / Portfolio" — Showcasing work, brand identity
  - "Infographic / Data" — Statistics, comparisons, data visualization

**Question 4: Images**
- Header: "Images"
- Question: "How should images be sourced?"
- Options:
  - "Use Unsplash (Recommended)" — Auto-find the best images via Unsplash API
  - "I have my own images" — Select Other and type/paste your image folder path
  - "No images" — Text-only design with CSS-generated visuals

**Question 5: Carousel Pages** *(only if "Carousel" selected)*
- Header: "Pages"
- Question: "How many carousel pages?"
- Options:
  - "3-5 pages" — Short carousel
  - "6-8 pages" — Medium carousel
  - "9-10 pages" — Long carousel (Instagram max is 10)

If user has content, ask them to share it (text, bullet points, etc.).

### Step 1.2: Image Evaluation

**If user selected "No images"** → Skip entirely. Use CSS-generated visuals.

**If user selected "Use Unsplash"** → Skip to Phase 3 where Unsplash search happens automatically based on content.

**If user provides an image folder:**

1. **Scan the folder** — List all image files
2. **View each image** — Use the Read tool to see what each image contains
3. **Evaluate each image** — For each, assess:
   - What it shows (screenshot, logo, chart, photo)
   - **Usability:** Clear, relevant, high quality? Mark as `USABLE` or `NOT USABLE`
   - **Content signal:** What concept does this image represent?
   - Dominant colors (important for style compatibility)
4. **Present evaluation and proposed layout** — Show which images are usable, propose where they go

**For carousels with user images:**
- Assign one primary image per page maximum
- Keep visual consistency (same treatment, same framing)
- Logo can appear on first and last pages

5. **Confirm via AskUserQuestion:**

**Question: Layout Confirmation**
- Header: "Layout"
- Question: "Does this layout and image selection look right?"
- Options:
  - "Looks good, proceed" — Move on to style selection
  - "Adjust images" — Change which images go where
  - "Adjust layout" — Change the page structure

---

## Phase 2: Style Discovery (Visual Exploration)

**CRITICAL: This is the "show, don't tell" phase.**

### How Users Choose Presets

**Option A: Guided Discovery (Default)**
- User answers mood questions
- Skill generates 3 preview files
- User picks their favorite

**Option B: Direct Selection**
- User names a preset: "Use the Bold Signal style"
- Skip to Phase 3

**Available Presets:**
| Preset | Vibe | Best For |
|--------|------|----------|
| Bold Signal | Confident, high-impact | Promotional posts, product launches |
| Electric Studio | Clean, professional | Agency portfolios, LinkedIn |
| Creative Voltage | Energetic, retro-modern | Event promotions, hype content |
| Dark Botanical | Elegant, sophisticated | Luxury brands, inspirational quotes |
| Notebook Tabs | Editorial, organized | Educational carousels, tutorials |
| Pastel Geometry | Friendly, approachable | Product features, how-to posts |
| Split Pastel | Playful, modern | Lifestyle, comparison posts |
| Vintage Editorial | Witty, personality-driven | Personal brands, thought leadership |
| Neon Cyber | Futuristic, techy | Tech launches, SaaS announcements |
| Terminal Green | Developer-focused | Dev tips, coding tutorials |
| Swiss Modern | Minimal, precise | Data visualizations, infographics |
| Paper & Ink | Literary, thoughtful | Quote posts, storytelling carousels |
| Neural Research | Academic, AI/tech-forward | Research carousels, model explainers, ML content |
| Stripe Blueprint | Clean, startup-engineering | AI educational carousels, SaaS features, tech blogs |

### Step 2.0: Style Path Selection

**Question: Style Selection Method**
- Header: "Style"
- Question: "How would you like to choose your style?"
- Options:
  - "Show me options" — Generate 3 previews based on my needs (recommended)
  - "I know what I want" — Let me pick from the preset list directly

**If "Show me options"** → Continue to Step 2.1

**If "I know what I want"** → Show preset picker, then skip to Phase 3.

### Step 2.1: Mood Selection (Guided Discovery)

**Question 1: Feeling**
- Header: "Vibe"
- Question: "What feeling should viewers get from your design?"
- Options:
  - "Impressed/Confident" — Professional, trustworthy, authoritative
  - "Excited/Energized" — Bold, innovative, attention-grabbing
  - "Calm/Focused" — Clean, thoughtful, easy to absorb
  - "Inspired/Moved" — Emotional, storytelling, memorable
- multiSelect: true (can choose up to 2)

### Step 2.2: Generate Style Previews

Based on mood, generate **3 distinct style previews** as mini HTML files. Each preview should be a single canvas (at the target dimensions) showing:

- Typography (font choices, heading/body hierarchy)
- Color palette (background, accent, text colors)
- Overall aesthetic feel
- Example content layout

**Preview Styles to Consider (pick 3 based on mood):**

| Mood | Style Options |
|------|---------------|
| Impressed/Confident | "Bold Signal", "Electric Studio", "Dark Botanical" |
| Excited/Energized | "Creative Voltage", "Neon Cyber", "Split Pastel" |
| Calm/Focused | "Notebook Tabs", "Paper & Ink", "Swiss Modern" |
| Inspired/Moved | "Dark Botanical", "Vintage Editorial", "Pastel Geometry" |
| Academic/Tech | "Neural Research", "Terminal Blue", "Swiss Modern" |
| Impressed/Confident (light) | "Stripe Blueprint", "Electric Studio", "Swiss Modern" |

**IMPORTANT: Never use generic patterns:**
- Purple gradients on white backgrounds
- Inter, Roboto, or system fonts
- Standard blue primary colors

### Step 2.3: Present Previews

Create the previews in: `.claude-design/style-previews/`

```
.claude-design/style-previews/
├── style-a.html   # First style
├── style-b.html   # Second style
├── style-c.html   # Third style
└── assets/        # Any shared assets
```

Each preview file should be:
- Self-contained (inline CSS/JS)
- A single canvas at the target dimensions showing the aesthetic
- ~50-100 lines, not a full design

Present to user:
```
I've created 3 style previews for you to compare:

**Style A: [Name]** — [1 sentence description]
**Style B: [Name]** — [1 sentence description]
**Style C: [Name]** — [1 sentence description]

Open each file to see them in action:
- .claude-design/style-previews/style-a.html
- .claude-design/style-previews/style-b.html
- .claude-design/style-previews/style-c.html
```

Then use AskUserQuestion:

**Question: Pick Your Style**
- Header: "Style"
- Question: "Which style preview do you prefer?"
- Options:
  - "Style A: [Name]" — [Brief description]
  - "Style B: [Name]" — [Brief description]
  - "Style C: [Name]" — [Brief description]
  - "Mix elements" — Combine aspects from different styles

---

## Phase 3: Generate Design

Now generate the full design based on:
- Content from Phase 1
- Style from Phase 2
- Platform & format specifications

### Unsplash Image Pipeline

**Skip this section if user chose "No images" or provided their own images.**

When the user selected "Use Unsplash", source images automatically based on the content.

#### Step 3.1: Unsplash API Configuration

The Unsplash API requires an access key. Check for it in this order:

1. Environment variable: `UNSPLASH_ACCESS_KEY`
2. If not found, ask the user:

**Question: Unsplash API Key**
- Header: "Unsplash"
- Question: "I need an Unsplash API key to search for images. You can get a free one at https://unsplash.com/developers. Paste your key, or select 'Skip' to use text-only design."
- Options:
  - "Skip" — Proceed without images (text-only with CSS visuals)

#### Step 3.2: Search for Images

Use the Unsplash API to find the most suitable images for the content:

```bash
# Search example — adapt query to match the design's content/topic
curl -s "https://api.unsplash.com/search/photos?query=SEARCH_QUERY&orientation=ORIENTATION&per_page=5" \
  -H "Authorization: Client-ID ${UNSPLASH_ACCESS_KEY}" | python3 -c "
import json, sys
data = json.load(sys.stdin)
for photo in data.get('results', []):
    print(f\"ID: {photo['id']}\")
    print(f\"Description: {photo.get('alt_description', 'N/A')}\")
    print(f\"URL (regular): {photo['urls']['regular']}\")
    print(f\"URL (full): {photo['urls']['full']}\")
    print(f\"Photographer: {photo['user']['name']}\")
    print(f\"Profile: {photo['user']['links']['html']}\")
    print('---')
"
```

**Search Strategy:**
- Derive search query from the content topic, headline, or theme
- Use appropriate orientation:
  - `landscape` for wide formats (Twitter, LinkedIn, YouTube thumbnails)
  - `portrait` for tall formats (stories, Pinterest pins)
  - `squarish` for square posts
- Search for 3-5 images, then evaluate which best fits the content
- For carousels: one search per unique topic across pages, or a thematic search for consistent imagery

**Choosing the Right Image:**
1. View each candidate image using the Read tool (Claude is multimodal)
2. Evaluate: Does the image match the content message? The chosen style's color palette?
3. Pick the best match — or search again with refined keywords
4. For dark-themed styles: prefer images with darker tones or that work well under dark overlays
5. For light-themed styles: prefer bright, airy images

#### Step 3.3: Download and Place Images

```bash
# Download the chosen image at appropriate size
# Use &w= parameter to control width (saves bandwidth, matches canvas)
curl -sL "https://api.unsplash.com/photos/PHOTO_ID/download" \
  -H "Authorization: Client-ID ${UNSPLASH_ACCESS_KEY}" > /dev/null  # Trigger download tracking (API requirement)

curl -sL "PHOTO_URL&w=1080&q=85&fm=jpg" -o assets/bg-image.jpg
```

**Size Guidelines:**
| Canvas Width | Download Width (`&w=`) |
|-------------|----------------------|
| 1080px | `&w=1080` |
| 1200px | `&w=1200` |
| 1280px | `&w=1280` |
| 1500px | `&w=1500` |

**Unsplash Attribution (Required by API Terms):**
Every design using Unsplash images MUST include photographer attribution:

```html
<a class="unsplash-credit" href="PHOTOGRAPHER_PROFILE_URL?utm_source=your_app&utm_medium=referral"
   target="_blank">Photo by PHOTOGRAPHER_NAME on Unsplash</a>
```

The `.unsplash-credit` class from `STYLE_PRESETS.md` styles this as a subtle bottom-right overlay.

### User-Provided Image Pipeline

**Skip if user chose "No images" or "Use Unsplash".**

If user provided their own images, process them for the target style:

#### Step 3.4: Image Processing (Pillow)

For each curated image, process based on the chosen style:

```bash
pip install Pillow
```

```python
from PIL import Image, ImageDraw

# Circular crop (for logos)
def crop_circle(input_path, output_path):
    img = Image.open(input_path).convert('RGBA')
    w, h = img.size
    size = min(w, h)
    left = (w - size) // 2
    top = (h - size) // 2
    img = img.crop((left, top, left + size, top + size))
    mask = Image.new('L', (size, size), 0)
    draw = ImageDraw.Draw(mask)
    draw.ellipse([0, 0, size, size], fill=255)
    img.putalpha(mask)
    img.save(output_path, 'PNG')

# Resize (for oversized images)
def resize_max(input_path, output_path, max_dim=1080):
    img = Image.open(input_path)
    img.thumbnail((max_dim, max_dim), Image.LANCZOS)
    img.save(output_path, quality=85)
```

#### Step 3.5: Place Images

**Use direct file paths** — do NOT convert to base64:

```html
<img src="assets/photo.jpg" alt="Description" class="canvas-image">
```

**Image CSS classes:**
```css
/* Base image constraint */
.canvas-image {
    max-width: 100%;
    max-height: 60%;
    object-fit: cover;
    border-radius: 8px;
}

/* Background image (full bleed) */
.canvas-bg-image {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
    z-index: 0;
}

/* Logo */
.canvas-image.logo {
    max-height: 30%;
    max-width: 40%;
    object-fit: contain;
}
```

### File Structure

**All generated HTML files go inside the `generated/` folder, organised by project name. Exports (PNG/JPG frames) go inside `output/{project-name}/`.**

For single designs:
```
generated/
└── {project-name}/
    ├── design.html
    └── assets/
        └── image.jpg        # Unsplash downloads or user-provided images
output/
└── {project-name}/
    └── slide-01.png         # Exported frames
```

For carousels:
```
generated/
└── {project-name}/
    ├── carousel.html
    └── assets/
        ├── logo.png         # Brand logo (appears in every slide header)
        ├── demo-*.png       # Product screenshots / demo images
        └── viz-*.png        # Charts, data visualizations
export-carousel.js           # Export script (workspace root — shared)
output/
└── {project-name}/
    ├── slide-01.png
    ├── slide-02.png
    └── ...                  # One file per carousel page
```

**Naming the project folder:**
Derive `{project-name}` from the content topic — use lowercase kebab-case:
- "5 AI productivity tips" → `ai-productivity-tips`
- "Product launch carousel" → `product-launch`
- "Q1 results infographic" → `q1-results`

**`assets/` folder rules:**
- Always place `assets/` as a **sibling** of the HTML file inside `generated/{project-name}/`
- Reference images with **relative paths only**: `src="assets/filename.png"`
- **Keep original filenames** — HTML files reference them directly
- The `export-carousel.js` export script uses Chrome's `--allow-file-access-from-files` flag, so local `assets/` files are loaded correctly without a server
- Typical contents:
  | File pattern | Purpose |
  |---|---|
  | `logo-*.png` | Brand logo used in page headers |
  | `demo-*.png` | Product/UI screenshots for demo slides |
  | `viz-*.png` | Chart or data visualization images |
  | `bg-*.jpg` | Background images (Unsplash downloads) |

### HTML Architecture — Single Post

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Social Media Design</title>

    <!-- Fonts -->
    <link rel="stylesheet" href="https://api.fontshare.com/v2/css?f[]=...">

    <!-- Icons (choose one; Phosphor preferred) -->
    <!-- Phosphor Icons -->
    <script src="https://unpkg.com/@phosphor-icons/web@2.1.1"></script>
    <!-- Font Awesome Free (alternative) -->
    <!-- <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css"> -->

    <style>
        /* ===========================================
           CSS CUSTOM PROPERTIES (THEME)
           =========================================== */
        :root {
            /* Canvas dimensions */
            --canvas-width: 1080px;
            --canvas-height: 1080px;

            /* Colors */
            --bg-primary: #0a0f1c;
            --text-primary: #ffffff;
            --text-secondary: #9ca3af;
            --accent: #00ffcc;

            /* Typography */
            --font-display: 'Clash Display', sans-serif;
            --font-body: 'Satoshi', sans-serif;
            --title-size: clamp(28px, 5.5cqw, 72px);
            --body-size: clamp(14px, 2.2cqw, 24px);

            /* Spacing */
            --canvas-padding: clamp(24px, 5cqw, 80px);
            --content-gap: clamp(12px, 2.5cqw, 32px);
        }

        /* ===========================================
           BASE STYLES
           =========================================== */
        * { margin: 0; padding: 0; box-sizing: border-box; }

        html, body {
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            background: #1a1a1a;
        }

        /* ===========================================
           CANVAS — FIXED DIMENSIONS
           =========================================== */
        .canvas {
            width: var(--canvas-width);
            height: var(--canvas-height);
            overflow: hidden;
            position: relative;
            display: flex;
            flex-direction: column;
            container-type: inline-size;
        }

        .canvas-content {
            flex: 1;
            display: flex;
            flex-direction: column;
            justify-content: center;
            padding: var(--canvas-padding);
            overflow: hidden;
            position: relative;
            z-index: 2;
        }

        /* ... style-specific CSS ... */
    </style>
</head>
<body>
    <div class="canvas">
        <!-- Optional: Background image from Unsplash -->
        <img src="assets/bg-image.jpg" alt="" class="canvas-bg-image">
        <div class="canvas-overlay canvas-overlay--dark"></div>

        <div class="canvas-content">
            <h1>Your Headline</h1>
            <p>Supporting text or CTA</p>
        </div>

        <!-- Unsplash attribution (required) -->
        <a class="unsplash-credit" href="...">Photo by Name on Unsplash</a>
    </div>
</body>
</html>
```

### HTML Architecture — Carousel

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Carousel Design</title>

    <!-- Fonts -->
    <link rel="stylesheet" href="...">

    <!-- Icons (choose one; Phosphor preferred) -->
    <!-- Phosphor Icons -->
    <script src="https://unpkg.com/@phosphor-icons/web@2.1.1"></script>
    <!-- Font Awesome Free (alternative) -->
    <!-- <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css"> -->

    <style>
        :root {
            --canvas-width: 1080px;
            --canvas-height: 1080px;
            /* ... theme variables ... */
        }

        * { margin: 0; padding: 0; box-sizing: border-box; }

        html, body {
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            background: #1a1a1a;
        }

        /* ===========================================
           CAROUSEL CONTAINER
           =========================================== */
        .carousel {
            width: var(--canvas-width);
            height: var(--canvas-height);
            overflow: hidden;
            position: relative;
        }

        .carousel-track {
            display: flex;
            width: 100%;
            height: 100%;
            transition: transform 0.4s cubic-bezier(0.25, 0.46, 0.45, 0.94);
        }

        .carousel-page {
            min-width: 100%;
            height: 100%;
            overflow: hidden;
            position: relative;
            display: flex;
            flex-direction: column;
            container-type: inline-size;
        }

        .carousel-page .canvas-content {
            flex: 1;
            display: flex;
            flex-direction: column;
            justify-content: center;
            padding: var(--canvas-padding);
            overflow: hidden;
            position: relative;
            z-index: 2;
        }

        /* Navigation dots */
        .carousel-dots {
            position: absolute;
            bottom: 24px;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            gap: 8px;
            z-index: 10;
        }

        .carousel-dot {
            width: 8px;
            height: 8px;
            border-radius: 50%;
            background: rgba(255,255,255,0.3);
            border: none;
            cursor: pointer;
            transition: all 0.3s ease;
            padding: 0;
        }

        .carousel-dot.active {
            background: rgba(255,255,255,0.9);
            transform: scale(1.3);
        }

        /* Arrow navigation */
        .carousel-nav {
            position: absolute;
            top: 50%;
            transform: translateY(-50%);
            width: 36px;
            height: 36px;
            border-radius: 50%;
            background: rgba(255,255,255,0.15);
            border: none;
            color: white;
            font-size: 18px;
            cursor: pointer;
            z-index: 10;
            backdrop-filter: blur(4px);
        }

        .carousel-nav--prev { left: 12px; }
        .carousel-nav--next { right: 12px; }

        /* Page counter */
        .carousel-counter {
            position: absolute;
            top: 16px;
            right: 20px;
            font-size: 14px;
            color: rgba(255,255,255,0.6);
            z-index: 10;
        }

        /* ... style-specific CSS ... */
    </style>
</head>
<body>
    <div class="carousel">
        <div class="carousel-track" id="carouselTrack">
            <!-- Page 1: Cover -->
            <div class="carousel-page">
                <div class="canvas-content">
                    <h1>Carousel Title</h1>
                    <p>Swipe to explore →</p>
                </div>
            </div>

            <!-- Page 2 -->
            <div class="carousel-page">
                <div class="canvas-content">
                    <h2>Point One</h2>
                    <p>Details...</p>
                </div>
            </div>

            <!-- Page 3: CTA -->
            <div class="carousel-page">
                <div class="canvas-content">
                    <h2>Follow for more!</h2>
                </div>
            </div>
        </div>

        <!-- Navigation -->
        <button class="carousel-nav carousel-nav--prev" id="prevBtn">‹</button>
        <button class="carousel-nav carousel-nav--next" id="nextBtn">›</button>

        <!-- Dots -->
        <div class="carousel-dots" id="carouselDots"></div>

        <!-- Page counter -->
        <div class="carousel-counter" id="carouselCounter">1 / 3</div>
    </div>

    <script>
        /* ===========================================
           CAROUSEL CONTROLLER
           Handles navigation, dots, counter, swipe
           =========================================== */
        class CarouselController {
            constructor() {
                this.track = document.getElementById('carouselTrack');
                this.pages = this.track.querySelectorAll('.carousel-page');
                this.currentPage = 0;
                this.totalPages = this.pages.length;

                this.dotsContainer = document.getElementById('carouselDots');
                this.counter = document.getElementById('carouselCounter');
                this.prevBtn = document.getElementById('prevBtn');
                this.nextBtn = document.getElementById('nextBtn');

                this.init();
            }

            init() {
                // Create dots
                for (let i = 0; i < this.totalPages; i++) {
                    const dot = document.createElement('button');
                    dot.className = 'carousel-dot' + (i === 0 ? ' active' : '');
                    dot.addEventListener('click', () => this.goTo(i));
                    this.dotsContainer.appendChild(dot);
                }

                // Arrow buttons
                this.prevBtn.addEventListener('click', () => this.prev());
                this.nextBtn.addEventListener('click', () => this.next());

                // Keyboard navigation
                document.addEventListener('keydown', (e) => {
                    if (e.key === 'ArrowLeft') this.prev();
                    if (e.key === 'ArrowRight') this.next();
                });

                // Touch/swipe support
                let startX = 0;
                this.track.addEventListener('touchstart', (e) => {
                    startX = e.touches[0].clientX;
                });
                this.track.addEventListener('touchend', (e) => {
                    const diff = startX - e.changedTouches[0].clientX;
                    if (Math.abs(diff) > 50) {
                        diff > 0 ? this.next() : this.prev();
                    }
                });

                this.updateUI();
            }

            goTo(index) {
                this.currentPage = Math.max(0, Math.min(index, this.totalPages - 1));
                this.track.style.transform = `translateX(-${this.currentPage * 100}%)`;
                this.updateUI();
            }

            prev() { this.goTo(this.currentPage - 1); }
            next() { this.goTo(this.currentPage + 1); }

            updateUI() {
                // Update dots
                const dots = this.dotsContainer.querySelectorAll('.carousel-dot');
                dots.forEach((dot, i) => {
                    dot.classList.toggle('active', i === this.currentPage);
                });

                // Update counter
                this.counter.textContent = `${this.currentPage + 1} / ${this.totalPages}`;

                // Update arrows visibility
                this.prevBtn.style.opacity = this.currentPage === 0 ? '0.3' : '1';
                this.nextBtn.style.opacity = this.currentPage === this.totalPages - 1 ? '0.3' : '1';
            }
        }

        new CarouselController();
    </script>
</body>
</html>
```

### CDN Icon Libraries

Prefer CDN-hosted icon libraries over emoji for decorative icons, feature bullets, and UI accents. Icon fonts are crisp at any size, respect the design's color palette, and export cleanly as screenshots.

**Recommended Libraries:**

| Library | `<head>` include | Usage |
|---------|-----------------|-------|
| **Phosphor Icons** (preferred) | `<script src="https://unpkg.com/@phosphor-icons/web@2.1.1"></script>` | `<i class="ph ph-arrow-right"></i>` |
| **Font Awesome Free** | `<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css">` | `<i class="fa-solid fa-rocket"></i>` |
| **Bootstrap Icons** | `<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css">` | `<i class="bi bi-lightning-fill"></i>` |

**When to use icons vs. emoji:**

| Use **CDN Icons** | Use **Emoji** |
|-------------------|---------------|
| Feature bullets, tip lists | User-supplied text that already has emoji |
| UI signals (arrows, checks, badges) | Playful/casual brand voice where emoji fits |
| Infographic data points | One-off designs with no consistent icon set |
| Educational / professional carousels | Cultural expressions (e.g. flags, faces) |

**Sizing icons for social media canvases:**

Icon fonts render as text — size them with `font-size`:

```css
.icon    { font-size: clamp(32px, 4.5cqw, 56px); line-height: 1; }
.icon-sm { font-size: clamp(20px, 2.5cqw, 32px); line-height: 1; }
.icon-lg { font-size: clamp(48px, 6cqw,   80px); line-height: 1; }
.icon-xl { font-size: clamp(64px, 8cqw,  120px); line-height: 1; }
.icon-accent { color: var(--accent); }
```

**Icon + text bullet pattern (Phosphor):**

```html
<ul class="feature-list">
  <li><i class="ph-fill ph-check-circle icon icon-accent"></i> Benefit one</li>
  <li><i class="ph-fill ph-check-circle icon icon-accent"></i> Benefit two</li>
  <li><i class="ph-fill ph-check-circle icon icon-accent"></i> Benefit three</li>
</ul>
```

```css
.feature-list {
    list-style: none;
    display: flex;
    flex-direction: column;
    gap: var(--content-gap);
}
.feature-list li {
    display: flex;
    align-items: center;
    gap: 0.5em;
    font-size: var(--body-size);
}
```

**Icon-only grid pattern (Font Awesome):**

```html
<div class="icon-grid">
  <div class="icon-card">
    <i class="fa-solid fa-bolt icon-lg icon-accent"></i>
    <span>Fast</span>
  </div>
  <div class="icon-card">
    <i class="fa-solid fa-shield-halved icon-lg icon-accent"></i>
    <span>Secure</span>
  </div>
  <div class="icon-card">
    <i class="fa-solid fa-sliders icon-lg icon-accent"></i>
    <span>Flexible</span>
  </div>
</div>
```

**Phosphor icon weights** — match to the design style:

| Class prefix | Weight | When to use |
|---|---|---|
| `ph-thin` | Thin | Minimal / elegant designs |
| `ph-light` | Light | Clean, airy layouts |
| `ph` (default) | Regular | General purpose |
| `ph-bold` | Bold | Strong, attention-grabbing |
| `ph-fill` | Filled | High-contrast, solid shapes |
| `ph-duotone` | Duotone | Two-tone accent icons |

---

### Carousel Design Rules

When creating carousel content, follow these guidelines:

**1. Visual Consistency:**
- Same background color/gradient across all pages
- Same font sizes and spacing
- Same accent color usage
- Page numbers or dots for navigation context

**2. Content Flow:**
- **Page 1 (Cover):** Bold headline + hook. The "thumb-stopper."
- **Pages 2-N (Content):** One key point per page. Don't cram.
- **Last Page (CTA):** Call to action — follow, share, visit link, etc.

**3. Common Carousel Patterns:**
- **Tips/Steps:** "5 Tips for..." → one tip per page
- **Educational:** Problem on page 1, solution steps on pages 2-5, summary/CTA on last page
- **Storytelling:** Beginning → middle → end → CTA
- **Comparison:** Before vs After, Old Way vs New Way
- **Listicle:** "10 Tools You Need" → 2-3 tools per page

**4. Page Transitions:**
- Use consistent transitions (all pages use the same swipe direction)
- Keep transition duration short (0.3-0.5s)
- Don't use different animation styles per page

**5. No Animations — Export Rule (MANDATORY):**
All `animation:` properties and `@keyframes` declarations **must be removed** before finalising a design for export or screenshot. Animated elements should be replaced with their static/visible state:

| Animated element | Static replacement |
|---|---|
| Blinking cursor `.cursor` | Solid block, `opacity: 1`, no `animation` |
| Blinking dot `.gc-dot` | Solid dot, `opacity: 1`, no `animation` |
| Fade-in entrance | Remove entirely; element visible from the start |
| CSS transition on hover | Keep — transitions only fire on interaction, not on load |

> **Why:** When exporting carousel frames via Puppeteer or screenshot tools, a mid-animation frame produces glitchy, unprofessional output. Static designs export perfectly at any moment.

**6. Navigation Default — Dots + Swipe Only:**
For social media carousels, omit prev/next arrow buttons by default. They appear as UI chrome in screenshots/exports and reduce visual quality on exported images. Use only:
- Dot indicators (`#carouselDots`) for position context
- Touch/swipe support for mobile
- Keyboard arrow keys for desktop
Only add arrow buttons if the user explicitly requests them.

**7. Brand Placement — Top of Every Slide:**
Place the post branding (logo, brand name, and/or website URL) at the **top** of every slide as a fixed-position header element (`position: absolute; top: <padding>`). This ensures instant brand recognition as readers swipe through.
- Use `position: absolute` so it doesn't push content down
- Align logo to the left, optional brand tag/URL to the right
- Keep brand elements subtle (low opacity, small size) — they should not compete with the headline
- Ensure slide content `padding-top` is large enough to clear the brand header

```css
.brand-header {
    position: absolute;
    top: 36px;
    left: var(--canvas-padding);
    right: var(--canvas-padding);
    display: flex;
    align-items: center;
    justify-content: space-between;
    z-index: 5;
}
```

**8. Swipe Instruction & Slide Number — Bottom of Every Slide:**
Place the slide number counter and any swipe/navigation hint text at the **bottom** of every slide:
- Slide counter (e.g. `1 / 10`) — `position: absolute; bottom: <padding>; right: <padding>`
- Swipe hint text (e.g. "Swipe →") — only on the cover slide (Slide 1), bottom-left or bottom-center
- Dot indicators — positioned just above the counter (`bottom` value slightly higher than the counter)

```css
.carousel-counter {
    position: absolute;
    bottom: 40px;
    right: var(--canvas-padding);
    font-family: var(--font-mono);
    font-size: var(--small-size);
    color: rgba(255,255,255,0.5);
    z-index: 10;
}
```

**9. Dense Multi-Section Slides (Data/Results pages):**
When two related sections naturally belong together (e.g., system diagram + comparison table), they can share one carousel page. Use this pattern:
- Compact section header (`font-size` 24–36px instead of the full `--h2-size`)
- A thin 1px divider line between sections
- `padding-top/bottom: 24px` on `.canvas-content` to reclaim vertical space
- `gap: 0` on `.canvas-content` and manage spacing manually per section
This avoids unnecessary extra slides and keeps narrative flow tight.

### Code Quality Requirements

**Comments:**
Every section should have clear comments explaining what it does and how to modify it.

**Accessibility:**
- Semantic HTML
- Alt text on all images
- Keyboard navigation for carousels

**CSS Function Negation:**
Never negate CSS functions directly — use `calc(-1 * clamp(...))` instead. See STYLE_PRESETS.md → "CSS Gotchas".

---

## Phase 4: Carousel Generation (Detailed)

When generating carousels specifically:

### Step 4.1: Plan Page Structure

Before generating, plan each page:

```
Page 1 (Cover): Bold headline + subtitle + visual hook
Page 2: First key point + supporting detail/image
Page 3: Second key point + supporting detail/image
...
Page N (CTA): Call to action + branding
```

Present to user via AskUserQuestion:

**Question: Carousel Outline**
- Header: "Outline"
- Question: "Here's the proposed carousel flow. Does this look right?"
- Options:
  - "Looks good" — Proceed to generation
  - "Adjust" — I want to change the page structure

### Step 4.2: Consistent Styling

All pages MUST share:
- Same `:root` CSS variables
- Same background treatment
- Same typography scale
- Same padding values

Only vary:
- Content (text, images)
- Accent elements (different icon per page, etc.)
- Page-specific imagery

### Step 4.3: Unsplash for Carousels

When using Unsplash for carousels:
- Use ONE cohesive search theme (e.g., all "workspace" or all "nature")
- OR use one background image across all pages for maximum consistency
- OR use unique images per page but apply the same overlay treatment

**Per-page image search:**
```bash
# Search for images matching each page's topic
curl -s "https://api.unsplash.com/search/photos?query=PAGE_TOPIC&orientation=squarish&per_page=3" \
  -H "Authorization: Client-ID ${UNSPLASH_ACCESS_KEY}"
```

### Step 4.4: Generate All Pages

Generate the complete carousel HTML with:
- All pages in the `.carousel-track`
- CarouselController JavaScript
- Navigation dots + arrows + counter
- Swipe/touch support
- Keyboard navigation (← →)

---

## Phase 5: Delivery

### Final Output

When the design is complete:

1. **Clean up temporary files**
   - Delete `.claude-design/style-previews/` if it exists

2. **Save generated file** to `generated/{project-name}/`
   - HTML file: `generated/{project-name}/design.html` (or `carousel.html`)
   - Assets: `generated/{project-name}/assets/`
   - Create the folder if it does not exist

3. **Open the design**
   - Use `open generated/{project-name}/design.html` to launch in browser

4. **For carousels — export frames** to `output/{project-name}/`:
   ```bash
   node export-carousel.js \
     --file generated/{project-name}/carousel.html \
     --out  output/{project-name} \
     --format png
   ```
   Creates `output/{project-name}/slide-01.png`, `slide-02.png`, etc.

5. **Provide summary**
```
Your design is ready!

Generated file : generated/{project-name}/design.html
Style          : [Style Name]
Platform       : [Platform] — [Format]
Dimensions     : [W] × [H] px
Images         : [Unsplash / User-provided / None]

To export as image:
- Open in browser → Right-click → Inspect → Device mode
- Set exact dimensions ([W] × [H])
- Screenshot (Cmd+Shift+P → "Capture screenshot")

To customize:
- Colors: Change `:root` CSS variables
- Fonts: Update the font link
- Content: Edit text directly in the HTML
```

For carousels, also include:
```
Pages          : [count]
Exported to    : output/{project-name}/

Navigation (in browser preview):
- Arrow keys (← →) to navigate
- Click dots or arrows
- Swipe on touch devices

Export command:
  node export-carousel.js --file generated/{project-name}/carousel.html --out output/{project-name}
```

If Unsplash was used:
```
Unsplash photos used — attribution included in design
  Photographer: [Name] (unsplash.com/@handle)
```

---

## Style Reference: Effect → Feeling Mapping

Use this guide to match effects to intended feelings:

### Bold / Attention-Grabbing
- High-contrast colors
- Large, bold typography
- Colored shapes and blocks
- Minimal text, maximum impact

### Techy / Futuristic
- Neon glow effects
- Grid patterns, scan lines
- Monospace fonts for accents
- Dark backgrounds with accent lighting

### Playful / Friendly
- Rounded corners
- Pastel or bright colors
- Floating/bouncing elements
- Hand-drawn or illustrated accents

### Professional / Corporate
- Clean sans-serif fonts
- Navy, slate, or charcoal backgrounds
- Precise alignment
- Data visualization focus

### Calm / Minimal
- High whitespace
- Muted color palette
- Serif typography
- Generous padding

### Editorial / Magazine
- Strong typography hierarchy
- Pull quotes and callouts
- Image-text interplay
- Serif headlines, sans-serif body

---

## Animation Patterns Reference

Animations are optional for social media designs. They're useful for:
- **Browser preview** — seeing the design come to life
- **Animated social content** — if exporting as video/GIF

### Entrance Animations

```css
/* Fade + Slide Up */
.reveal {
    opacity: 0;
    transform: translateY(20px);
    transition: opacity 0.6s cubic-bezier(0.16, 1, 0.3, 1),
                transform 0.6s cubic-bezier(0.16, 1, 0.3, 1);
}

.canvas.loaded .reveal {
    opacity: 1;
    transform: translateY(0);
}

/* Stagger children */
.reveal:nth-child(1) { transition-delay: 0.1s; }
.reveal:nth-child(2) { transition-delay: 0.2s; }
.reveal:nth-child(3) { transition-delay: 0.3s; }
```

### Background Effects

```css
/* Gradient Mesh */
.gradient-bg {
    background:
        radial-gradient(ellipse at 20% 80%, rgba(120, 0, 255, 0.3) 0%, transparent 50%),
        radial-gradient(ellipse at 80% 20%, rgba(0, 255, 200, 0.2) 0%, transparent 50%),
        var(--bg-primary);
}

/* Grid Pattern */
.grid-bg {
    background-image:
        linear-gradient(rgba(255,255,255,0.03) 1px, transparent 1px),
        linear-gradient(90deg, rgba(255,255,255,0.03) 1px, transparent 1px);
    background-size: 50px 50px;
}
```

---

## Troubleshooting

### Common Issues

**Fonts not loading:**
- Check Fontshare/Google Fonts URL
- Ensure font names match in CSS

**Canvas too large/small in browser:**
- The canvas is displayed at its actual pixel size
- Use browser zoom or DevTools device mode to preview at different scales

**Unsplash API errors:**
- Verify `UNSPLASH_ACCESS_KEY` is set and valid
- Free tier: 50 requests/hour — check rate limits
- Ensure search query is URL-encoded

**Carousel not swiping:**
- Check touch event listeners are attached to `.carousel-track`
- Verify `transform: translateX()` is updating correctly

**Content overflows canvas:**
- Reduce content — split into carousel pages
- Check `overflow: hidden` on `.canvas` or `.carousel-page`
- Verify font sizes aren't too large for the content amount

**Export quality issues:**
- Ensure screenshot is taken at exact canvas dimensions
- Use `&w=1080&q=85` on Unsplash URLs for high quality
- For retina: consider 2x dimensions (2160×2160) scaled down

---

## Example Session Flow

1. User: "Create an Instagram carousel about 5 productivity tips"
2. Skill asks about platform, format, content type, images, page count
3. User picks: Instagram, Carousel, Educational, Use Unsplash, 6 pages
4. Skill asks about desired feeling → User picks "Calm/Focused"
5. Skill generates 3 style previews at 1080×1080
6. User picks Style A (Notebook Tabs)
7. Skill searches Unsplash for "productivity workspace minimal" with `orientation=squarish`
8. Skill downloads best matching image, generates 6-page carousel:
   - Page 1: Cover — "5 Productivity Tips" + Unsplash background
   - Pages 2-6: One tip per page with consistent styling
   - Page 7: CTA — "Save this & follow for more!"
9. Skill opens in browser, user reviews
10. User requests tweaks → final design delivered

---

## Related Skills

- **learn** — Generate documentation for the design
- **frontend-design** — For more complex interactive pages beyond social media
- **design-and-refine:design-lab** — For iterating on component designs

---
> Source: [dikaizm/frontend-social-media-design](https://github.com/dikaizm/frontend-social-media-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
