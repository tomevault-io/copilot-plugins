## marp-ai-slide-generator

> This file provides guidance to Gemini when working with code in this repository.

# GEMINI.md

This file provides guidance to Gemini when working with code in this repository.

## Project Overview

This is a Marp (Markdown Presentation) template repository for creating professional presentations. Marp converts Markdown files into HTML, PDF, or PPTX presentations with custom styling.

## Flexible Task Handling

This directory may be used for tasks beyond slide creation. Respond flexibly based on user instructions:

### Supported Task Types

1. **Slide Creation**: Creating presentation materials using Marp (primary purpose)
2. **Planning Documents**: Creating proposals and outlines for presentations
3. **Research**: Gathering information needed for slide creation
4. **Review**: Reviewing and suggesting improvements to existing slides
5. **Other Documents**: Meeting notes, reports, and related documentation

### Decision Criteria

- Keywords like "slide", "presentation", "Marp" → Slide creation task
- Keywords like "plan", "outline", "idea", "proposal" → Planning document task
- If unclear, ask the user before starting

## Slide Creation Workflow (Recommended)

When creating slides, use the **generator (JSON→Markdown conversion) workflow** as the standard approach.

### Recommended Workflow: JSON + Generator

1. **Create slide structure in JSON format**
   - Use existing templates in `generator/templates/`
   - Define `type` and `data` for each slide

2. **Generate Markdown with Generator**
   ```bash
   node generator/md_build.js <input.json> [output.md]
   ```

3. **Fine-tune Markdown if needed**
   - Edit the generated Markdown directly for adjustments

4. **Convert to HTML/PDF/PPTX**

### Why Use the Generator

- **Reusability**: Leverage templates in `generator/templates/`
- **Consistency**: Maintain unified design system
- **Efficiency**: Generate large numbers of slides efficiently
- **Maintainability**: Update slides by modifying JSON only

### Available Templates

Over 30 templates available in `generator/templates/`:

See `generator/README.md` for details.

## Adding New Components/Templates

Follow these steps and rules when adding new slide layouts.

### Files to Create

| File | Location | Purpose |
|------|----------|---------|
| HBS Template | `generator/templates/<name>.hbs` | Generator template |
| Component | `generator/templates/<name>.md` | Preview/documentation |

### Component (.md) Requirements

1. **Include Marp frontmatter** (for preview capability)
   ```markdown
   ---
   marp: true
   theme: deskrex
   size: 16:9
   paginate: true
   header: "Header Text"
   footer: "@your-handle"
   ---
   ```

2. **Include JSON structure example in comments**
   ```markdown
   <!--
     Component name and description

     JSON structure example:
     {
       "type": "template-name",
       "data": { ... }
     }
   -->
   ```

3. **Separate multiple patterns with `---`**
   - Show variations as multiple slides

### Naming Conventions

- **Use generic names**
  - Good: `image-grid`
  - Bad: `client-logos-grid` (too specific)
- **Use kebab-case**: `three-step-process`, `image-grid`

### Directory Structure Rules

- **Place directly under `generator/templates/`** (no subdirectories)
  - Good: `generator/templates/image-grid.md`
  - Bad: `generator/templates/layouts/image-grid.md`

### Post-Addition Tasks

1. **Update `generator/README.md`**: Add to template list
2. **Verify**: Preview component display in Marp

### JSON Structure Example

```json
{
  "meta": {
    "theme": "deskrex",
    "header": "Presentation Title",
    "footer": "@your-handle"
  },
  "slides": [
    {
      "type": "title-cover",
      "data": {
        "mainTitle": "Main Title",
        "subtitle": "Subtitle"
      }
    },
    {
      "type": "section-divider",
      "data": {
        "title": "Chapter 1",
        "subtitle": "Section Name"
      }
    }
  ]
}
```

## Common Commands

### Generate Slides with Generator (Recommended)

