## presentation-builder

> Create professional, branded slide presentations from storyboards or from scratch. Produces HTML/CSS decks with data charts, illustrated graphics, callout boxes, and timelines -- exported as PDF. Use this skill whenever the user wants to build a presentation, create slides, make a deck, design a workshop, or turn research/content into visual slides. Also use when the user mentions PowerPoint, PPTX, keynote, slide deck, storyboard-to-slides, or wants to present information visually to an audience. Works end-to-end: can research topics and create the storyboard, or take an existing storyboard and produce the visual deck.


# Presentation Builder

Build professional, story-driven slide presentations from storyboards. Produces HTML/CSS decks with branded styling, data visualizations, and structured layouts -- exported as PDF for presenting.

The presentations this skill creates look like they were designed by a professional agency -- not like default PowerPoint templates. They use CSS Grid for precise layouts, matplotlib for publication-quality charts, inline SVG for icons and illustrations, and a consistent brand system throughout.

## When to Use This Skill

- User wants to create a presentation, slide deck, or workshop materials
- User has a storyboard or outline they want turned into slides
- User wants to research a topic AND produce a presentation about it
- User mentions slides, deck, PowerPoint, PDF presentation, keynote, or workshop
- User has content (research, notes, bullet points) they want to present visually

## End-to-End Workflow

The skill operates in two phases. Phase 1 can be skipped if the user already has a storyboard.

### Phase 1: Storyboard Creation (if needed)

If no storyboard exists, create one through research and synthesis.

**Step 1: Understand the presentation context**