```bash
# Generate Markdown from JSON (no timestamp, default)
node generator/md_build.js <input.json>
# → Generates input.marp.md

# Output with timestamp
node generator/md_build.js <input.json> --timestamp
# → Generates input_20251215_120000.md

# Specify output path
node generator/md_build.js slides.json output/presentation.md
```

### Split JSON Management and Merging

Large presentations can be split into sections for management.

**File naming convention**: `part_XX_section-name.json` (XX is a number)

```
projects/my-presentation/
├── part_00_intro.json           # Merge target
├── part_01_about.json           # Merge target
├── part_02_main.json            # Merge target
├── part_03_closing.json         # Merge target
├── config.json                  # Excluded (doesn't start with part_)
└── slides.json                  # ← Merge output destination
```

**Merge commands**:
```bash
# Merge part_XX_*.json and generate Markdown
node generator/merge-json.js <project-dir> --build

# Specify output filename
node generator/merge-json.js <project-dir> --output <name> --build

# Example
node generator/merge-json.js ./projects/my-presentation --output slides --build
```

**Benefits**:
- Edit sections independently
- Easier to manage large presentations
- Auto-sorted by number and merged

### Presentation Conversion

```bash
# Convert to PDF (using script, recommended)
# Image compression + Ghostscript compression applied automatically
./generator/pdf_build.sh <input.md or folder>

# Convert to PDF without compression
./generator/pdf_build.sh --no-compress <input.md>

# Convert to PPTX (using script, recommended)
./generator/pptx_build.sh <input.md or folder>

# Direct commands
npx @marp-team/marp-cli@latest [input.md] -o [output.html]
npx @marp-team/marp-cli@latest [input.md] --pdf -o [output.pdf]
npx @marp-team/marp-cli@latest [input.md] --pptx-editable -o [output.pptx]
```

### PDF to Markdown Conversion (OCR)

Convert image-based PDFs (scanned PDFs, NotebookLM PDFs, etc.) to Markdown. Uses Google Cloud Vision API for OCR.

```bash
python3 scripts/pdf_to_markdown.py <pdf_path>
```

**Dependencies**:
- `pip3 install pypdf pdf2image google-cloud-vision`
- poppler (`brew install poppler`)
- Google Cloud auth (`~/.config/gcloud/application_default_credentials.json`)

**Output**: Generates `.md` file in the same directory

### PDF Compression Features

`pdf_build.sh` includes automatic image compression:

| Process | Description |
|---------|-------------|
| PNG/JPG | Convert to JPEG at 30% quality (size maintained) |
| SVG | Keep as-is (quality preserved) |
| Ghostscript | Auto-compress PDF if over 20MB |

**Example**: 100MB → 10MB (about 90% reduction)

Compression settings can be changed at the top of `generator/pdf_build.sh`:
```bash
COMPRESS_QUALITY=30  # JPEG quality (0-100)
```

## Architecture and Structure

### Main Directories

- **`assets/`**: Images and media files organized by type
- **`generator/templates/`**: Reusable Marp components (HBS templates and preview MD files)

### Template System

This repository uses a component-based approach with templates in `generator/templates/`.

1. **HBS Templates** (`generator/templates/*.hbs`): Handlebars templates that generate Markdown from JSON.
2. **Preview MD** (`generator/templates/*.md`): Sample files that can be previewed directly in Marp.

### Typical Presentation Structure

```markdown
---
marp: true
size: 16:9
paginate: true
theme: gaia
backgroundColor: white
style: |
  /* Custom styles */
  @import url('https://cdn.tailwindcss.com/3.0.0');
---

<!-- _paginate: skip -->
# Title Slide

---

# Content Slide...
```

## Design System

Follow these design principles. Avoid overly colorful styling that ignores design consistency.

- **Primary Colors**: Blue palette (#2563eb, #3b82f6, #60a5fa)
- **Secondary Colors**: Green palette (#059669, #10b981, #34d399)
- **Typography**: Use Tailwind CSS classes for consistent text styling
- **Layout**: Pre-built templates for common slide patterns (split layouts, grids, comparisons)

### Font Awesome Icons

When using `theme: deskrex`, Font Awesome 6 icons are available (built into the theme).

#### Basic Usage

```html
<!-- Solid -->
<i class="fa-solid fa-house"></i>
<i class="fa-solid fa-check"></i>
<i class="fa-solid fa-gear"></i>

<!-- Regular -->
<i class="fa-regular fa-bell"></i>
<i class="fa-regular fa-envelope"></i>

<!-- Brands -->
<i class="fa-brands fa-github"></i>
<i class="fa-brands fa-slack"></i>
```

#### Sizing

```html
<i class="fa-solid fa-rocket fa-xs"></i>   <!-- Extra small -->
<i class="fa-solid fa-rocket fa-sm"></i>   <!-- Small -->
<i class="fa-solid fa-rocket fa-lg"></i>   <!-- Large -->
<i class="fa-solid fa-rocket fa-xl"></i>   <!-- Extra large -->
<i class="fa-solid fa-rocket fa-2x"></i>   <!-- 2x -->
<i class="fa-solid fa-rocket fa-3x"></i>   <!-- 3x -->
```

#### Color

```html
<!-- Use Tailwind classes for colors (recommended) -->
<i class="fa-solid fa-check fa-lg text-green-600"></i>
<i class="fa-solid fa-xmark fa-lg text-red-600"></i>
<i class="fa-solid fa-star fa-lg text-amber-500"></i>
```

**Note**: Inline styles like `style="color: ...;"` **do not work**. Always use Tailwind classes (`text-*`) for colors.

#### Icon Search

Search available icons at [Font Awesome Icons](https://fontawesome.com/icons).

### Mermaid Diagrams

Mermaid allows generating flowcharts, sequence diagrams, and more from text.

#### Setup

Add this script at the beginning of your slide (after frontmatter):

```html
<script type="module">
  import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.esm.min.mjs';
  mermaid.initialize({ startOnLoad: true });
</script>
```

#### Basic Usage

```html
<div class="mermaid">
graph TD;
    A[Start] --> B{Decision};
    B -->|Yes| C[Action A];
    B -->|No| D[Action B];
    C --> E[End];
    D --> E;
</div>
```

#### Available Diagrams

| Type | Syntax | Use Case |
|------|--------|----------|
| Flowchart | `graph TD` / `graph LR` | Process flows |
| Sequence | `sequenceDiagram` | System interactions |
| Class | `classDiagram` | Object design |
| State | `stateDiagram-v2` | State changes |
| ER | `erDiagram` | Database design |
| Gantt | `gantt` | Schedules |
| Pie | `pie` | Proportions |
| Mindmap | `mindmap` | Idea organization |
| Git Graph | `gitGraph` | Branch/commit history |
| User Journey | `journey` | User experience |
| Quadrant | `quadrantChart` | 2-axis classification |
| XY Chart | `xychart-beta` | Bar/line charts |

See [Mermaid Documentation](https://mermaid.js.org/intro/) for details.

### Basic Markdown Rules

- **Slide separator:** `---`
- **Title:** `# Slide Title`
- **Section:** `## Section Title`
- **Bullet points:**
  ```markdown
  - Item 1
    - Sub-item 1
  - Item 2
  ```

### Important Notes

- **Absolute Positioning (Important):** In Marp, the `<section>` tag has `position: relative`. When positioning images at slide edges, use `absolute` directly without wrapping in `relative`.
  - **Recommended (positioning at slide edge):**
    ```html
    <!-- Use absolute directly without relative -->
    <div class="absolute right-0 top-0">
      <img src="image.jpg" alt="Image" class="w-full h-full object-cover">
    </div>
    ```
  - **Bad example (shifts inside slide):**
    ```html
    <!-- Wrapping in relative makes the inside the reference point -->
    <div class="relative h-full">
      <div class="absolute top-0">
        <img src="image.jpg" alt="Image" class="w-full h-full object-cover">
      </div>
    </div>
    ```
  - Reason: In most Marp themes, `<section>` has `position: relative`, so you don't need to write `relative` for "slide-based" positioning

- **CSS Styling Limitations (Important):** Marp ignores **all** inline styles via `style=""` attribute (including icons). Only CSS classes in `theme/deskrex.css` work.
  - Bad: `<div style="width: 60%;">` → **Does not work**
  - Bad: `<i class="fa-solid fa-star" style="color: #f59e0b;"></i>` → **Does not work**
  - Good: `<div class="w-3/5">` or `<div class="grid grid-cols-[60%_40%]">`
  - Good: `<i class="fa-solid fa-star text-amber-500"></i>`
  - If you need new styles, add classes to `theme/deskrex.css`

- **Images:** Refer to image templates in `generator/templates/` like `image-grid.md`, `centered-image.md`

- **Image Generation Integration (Important):** When images or diagrams would be effective, consider using the `nano-banana` skill for AI image generation. See `.claude/skills/nano-banana/SKILL.md` for details.
  - **Target templates**: `centered-image`, `single-case-detailed-split-with-image`, `image-overlay-captioned-photo`, etc.
  - **Placeholder method**: If actual image path isn't available yet, write detailed description in `alt` attribute
  - **Example**:
    ```json
    {
      "type": "centered-image",
      "data": {
        "title": "Future of AI",
        "image": {
          "src": "placeholder.png",
          "alt": "[Generate] Futuristic office with AI robots and humans collaborating, bright natural light, minimal design"
        }
      }
    }
    ```
  - **Workflow**:
    1. Write "[Generate] + image description" in `alt` to create placeholder slide
    2. Generate image using `.claude/skills/nano-banana/SKILL.md` (uses Gemini API)
    3. Update `src` with generated image path
  - **Effective slides**: Concept explanations, vision presentations, case studies, metaphorical expressions - slides where visual impact matters

- **Citations:** When quoting URLs or external data, always use format: "Source: [Source Name](URL)"

- **Font Sizes**: In Marp, heading tags (H1, H2, H3, etc.) have fixed font sizes. Adjust text size using `<p>` tags with appropriate TailwindCSS classes based on content length.
  - Few items: Use `text-2xl`, `text-xl`
  - Many items: Use `text-lg`, `text-base`
  - Very many items: Use `text-sm`, `text-xs`

- **Size/Spacing Guidelines (Important)**: Slides have limited 16:9 space. Elements that are too large will overflow. Use these guidelines:
  - **Font size**: Title `text-2xl`, heading `text-base`~`text-sm`, body `text-2xs`~`text-xs`, notes `text-3xs`
  - **Padding**: Inside boxes `p-3`~`p-4` (not `p-6` or larger)
  - **Gap**: Between elements `gap-2`~`gap-4`
  - **Icons**: `w-10 h-10`~`w-12 h-12` (not `w-16` or larger)
  - **Box width**: Around `w-40`~`w-52`
  - **Margin**: `mb-1`~`mb-4`
  - Reference: Check sizing in `generator/templates/four-quadrant-service-scope.md`

- Avoid blank lines between code blocks within a single slide, as Marp interprets empty lines as paragraph breaks.
  - Bad example (has blank line):
    ```
      </div>

      <div class="bg-white p-3 rounded border-l-4 border-green-400">
    ```
  - Good example (no blank line):
    ```
      </div>
      <div class="bg-white p-3 rounded border-l-4 border-green-400">
    ```

- There's a limit to how much content fits on one slide. If content doesn't fit, actively split with `---` separator.

---
> Source: [itarutomy97/marp-ai-slide-generator](https://github.com/itarutomy97/marp-ai-slide-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