Ask the user (skip questions they've already answered):
- What is the topic and who is the audience?
- How long is the presentation? (determines slide count: ~1 slide per 5-7 minutes)
- What is the narrative arc? (chronological, problem-solution, journey, comparison)
- Are there co-presenters? What sections do they own?
- Any specific data, case studies, or examples to include?

**Step 2: Research (if the topic requires it)**

Deploy parallel research agents (up to 5) to gather material:
- Each agent gets a specific subtopic with a clear objective
- Each must return structured summaries with sources/citations
- Synthesize findings after all agents return

**Step 3: Write the storyboard**

Create `storyboard.md` with this structure for each slide:

```markdown
## SLIDE N: [Title]
**Title:** "[Display title]"

### Setting the Scene
[Narrative context -- what's happening at this point in the story]

### Key Content
[Bullet points, data, tables -- the substance of the slide]

### Messaging
> [The key takeaway quote or message for this slide]

### Graphic Concept
[What visual/chart/diagram should appear and what it communicates]

### Audience Question (optional)
> [Discussion prompt to engage the audience]
```

Also include:
- **Workshop flow & timing** table (if it's a workshop/talk)
- **Graphics to produce** inventory (numbered list of every visual asset)
- **Case studies** section (if applicable)

Save as `storyboard.md` in the project directory. Present to user for review before proceeding.

### Phase 2: Visual Production

Turn the storyboard into a finished presentation.

**Step 1: Establish the style system**

Check if a `style_guide.md` exists. If not, ask the user:
- What organization/brand is this for? (to look up official colors)
- Or: what are 2-3 brand colors? (hex codes or color names)
- Serif or sans-serif preference?
- Professional/corporate or modern/startup feel?

If the user gives a brand name, research official brand colors and typography. Then generate `style_guide.md` with:
- Color palette (primary, secondary, accent, neutral, functional)
- Typography scale (heading sizes, body size, caption size -- presentation scale, not document scale)
- Component specs (callout boxes, stat boxes, tables, timelines)
- Chart color assignments (consistent series colors across all charts)
- Print/PDF settings

If a style guide already exists, read it and use it as-is.

**Step 2: Analyze the storyboard for visual assets**

Read the storyboard and catalog every graphic needed:
- **Data charts** (pie, bar, line, dual-axis, stacked) -- will use matplotlib
- **Layout graphics** (timelines, matrices, grids) -- will use HTML/CSS
- **Icons/illustrations** (characters, objects) -- will use inline SVG or icon libraries
- **Tables** -- will render as styled HTML tables

Create a mental inventory. You do not need to write a separate graphic_preparation_plan.md unless the user asks for one -- just proceed to production.

**Step 2b: Generate illustrations with Recraft V3 (optional)**

If the user wants illustrated graphics (characters, objects, icons) rather than plain inline SVG, use the Recraft V3 API to generate professional vector illustrations. This produces significantly higher-quality visuals than Claude-generated SVG -- the difference is like clip art vs. professional illustration.

**When to use Recraft:**
- User explicitly asks for illustrations, icons, or visual storytelling elements
- The presentation has characters/roles that need visual representation (e.g., "scientist", "investor", "patient")
- The deck needs object illustrations (lab equipment, pills, documents, money)
- User mentions Recraft specifically

**When NOT to use Recraft:**
- User wants a clean, data-focused deck with no illustrations
- No API key is available (check `.env` for `RECRAFT_API_KEY`)
- Budget constraints (each image costs ~$0.04 raster, ~$0.08 vector)

**How to generate illustrations:**

1. Check for the API key in `.env` or environment variables
2. Write a Python script (`graphics/src/generate_recraft.py`) that calls the Recraft API:

```python
import requests, os, time

# Load from environment variable or .env file
API_KEY = os.environ.get("RECRAFT_API_KEY", "")
if not API_KEY:
    from pathlib import Path
    env = Path(".env")
    if env.exists():
        for line in env.read_text().splitlines():
            if line.startswith("RECRAFT_API_KEY="):
                API_KEY = line.split("=", 1)[1].strip()
BASE_URL = "https://external.api.recraft.ai/v1/images/generations"
HEADERS = {"Authorization": f"Bearer {API_KEY}", "Content-Type": "application/json"}

def generate(name, prompt, output_dir):
    resp = requests.post(BASE_URL, headers=HEADERS, json={
        "prompt": prompt,
        "style": "digital_illustration",
        "model": "recraftv3",
        "size": "1024x1024",
    }, timeout=60)
    resp.raise_for_status()
    image_url = resp.json()["data"][0]["url"]
    img_data = requests.get(image_url, timeout=30).content
    path = os.path.join(output_dir, f"recraft_{name}.png")
    with open(path, "wb") as f:
        f.write(img_data)
    return path
```

3. Generate all needed illustrations with consistent prompting:
   - Always include "flat vector illustration style, simple clean lines, solid colors, white background, professional minimal design" in every prompt
   - Include brand colors in the prompt (e.g., "Cornell red and navy blue color palette")
   - Use specific role descriptions for characters (e.g., "scientist in white lab coat holding a flask")
   - Add `time.sleep(0.5)` between calls for rate limiting

4. Save all illustrations to `graphics/recraft_[name].png`

**Typical illustration set for a business presentation (~$0.50-$1.00):**
- 6-8 character illustrations (roles/stakeholders relevant to the topic)
- 4-6 object illustrations (domain-specific items -- lab equipment, documents, money, etc.)
- Total: 10-14 images at $0.04 each = ~$0.50

**Integrating illustrations into slides:**
- **Slide headers**: 100-120px illustration next to the title (use `position: absolute` or grid placement)
- **Inline accents**: 80px illustration paired with a text section in a card component
- **Timeline icons**: 40-50px illustrations above each timeline node instead of plain dots
- **Cast of characters**: 60-80px circular-cropped illustrations in a row
- **Intro slide decorations**: Semi-transparent (10-15% opacity) illustrations on the gradient background
- **Always use**: `border-radius`, `filter: drop-shadow()`, `object-fit: contain` for polished appearance

**Step 3: Generate data charts with matplotlib**

For every data chart identified in Step 2:

1. Create a shared style config (`graphics/src/chart_style.py`) using the brand colors from the style guide:

```python
import matplotlib.pyplot as plt
import matplotlib as mpl

COLORS = {
    'primary':   '#hex',  # from style guide
    'secondary': '#hex',
    'accent':    '#hex',
    # ... map all brand colors
}

SERIES_COLORS = {
    # Consistent color per data series across all charts
}

def apply_style():
    mpl.rcParams.update({
        'font.family': 'sans-serif',
        'font.sans-serif': ['Segoe UI', 'Helvetica Neue', 'Arial'],
        'font.size': 14,
        'axes.titlesize': 22,
        'axes.titleweight': 'bold',
        'axes.spines.top': False,
        'axes.spines.right': False,
        'figure.facecolor': 'white',
        'axes.facecolor': 'white',
        'legend.frameon': False,
        'savefig.dpi': 300,
        'savefig.bbox': 'tight',
    })
```

2. Write a Python script for each chart in `graphics/src/`
3. Run all chart scripts (in parallel if independent) to produce PNGs in `graphics/`

**Chart design principles:**
- Charts are the hero of their slide -- size them large, no artificial max-height constraints
- Use consistent colors across related charts (same stakeholder = same color everywhere)
- Include annotations (arrows, callout boxes, highlighted bars) for key insights
- White background, clean axes, no chartjunk
- Label directly on the chart rather than relying on legends when possible

**Step 4: Build the HTML presentation**

Create `presentation.html` -- a single file containing all slides. This is the core production step.

**Slide layout system (CSS Grid):**

Every content slide uses this grid structure:
```css
section.slide {
  width: 1920px;
  height: 1080px;
  display: grid;
  grid-template-columns: 1fr [sidebar-width];  /* e.g., 1fr 400px */
  grid-template-rows: auto 1fr auto auto;
  /* Row 1: Header (title + subtitle) -- full width */
  /* Row 2: Content column + Sidebar column */
  /* Row 3: Bottom element (audience Q, key takeaway) -- full width */
  /* Row 4: Timeline/footer -- full width */
}
```

This layout ensures:
- Content fills the available space (no wasted width)
- Sidebar elements (callout boxes, key stats) are grid-placed, not absolutely positioned
- Charts expand via `flex: 1` to fill vertical space naturally
- Bottom elements span full width
- Nothing overflows the 1080px height

**Slide components to include (based on storyboard):**
- Stat boxes row (key numbers with labels, branded bottom border)
- Data tables (navy header, alternating row colors)
- Chart images (`<img>` referencing the matplotlib PNGs)
- Callout/sidebar boxes (dark background, accent border, structured list items)
- Audience questions (light background, brand-color left border, italic text)
- Quote blocks (left border accent, italic, muted color)
- Progress timeline (circles connected by lines, progressive highlighting)
- 2x2 matrices (CSS Grid with colored quadrants)
- Inline SVG icons for simple illustrations

**The intro slide** should use `display: flex` (not the grid), centered content, dark background with brand gradient, and placeholder text for the user to fill in (`[Your Name]`, `[Date]`, etc.).

**Step 5: Render Verification with Playwright**

After building the HTML, verify that every slide renders correctly:

1. Start a local server: `python -m http.server 8765 --bind 127.0.0.1`
2. Open the presentation in Playwright at 1920x1080 viewport:
   ```bash
   playwright-cli open http://localhost:8765/presentation.html
   playwright-cli resize 1920 1080
   ```
3. Query the DOM to discover all slides:
   ```bash
   playwright-cli eval "JSON.stringify(Array.from(document.querySelectorAll('.slide')).map(function(s,i){return {slide:i+1, id:s.id}}))"
   ```
4. Screenshot each slide individually by scrolling to it:
   ```bash
   playwright-cli eval "document.querySelector('#s1').scrollIntoView()"
   playwright-cli screenshot --filename=data/screenshots/slide1.png
   # repeat for each slide
   ```
5. Read each screenshot image to check for: content overflow, chart sizing, broken layouts, missing images
6. Fix any render-breaking issues before proceeding to the design audit

**Step 6: Design Audit & A/B Testing**

This is an independent aesthetic review of the finished deck. The goal is to catch design issues that a render check misses -- contrast problems, font sizes below the presentation minimum, wasted whitespace, visual hierarchy failures, and CSS bugs.

**Audit procedure:**

1. **Read every screenshot** from Step 5 as images. For each slide, evaluate against this checklist:

   | Dimension | What to check |
   |---|---|
   | **Contrast** | Text on colored backgrounds must be readable. Check header badges, callout text, links on dark/colored surfaces. WCAG AA minimum (4.5:1 for body text, 3:1 for large text). |
   | **Font floor** | Body text must meet the minimum set in the storyboard or style guide (typically 18px+ for presentations). Measure by reading inline `font-size` styles. Common offenders: table cells, card bullets, footnotes. |
   | **Whitespace** | No slide should have large empty areas below content. Tables, charts, and cards should fill their containers. Use `flex: 1` or increased padding to distribute space. |
   | **CSS correctness** | Grep for common bugs: `border-radius: 50` (missing `%`), unclosed tags, `position: absolute` elements that could overlap. |
   | **Visual hierarchy** | The most important information on each slide should be the largest/boldest element. Numbers the audience needs to remember should be in stat boxes, not buried in paragraphs. |
   | **Color consistency** | Same data category = same color across all slides. Check that the style guide palette is applied consistently. |

2. **Read the HTML source** to verify inline styles match the design intent. Grep for font sizes below the minimum:
   ```bash
   # Find all font-size declarations below 18px (adjust threshold to match style guide)
   grep -oP 'font-size:\s*\d+px' presentation.html | sort | uniq -c | sort -n
   ```

3. **Write a slide-by-slide review** with findings structured as:
   - **Good:** What works well on this slide (anchor points to preserve)
   - **Issues:** Specific problems with the dimension they violate
   - **Priority fix list:** Top 5 improvements ranked by visual impact

4. **If issues are found, create `presentationB.html`:**
   - Copy the original: `cp presentation.html presentationB.html`
   - Apply all fixes from the priority list to the B variant
   - Do NOT modify the original `presentation.html` -- it stays as-is for comparison

5. **Screenshot the B variant** using the same per-slide procedure from Step 5:
   ```bash
   playwright-cli goto http://localhost:8765/presentationB.html
   # screenshot each slide as slideB1.png, slideB2.png, etc.
   ```

6. **Present the A/B comparison to the user.** For each slide that changed, show:
   - The A screenshot (original) and the B screenshot (updated)
   - A table summarizing what changed and why
   - A verdict on which is better

7. **Let the user decide:**
   - **Pick B:** Rename `presentationB.html` to `presentation.html` (replace the original)
   - **Pick A:** Delete `presentationB.html`, keep the original
   - **Cherry-pick:** User specifies which fixes to keep; apply only those to the original

**Common fixes this audit catches:**
- Low-contrast text on colored headers (swap to white text or add a frosted badge background)
- Font sizes below the presentation minimum (15-16px table cells that should be 17-18px)
- Slides with content crammed into the top half and empty space below (add `flex: 1` to tables, increase row padding)
- CSS unit bugs (`border-radius: 50` instead of `50%`, missing `px` units)
- Key numbers buried in text instead of elevated into stat boxes
- Inconsistent card/table sizing between slides that should feel uniform

**Step 7: Export to PDF**

**CRITICAL: Do NOT use `npx playwright pdf`** -- it does not support `printBackground` and will silently strip all background colors, gradients, and dark sections from the PDF. Use a Python script instead:

```python
# export_pdf.py
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto('http://localhost:8765/presentation.html', wait_until='networkidle')
    page.pdf(
        path='presentation.pdf',
        width='17in',
        height='11in',
        margin={'top': '0', 'right': '0', 'bottom': '0', 'left': '0'},
        print_background=True,
        prefer_css_page_size=True,
    )
    browser.close()
```

If Python Playwright is not installed, run `pip install playwright` first. The browsers from `npx playwright` are shared and do not need to be reinstalled.

**Required print CSS in the HTML** -- both the `@page` rule AND `print-color-adjust: exact` are needed. Add this to the presentation stylesheet:
```css
/* Force backgrounds to render in print/PDF */
* {
  -webkit-print-color-adjust: exact !important;
  print-color-adjust: exact !important;
  color-adjust: exact !important;
}

@page { size: 17in 11in; margin: 0; }
@media print {
  body { background: white !important; margin: 0; padding: 0; width: 17in; gap: 0; }
  section.slide {
    margin: 0;
    box-shadow: none;
    zoom: 0.85;
    page-break-after: always;
  }
}
```

Verify the PDF has the correct page count (one page per slide). If a slide spills, the content needs to be trimmed or the layout adjusted -- never let slides overflow. Also verify the PDF file size is comparable to the HTML -- if the PDF is suspiciously small (e.g., 300KB vs 900KB expected), backgrounds are likely being stripped.

**Step 8: Present deliverables to user**

Tell the user what was produced:
- `presentation.html` -- open in browser for full-resolution viewing (the final version after A/B resolution)
- `presentation.pdf` -- for presenting and sharing
- `storyboard.md` -- content reference (if created)
- `style_guide.md` -- visual specs (if created)
- `graphics/` -- chart PNG assets and source scripts
- `data/screenshots/` -- per-slide screenshots (slide1.png ... slideN.png, and slideB*.png if A/B test was run)

## Design Principles

These principles separate a professional deck from a default-looking one:

**1. Story-driven structure.** Every slide advances a narrative. The audience should feel momentum building. Use a recurring visual element (timeline, progress bar) to show where they are in the story.

**2. Charts as heroes.** When a slide has a chart, the chart should dominate the space. Don't shrink charts to make room for text -- the chart IS the content. Use annotations on the chart itself rather than surrounding text to explain insights.

**3. Consistent visual language.** Same data series = same color everywhere. Same component type = same styling. The audience should never have to re-learn the visual system.

**4. Progressive disclosure.** Don't put everything on one slide. Layer information across slides. The first mention of a concept gets a simple visual; later slides can show more detail.

**5. White space is structure.** Don't fill every pixel. Padding, margins, and gaps create visual hierarchy. A slide with breathing room looks more professional than one crammed with content.

**6. Typography hierarchy.** Title > Section header > Body > Caption. Each level should be visually distinct through size, weight, and color. Presentation text should be MUCH larger than document text (20px+ body, 50px+ titles at 1920px width).

## Tooling

The skill uses tools that are commonly available -- no exotic dependencies:

| Tool | Purpose | Likely Status |
|---|---|---|
| matplotlib | Data charts (pie, bar, line, dual-axis) | Usually installed |
| Playwright (npx) | Screenshots for slide verification | Check with `npx playwright --version` |
| Playwright (Python) | PDF export with `print_background=True` | `pip install playwright` -- required for correct PDF colors |
| PIL/Pillow | Image cropping for slide-by-slide verification | Usually installed with matplotlib |
| Python http.server | Local server for Playwright | Built into Python |
| Recraft V3 API | AI-generated illustrations (characters, objects, icons) | Optional -- needs API key in `.env` |

**Fallback chain:**
- If Recraft API key is available and user wants illustrations -> use Recraft V3
- If no Recraft but illustrations are needed -> use inline SVG (Claude-generated geometric figures)
- If matplotlib is not installed -> use inline SVG for charts too
- If Python Playwright is not installed -> `pip install playwright` (shares browsers with npx version)
- If Playwright is not available at all -> user prints to PDF from browser manually (Ctrl+P, enable "Background graphics")

## Output Structure

```
project/
  storyboard.md                # Content (created or provided)
  style_guide.md               # Visual specs (created or provided)
  presentation.html            # The presentation (primary deliverable)
  presentationB.html           # A/B variant with design fixes (if audit found issues)
  presentation.pdf             # PDF export (for presenting)
  graphics/
    src/
      chart_style.py            # Shared matplotlib config
      [chart_name].py           # One script per chart
      generate_recraft.py       # Recraft API generation script (if used)
    [chart_name].png            # Generated chart images
    recraft_[name].png          # AI-generated illustrations (if Recraft used)
  data/
    screenshots/
      slide1.png ... slideN.png   # Per-slide screenshots (A version)
      slideB1.png ... slideBN.png # Per-slide screenshots (B version, if A/B test run)
```

## Common Slide Types

Reference these patterns when building slides:

**Title slide:** Flex layout, centered, dark gradient background, brand watermark, placeholders for speaker names and date.

**Content + Sidebar:** Grid `1fr 400px`. Left column has stats row + body content + chart. Right column has a styled callout box. Bottom spans full width for audience question or key takeaway.

**Full-width chart:** Grid `1fr` (no sidebar). Chart fills the entire content area. Minimal text -- just a title and one insight sentence.

**Two-chart comparison:** Grid `1fr 1fr` inside the content area. Two charts side by side with a shared legend or complementary data.

**Matrix/Framework:** CSS Grid for 2x2 or 3x3 layouts with colored cells, axis labels, and examples in each quadrant.

**Summary/Closing:** Full-width layout with a key quote, recap bullets, and a fully-lit progress timeline showing the complete journey.

---
> Source: [sk2977/presentation-builder](https://github.com/sk2977/presentation-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
