## agent4ppt

> Interactively revise slides via 3-step interview workflow


# agent4ppt

> **Claude Code plugin** — Parse PPTX templates into markdown, generate PPTX from markdown, and interactively revise slides.
> Compatible with **Claude Code**, **OpenClaw**, and **Codex CLI**.

---

## Installation

```bash
claude plugin marketplace add JinwangMok/agent4ppt
claude plugin install agent4ppt
```

The installer runs `scripts/install.sh` which checks Python ≥ 3.10, installs the three required packages (`python-pptx`, `pyyaml`, `markdown-it-py`), and reports the optional LibreOffice dependency for PNG previews.

### Manual dependency installation

```bash
bash scripts/install.sh
# or directly:
pip install "python-pptx>=0.6.21" "pyyaml>=6.0" "markdown-it-py>=3.0"
```

---

## Skills overview

| Skill | Invocation | Purpose |
|-------|-----------|---------|
| [parse-ppt-template](#parse-ppt-template) | `/parse-ppt-template <pptx_file>` | Parse a PPTX template → markdown template |
| [generate-ppt](#generate-ppt) | `/generate-ppt <markdown_file>` | Generate PPTX from a markdown file |
| [revise-ppt](#revise-ppt) | `/revise-ppt <markdown_file>` | Interactively revise slides (3-step workflow) |

**Markdown is the single source of truth.** The recommended workflow is:

```
PPTX template ──[parse-ppt-template]──► markdown template
                                               │
                              (edit content)   ▼
                               markdown file ──[generate-ppt]──► PPTX output
                                               │
                              (need changes)   ▼
                               markdown file ──[revise-ppt]──► revised PPTX
```

---

## Core design principles

### Canonical document model

Markdown is the **canonical document representation**, not a serialisation format. This means:

- All content edits are made in markdown first; the PPTX is always a derived artifact.
- The `template:` field in YAML frontmatter is the **sole authoritative reference** to the source PPTX. There is no second path to the template.
- `raw_text` inside a placeholder is always **derived from** the structured markdown representation and never treated as an independent field.

### Template contract vs. spatial layout

The placeholder annotation encodes two distinct concerns:

| Concern | Annotation | Meaning |
|---------|-----------|---------|
| **Template contract** | `type:TYPE` | What a placeholder *demands* — the kind of content it accepts (`title`, `body`, `picture`, `chart`, `table`, …). This is the editable semantic contract. |
| **Spatial layout** | `pos:(x,y) size:WxH` | Where the placeholder *lives* on the slide. Informational metadata; preserved passthrough, never independently edited. |

### Passthrough semantics

The following slide properties are **preserved passthrough** — they are neither modelled in markdown nor destroyed during PPTX generation:

- Slide transitions and animations
- Speaker notes
- Master slide properties and theme assets
- Custom XML extensions not relating to content placeholders

---

## Bilingual support

All three skills auto-detect language from the `LANG` environment variable (POSIX locale format, e.g. `ko_KR.UTF-8`).

| `LANG` value | Resolved language |
|---|---|
| `ko`, `ko_KR`, `ko_KR.UTF-8`, … | Korean (`ko`) |
| anything else | English (`en`, default) |

Override per-invocation with the `--lang ko|en` flag. The `lang` field in markdown frontmatter takes the highest precedence for `generate-ppt` and `revise-ppt`.

---

## parse-ppt-template

Parse a PPTX template file and generate a markdown template with placeholder annotations. Placeholders are identified by python-pptx `idx`-based indexing and annotated with `<!-- ph:idx type:TYPE pos:(x,y) size:WxH -->` HTML comments. After writing the output file the skill runs an **automated verification step** that confirms structural invariants without manual inspection.

### Interface

```
/parse-ppt-template <pptx_file> [--output <output.md>] [--lang ko|en] [--layouts-only]
```

Executed via: `python skills/parse-ppt-template/parse_template.py`

### Parameters

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `pptx_file` | ✅ | — | Path to the source PPTX template file |
| `--output`, `-o` | ❌ | `<input>.md` | Output path for the generated markdown template |
| `--lang`, `-l` | ❌ | `$LANG` or `en` | Language for guide comments in the output (`ko` \| `en`) |
| `--layouts-only` | ❌ | `false` | Generate one section per layout definition instead of per slide. Useful when the template has no slide content yet. |

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `pptx_file` | File path | A valid PPTX file readable by python-pptx. Must exist on disk. |

### Returns / Output artifacts

A markdown file written to `--output` (or `<input>.md` by default).

#### Exit codes

| Code | Meaning |
|------|---------|
| `0` | Success — output written and all verification checks passed |
| `1` | Input / IO error (file not found, missing dependency, unreadable PPTX) |
| `2` | Verification failure — output was written but failed one or more structural invariant checks |

**Success markers** (printed to stdout, machine-checkable):
```
[agent4ppt] Markdown template written → <path>
[agent4ppt] Verification passed ✓  (<N> section(s), template_path confirmed, idx annotations present)
```

### Automated verification (built-in)

After writing the output file the skill automatically verifies three structural invariants:

| # | Invariant | Check |
|---|-----------|-------|
| 1 | **YAML frontmatter `template` key** | Frontmatter exists, contains `template:` key, value is non-empty. This is the sole authoritative reference to the source PPTX. |
| 2 | **Separator count** | Number of `---` lines after frontmatter == `slide_count - 1` (one separator between each consecutive section, none before the first or after the last). |
| 3 | **Placeholder idx annotations** | Every non-empty, non-metadata-only section contains at least one `<!-- ph:N type:TYPE … -->` annotation. Layout sections with zero placeholders are exempt. |

Verification failure yields exit code `2` (distinguishable from IO errors at exit code `1`).

### Verifiable artifacts

- The output markdown file exists at the declared path
- Contains `---` YAML frontmatter with `template:` field pointing back to the source PPTX
- Contains one or more slide sections separated by `---`
- Each substantive slide section includes at least one `<!-- ph:\d+ type:\w+ -->` comment
- The skill's own verification message (`[agent4ppt] Verification passed ✓`) is present in stdout

**Artifact validation commands:**
```bash
# Run the skill and capture output
python skills/parse-ppt-template/parse_template.py template.pptx --output out.md > parse_out.txt 2>&1
echo "Exit: $?"                                                    # Must be 0

# Verify output file structure
grep -q "^template:" out.md            && echo "PASS: frontmatter present"
grep -qP "ph:\d+ type:\w+" out.md      && echo "PASS: placeholders found"

# Verify built-in verification message was printed to stdout
grep -q "Verification passed" parse_out.txt && echo "PASS: built-in verification passed"
```

### Output format

The generated markdown file contains:

- **YAML frontmatter** with `template`, `fname`, `title`, `subTitle`, `date`, `author`, `lang` fields
- **Slide sections** separated by `---` (one section per slide or layout, depending on `--layouts-only`)
- **Layout indicator** `> layout: N` at the top of each slide section
- **Layout name comment** `<!-- layout_name: NAME -->` (informational)
- **Placeholder annotations** `<!-- ph:idx type:TYPE pos:(x,y) size:WxH -->` with content ready to fill
- **Guide comments** `<!-- Enter the ... -->` (in the selected language) explaining each placeholder

#### Placeholder type values

| Type | python-pptx placeholder kind |
|------|-------------------------------|
| `title` | Title |
| `center_title` | Center Title |
| `subtitle` | Subtitle |
| `body` | Body |
| `object` | Object (generic) |
| `picture` | Picture |
| `table` | Table |
| `chart` | Chart / diagram |
| `media` | Media clip |
| `ftr` | Footer |
| `dt` | Date |
| `sldnum` | Slide number |

#### Ontology: template contract vs. spatial layout

Each placeholder annotation encodes two distinct concerns in a single line:

```
<!-- ph:0 type:title pos:(0.4583",0.25") size:8.5"×1.2" -->
         ^^^^^^^^^^^                                       ← Template contract: what content type is demanded
                      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  ← Spatial layout: where it lives (informational)
```

The `type:` field is the **template contract** — it determines which markdown syntax is valid for this placeholder (headings for `title`/`subtitle`/`center_title`, bullets for `body`, GFM table for `table`, chart YAML for `chart`, image path for `picture`).

The `pos:` and `size:` fields are **spatial layout** — they document the placeholder's geometry on the slide. These are preserved passthrough and must not be independently edited.

### Example output

```markdown
---
template: ./my_template.pptx
fname: output.pptx
title: ""
subTitle: ""
date: ""
author: ""
lang: en
---

> layout: 0
<!-- layout_name: Title Slide -->

<!-- ph:0 type:center_title pos:(0.9167",1.75") size:7.6548"×0.7547" -->
<!-- Enter the centered slide title -->
# Slide Title

<!-- ph:1 type:subtitle pos:(0.9167",2.6495") size:5.588"×0.8832" -->
<!-- Enter the subtitle -->
Subtitle text here

---

> layout: 1

<!-- ph:0 type:title pos:(0.4583",0.25") size:8.5"×1.2" -->
<!-- Enter the slide title -->
# Section Header

<!-- ph:1 type:body pos:(0.4583",1.5") size:8.5"×4.5" -->
<!-- Enter the main body content. Supports bullets, bold, italic. -->
- Bullet one
- Bullet two
```

### Error handling

| Condition | Behaviour |
|-----------|-----------|
| `pptx_file` not found | Exits 1, prints error to stderr |
| File has no `.pptx` extension | Warning printed to stderr, continues |
| PPTX file is corrupt or unreadable | Exits 1, prints error to stderr |
| Missing dependency (`python-pptx`, `pyyaml`) | Exits 1, prints install instructions to stderr |
| Verification invariant fails | Exits 2, prints which invariants failed to stderr |

### Examples

```bash
# Basic — output written to same directory as template
/parse-ppt-template "GIST NetAI PPT Theme.pptx"

# Specify output path and Korean guide comments
/parse-ppt-template "GIST NetAI PPT Theme.pptx" --output slides_template.md --lang ko

# One section per layout (for templates with no slide content)
/parse-ppt-template my_template.pptx --layouts-only --output layouts.md
```

### Dependencies

- `python-pptx >= 0.6.21`
- `pyyaml >= 6.0`

---

## generate-ppt

Generate an editable PPTX file from a markdown file. Reads the PPTX template path from the markdown frontmatter (`template:` field — the sole authoritative reference to the source PPTX), maps each `<!-- ph:idx -->` block to the corresponding placeholder by `idx`, and converts markdown tables, chart YAML blocks, and images to native python-pptx objects. Content-placeholder mismatches are handled gracefully with per-warning output.

### Interface

```
/generate-ppt <markdown_file> [--output <output.pptx>] [--lang ko|en]
```

Executed via: `python skills/generate-ppt/generate_ppt.py`

### Parameters

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `markdown_file` | ✅ | — | Path to the markdown content file |
| `--output`, `-o` | ❌ | `frontmatter.fname` or `<input>.pptx` | Output PPTX file path |
| `--lang`, `-l` | ❌ | `$LANG` or `en` | Language for warning/info messages (`ko` \| `en`) |

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `markdown_file` | File path | Markdown file with YAML frontmatter. Must contain a `template:` field pointing to the source PPTX. |
| `template` (frontmatter) | PPTX path | The source PPTX template. Resolved relative to the markdown file's directory. This is the sole authoritative reference — no secondary path exists. |

### Returns / Output artifacts

A PPTX file at the resolved output path.

#### Exit codes

| Code | Meaning |
|------|---------|
| `0` | Success — PPTX written |
| `1` | Error (file not found, missing `template:` field, dependency missing, etc.) |

**Success marker** (printed to stdout, machine-checkable):
```
[agent4ppt] PPTX generated → <path>
```

### Verifiable artifacts

- The output PPTX file exists at the declared path and is a valid ZIP/PPTX structure
- Slide count matches the number of `---`-separated sections in the markdown
- Each slide in the PPTX contains text extracted from the corresponding markdown section

**Artifact validation commands:**
```bash
python skills/generate-ppt/generate_ppt.py out.md --output out.pptx
echo "Exit: $?"      # Must be 0
python -c "
from pptx import Presentation
import re
p = Presentation('out.pptx')
md = open('out.md').read()
body = md.split('---\n', 2)[-1]
slides_md = len(re.split(r'\n---\s*\n', body.strip()))
assert len(p.slides) == slides_md, f'Mismatch: pptx={len(p.slides)} md={slides_md}'
print('PASS: slide count matches')
"
```

### Markdown format

The markdown file **must** begin with YAML frontmatter.

#### Frontmatter fields

| Field | Required | Description |
|-------|----------|-------------|
| `template` | ✅ | Relative or absolute path to the PPTX template. Relative paths are resolved from the markdown file's directory. This is the sole authoritative reference to the source PPTX. |
| `fname` | ❌ | Default output filename (e.g. `output.pptx`). Overridden by `--output`. |
| `title` | ❌ | Presentation title (metadata) |
| `subTitle` | ❌ | Presentation subtitle (metadata) |
| `date` | ❌ | Presentation date string |
| `author` | ❌ | Author name |
| `lang` | ❌ | Language override (`en` or `ko`). Takes precedence over `$LANG` and `--lang`. |

#### Slide section structure

Each slide section is separated by `---` and may contain:

```markdown
> layout: N               ← layout index (0-based), required

<!-- ph:0 type:title -->  ← placeholder annotation (idx=0, type=title)
# My Slide Title          ← content for this placeholder

<!-- ph:1 type:body -->
- Bullet one
- Bullet two
- **Bold text** and *italic*
```

#### Input contract per placeholder type

The `type:TYPE` annotation declares what content is expected. The generator maps markdown syntax to python-pptx objects according to the following contract:

| Placeholder type | Expected input syntax | python-pptx mapping |
|------------------|-----------------------|---------------------|
| `title`, `center_title` | `# Heading` text | Heading stripped of `#`, written as single run |
| `subtitle` | Plain text or `# Heading` | Heading stripped of `#`, written as single run |
| `body` | Bullets, numbered lists, bold, italic | Paragraph list with indent levels |
| `object` | Any inline content or `\|\|\|` multi-column | Fills first column; `\|\|\|` splits into columns |
| `picture` | `![alt](path/to/image.png)` | Image inserted into placeholder frame |
| `table` | GFM markdown table | First row → bold header; remaining rows → body |
| `chart` | ` ```chart ` YAML block | Bar, column, line, or pie chart via python-pptx Chart API |
| `media` | (not yet supported) | Placeholder left empty with warning |
| `ftr`, `dt`, `sldnum` | Plain text | Written as-is; typically auto-populated by PPTX |

Mismatches between the declared `type:` and the actual content syntax produce a warning to stderr but do not abort generation.

#### Supported content types

| Type | Syntax | Notes |
|------|--------|-------|
| Plain text | Any text | Written as-is |
| Headings | `# Title`, `## Sub` | Leading `#` stripped for title/subtitle placeholders |
| Bullet list | `- item` or `* item` | Up to 9 indent levels |
| Numbered list | `1. item` | Treated as bullets |
| Bold | `**text**` | `run.font.bold = True` |
| Italic | `*text*` | `run.font.italic = True` |
| Link | `[text](url)` | Underlined text; hyperlink relationship not inserted |
| Image | `![alt](path/to/image.png)` | For `picture` or `object` placeholders |
| Table | GFM markdown table | First row becomes bold header |
| Chart | ` ```chart ` YAML block | Bar, column, line, pie charts |
| Multi-column | `\|\|\|` separator | Splits content; first column fills the placeholder |

#### Chart block syntax

````markdown
<!-- ph:2 type:chart -->
```chart
type: bar          # bar | column | line | pie
title: My Chart
categories: [Q1, Q2, Q3, Q4]
series:
  - name: Revenue
    values: [100, 150, 120, 200]
  - name: Expenses
    values: [80, 90, 100, 110]
```
````

#### Multi-column layout

```markdown
<!-- ph:1 type:object -->
Left column content
- Item A
- Item B

|||

Right column content
- Item C
- Item D
```

### Error handling

| Condition | Behaviour |
|-----------|-----------|
| `markdown_file` not found | Exits 1, prints error to stderr |
| `template` frontmatter field missing | Exits 1, prints error to stderr |
| Template PPTX file not found | Exits 1, prints error to stderr |
| Layout index out of range | Warning printed to stderr, uses layout 0 |
| Placeholder `idx` not found in slide | Warning printed to stderr, placeholder skipped |
| Image file not found | Warning printed to stderr, placeholder left empty |
| Invalid chart YAML | Warning printed to stderr, placeholder left empty |
| Missing dependencies | Exits 1, prints install instructions to stderr |

### Examples

```bash
# Generate PPTX — output filename comes from frontmatter fname field
/generate-ppt slides.md

# Override output path
/generate-ppt slides.md --output final_presentation.pptx

# Korean messages
/generate-ppt slides.md --lang ko
```

### Full markdown example

```markdown
---
template: ./GIST NetAI PPT Theme.pptx
fname: my_presentation.pptx
title: "Research Overview"
subTitle: "GIST NetAI Lab"
date: "2026-03-22"
author: "Jinwang Mok"
lang: en
---

> layout: 0

<!-- ph:0 type:title -->
# Research Overview

<!-- ph:1 type:subtitle -->
GIST NetAI Lab · 2026

---

> layout: 2

<!-- ph:0 type:title -->
# Benchmark Results

<!-- ph:1 type:table -->
| Model | Accuracy | Latency (ms) |
|-------|----------|--------------|
| Ours  | 94.2%    | 12           |
| Base  | 88.7%    | 15           |

---

> layout: 3

<!-- ph:0 type:title -->
# Training Loss

<!-- ph:1 type:chart -->
```chart
type: line
title: Training Loss
categories: [Epoch 1, Epoch 2, Epoch 3, Epoch 4, Epoch 5]
series:
  - name: Train
    values: [1.8, 1.2, 0.9, 0.7, 0.6]
  - name: Val
    values: [2.0, 1.5, 1.1, 0.9, 0.8]
```
```

### Dependencies

- `python-pptx >= 0.6.21`
- `pyyaml >= 6.0`
- `markdown-it-py >= 3.0`

---

## revise-ppt

Interactively revise slides through a **3-step interview workflow**: select target slides, describe changes per slide, apply edits to the markdown source and regenerate the PPTX. The markdown file is the single source of truth — all changes are applied to markdown first. Optionally renders PNG slide previews via LibreOffice headless; falls back to a deterministic text artifact when LibreOffice is unavailable. Revision state is tracked in a JSON file (`<stem>_revision_state.json`) for resumable multi-round editing.

### Interface

```
/revise-ppt <markdown_file> [--pptx <pptx_file>] [--output <output.pptx>] [--lang ko|en]
```

Executed via: `python skills/revise-ppt/revise_ppt.py`

### Parameters

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `markdown_file` | ✅ | — | Path to the markdown source file to revise (the single source of truth) |
| `--pptx` | ❌ | derived from `frontmatter.fname` or `<markdown>.pptx` | Path to the existing PPTX file used for PNG preview generation |
| `--output`, `-o` | ❌ | `frontmatter.fname` or `<markdown>.pptx` | Output path for the regenerated PPTX (passed to `generate-ppt`) |
| `--lang`, `-l` | ❌ | `$LANG` or `en` | Language for interview prompts (`ko` \| `en`) |

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `markdown_file` | File path | Markdown source file to revise. Must exist and contain YAML frontmatter with a `template:` field. |
| `--pptx` | File path (optional) | Existing PPTX used only for PNG preview generation via LibreOffice. If absent or missing, LibreOffice preview is skipped and a text fallback artifact is written instead. |
| Interactive prompts | stdin | User answers to the 3-step interview (slide selection, revision instructions). |

### Returns / Output artifacts

The `markdown_file` is **updated in place** with the requested changes. A new PPTX is generated by delegating to `generate-ppt`.

#### Exit codes

| Code | Meaning |
|------|---------|
| `0` | Success (including graceful Ctrl-C and "no slides selected" exit) |
| `1` | Error (file not found, `generate-ppt` subprocess failure, unexpected exception) |

**Success markers** (printed to stdout, machine-checkable):
```
[agent4ppt] PPTX generated → <path>      ← emitted by the generate-ppt subprocess
Revision complete! Updated file: <path>   ← emitted by revise-ppt
  Markdown updated: <markdown_file>
  PPTX regenerated: <pptx_path>
```

### Verifiable artifacts

All output artifacts from `revise-ppt` are written to **deterministic, predictable paths** — no temporary directory names are used, so artifacts can be verified without prior knowledge of the run.

| Artifact | Path | When present |
|----------|------|-------------|
| Updated markdown | `<markdown_file>` (in place) | Always after Step 3 |
| Regenerated PPTX | `<output>` (from `--output` or frontmatter `fname`) | Always after Step 3 |
| Revision state JSON | `<markdown_stem>_revision_state.json` (sibling of markdown) | After Step 1 and after Step 3 |
| Step 1 previews (PNG) | `<pptx_stem>_previews/slide_N.png` | When LibreOffice is available |
| Step 1 previews (text) | `<pptx_stem>_previews/slide_N.txt` | When LibreOffice is unavailable |

**Artifact validation commands:**
```bash
# After running revise-ppt, verify completion
grep -q "Revision complete" output.txt         && echo "PASS: revision completed"
grep -q "\[agent4ppt\] PPTX generated" output.txt && echo "PASS: PPTX generated"

# Validate revision state JSON
python -c "
import json, sys
state = json.load(open('slides_revision_state.json'))
assert 'markdown_path' in state
assert 'iteration_count' in state
print('PASS: revision state valid, iteration_count =', state['iteration_count'])
"

# Validate that preview artifacts exist in deterministic path
ls my_presentation_previews/slide_1.*   # PNG or TXT — deterministic, verifiable
```

### Workflow — 3 steps

#### Step 1 — Select target slides

The skill displays a slide count and generates preview artifacts to deterministic paths. If LibreOffice is available, PNG files are written to `<pptx_stem>_previews/slide_N.png`; otherwise text summaries are written to `<pptx_stem>_previews/slide_N.txt`. Revision state is initialised / updated at the end of Step 1.

```
=== Slide Revision Workflow ===
Your presentation has 8 slide(s).

STEP 1: Which slides would you like to revise?
Enter slide numbers separated by commas (e.g. 1,3,5), a range (e.g. 2-4), or 'all':
> 2,4-6
```

**Selection syntax:**

| Input | Meaning |
|-------|---------|
| `1,3,5` | Slides 1, 3, and 5 |
| `2-4` | Slides 2, 3, and 4 |
| `2~4` | Same as `2-4` |
| `all` | All slides |
| `*` | All slides |
| `모두` | All slides (Korean alias) |

#### Step 2 — Describe changes per slide

For each selected slide, the preview artifact path (PNG or text) is displayed, then the user is prompted for revision instructions. The state file is verified readable before this step begins.

```
STEP 2: Let's go through each selected slide.

--- Slide 2 (layout: 1) ---
[Preview: /path/to/my_presentation_previews/slide_2.png]

Describe your changes. Examples:
  - Change title to 'New Title'
  - Replace bullet 2 with 'Updated content'
  - Change layout to layout 3
  - Replace chart with new data
  - Add image path/to/image.png
Your instructions for slide 2:
> Change title to 'Updated Results'
```

**Supported instruction patterns** (applied automatically):

| Pattern | Example | Effect |
|---------|---------|--------|
| Change title | `Change title to 'New Title'` | Replaces title placeholder heading text |
| Change layout | `Change layout to 3` | Updates `> layout: N` in the slide section |
| Free-form | Any other text | Appended as `<!-- REVISION: ... -->` comment for manual / Claude review |

#### Step 3 — Apply and regenerate

```
STEP 3: Applying changes and regenerating PPTX...
  Applying changes to slide 2...
  Applying changes to slide 4...
Regenerating PPTX...
[agent4ppt] PPTX generated → ./my_presentation.pptx

Revision summary (2 slide(s) modified):
  Slide 2: Change title to 'Updated Results'
  Slide 4: Change layout to 3

Revision complete! Updated file: ./my_presentation.pptx
  Markdown updated: ./slides.md
  PPTX regenerated: ./my_presentation.pptx
```

The modified markdown is written back to `markdown_file`, then `generate-ppt` is invoked as a subprocess to produce the new PPTX. After Step 3 the revision state JSON is updated with incremented `iteration_count`.

### Revision state tracking

A JSON state file `<markdown_stem>_revision_state.json` is created (or updated) after Step 1 and after Step 3.

```json
{
  "markdown_path": "/absolute/path/to/slides.md",
  "pptx_path": "/absolute/path/to/output.pptx",
  "iteration_count": 1
}
```

| Field | Type | Description |
|-------|------|-------------|
| `markdown_path` | string | Absolute path to the markdown source being revised |
| `pptx_path` | string \| null | Absolute path to the current PPTX (null if not yet generated) |
| `iteration_count` | integer | Number of completed revision cycles; increments after each successful Step 3 |

State persists across multiple invocations, allowing resumable multi-round editing. Each new invocation loads the existing `iteration_count` and increments it.

### PNG preview

When LibreOffice is installed, slide previews are generated to a **deterministic directory** `<pptx_stem>_previews/` next to the PPTX:

```bash
libreoffice --headless --convert-to png --outdir <pptx_stem>_previews/ presentation.pptx
```

If LibreOffice is not found, a plain-text `.txt` file is written to the same deterministic directory with a text summary of the slide's placeholder content. Either way the preview artifacts are at predictable, verifiable paths.

**Install LibreOffice:**

```bash
# Ubuntu / Debian
sudo apt install libreoffice

# macOS (Homebrew)
brew install --cask libreoffice
```

### Error handling

| Condition | Behaviour |
|-----------|-----------|
| `markdown_file` not found | Exits 1, prints error to stderr |
| No slides selected | Exits 0, prints advisory message |
| LibreOffice not found | Warning printed, falls back to deterministic text preview |
| `generate-ppt` subprocess fails | Stderr from subprocess printed, exits 1 |
| `KeyboardInterrupt` (Ctrl-C) | Exits 0 with `[agent4ppt] Cancelled.` |
| Missing dependencies | Exits 1, prints install instructions to stderr |

### Examples

```bash
# Standard interactive revision (auto-detects PPTX from frontmatter fname)
/revise-ppt slides.md

# Provide PPTX path explicitly for PNG preview generation
/revise-ppt slides.md --pptx final_presentation.pptx

# Specify an explicit output PPTX path
/revise-ppt slides.md --output revised_v2.pptx

# Korean prompts
/revise-ppt slides.md --lang ko
```

### Dependencies

- `python-pptx >= 0.6.21`
- `pyyaml >= 6.0`
- `markdown-it-py >= 3.0`
- `libreoffice` (optional — PNG slide preview)

---

## Complete end-to-end example

```bash
# 1. Parse the PPTX template into a markdown template
/parse-ppt-template "GIST NetAI PPT Theme.pptx" --output my_talk.md

# 2. Edit my_talk.md — fill in slide content, add tables/charts/images
#    (use your editor or ask Claude to fill in the content)

# 3. Generate the PPTX from the markdown
/generate-ppt my_talk.md

# 4. Review the result and interactively revise if needed
/revise-ppt my_talk.md
```

---

## Spec conformance verification

Each skill emits a machine-readable success marker and exits with a documented exit code. The full pipeline can be verified non-interactively without manual inspection.

### Verification override policy

> **Non-negotiable rule:** A spec verification failure is a hard failure. It must **never** be silently downgraded to a pass.

This policy governs both the skills themselves and any tooling that evaluates their output:

| Rule | Implication |
|------|-------------|
| **Non-zero exit = failure** | `exit 1` (IO error) and `exit 2` (invariant failure) are both failures. A wrapper that converts either to `exit 0` is a conformance violation. |
| **Absent success marker = failure** | If a skill claims success but `[agent4ppt] Verification passed ✓` is absent from its stdout, the run has failed — not passed with a warning. |
| **Partial invariant failure = failure** | `parse-ppt-template` verifies three structural invariants. Passing two and failing one exits `2`. It is not a "partial pass". |
| **No override tolerance** | Evaluation frameworks must not define thresholds such as "2 of 3 invariants passed → accept". Every documented invariant must pass. |
| **Regression is hard failure** | Any acceptance criterion that passed in a prior generation must pass in the current generation. A regression that is relabelled "informational" or "known issue" is a conformance override and must be treated as a failure. |

### Verification paths summary

| Skill | Criterion | Verification command | Pass condition |
|-------|-----------|----------------------|----------------|
| `parse-ppt-template` | `template` key in YAML frontmatter | `grep "^template:" out.md` | Key present, value non-empty |
| `parse-ppt-template` | Placeholder idx annotations present | `grep -P "ph:\d+ type:\w+" out.md` | At least one match |
| `parse-ppt-template` | Separator count == slide_count − 1 | Built-in verification — exit code | `$? == 0`; exits `2` on invariant failure |
| `parse-ppt-template` | Built-in verification message in stdout | `grep "Verification passed" parse_out.txt` | Message present |
| `generate-ppt` | PPTX file written and valid | `python -c "from pptx import Presentation; Presentation('out.pptx')"` | No exception |
| `generate-ppt` | Slide count roundtrip fidelity | Python assertion (see below) | `len(p.slides) == markdown_section_count` |
| `generate-ppt` | Success marker in stdout | `grep "\[agent4ppt\] PPTX generated" gen_out.txt` | Message present |
| `revise-ppt` | Markdown updated in-place | `grep "Revision complete" rev_out.txt` | Message present |
| `revise-ppt` | Revision state JSON written | `python -c "import json; json.load(open('…_revision_state.json'))"` | No exception |
| `revise-ppt` | `iteration_count` field is integer | Python assertion (see below) | `isinstance(state['iteration_count'], int)` |
| `revise-ppt` | Preview artifacts at deterministic path | `ls <pptx_stem>_previews/slide_1.*` | File exists |

### Per-skill conformance claims

The following table documents what each skill **guarantees** and what it **explicitly does not guarantee**. An absent guarantee is not a soft constraint — it is outside the spec boundary and must not be used as a verification criterion.

#### parse-ppt-template

| Claim | Type | Verified by |
|-------|------|-------------|
| Output markdown exists at declared path | **Guarantee** | File system existence check |
| YAML frontmatter contains `template:` key with non-empty value | **Guarantee** | Built-in invariant 1; exit 2 on failure |
| Separator count equals slide_count − 1 | **Guarantee** | Built-in invariant 2; exit 2 on failure |
| Every substantive section has ≥ 1 `<!-- ph:N type:TYPE -->` annotation | **Guarantee** | Built-in invariant 3; exit 2 on failure |
| `[agent4ppt] Verification passed ✓` in stdout | **Guarantee** | Machine-readable success marker |
| Pixel-perfect preservation of PPTX visual fidelity | **Not guaranteed** | Out of scope for parsing stage |
| Speaker notes, transitions, animations preserved | **Guaranteed (passthrough)** | python-pptx clones the template — not modelled, not destroyed |

#### generate-ppt

| Claim | Type | Verified by |
|-------|------|-------------|
| Output PPTX exists at declared path | **Guarantee** | File system existence check |
| Output is a valid PPTX/ZIP structure openable by python-pptx | **Guarantee** | `Presentation('out.pptx')` no-exception test |
| Slide count matches markdown section count | **Guarantee** | Python assertion (`len(p.slides) == markdown_section_count`) |
| Template path read from `template:` frontmatter field (sole reference) | **Guarantee** | No secondary path; missing field exits 1 |
| `[agent4ppt] PPTX generated → <path>` in stdout | **Guarantee** | Machine-readable success marker |
| Content-placeholder type mismatch aborts generation | **Not guaranteed** | Mismatches emit stderr warnings; generation continues |
| Complete rendering of unsupported placeholder types (`media`) | **Not guaranteed** | Placeholder left empty with warning |

#### revise-ppt

| Claim | Type | Verified by |
|-------|------|-------------|
| Markdown source file updated in-place after Step 3 | **Guarantee** | `Revision complete! Updated file: …` in stdout |
| PPTX regenerated after Step 3 (via generate-ppt subprocess) | **Guarantee** | `[agent4ppt] PPTX generated → …` in stdout |
| Revision state JSON written at `<stem>_revision_state.json` | **Guarantee** | File existence + valid JSON parse |
| `iteration_count` is an integer | **Guarantee** | `isinstance(state['iteration_count'], int)` assertion |
| Preview artifacts written to deterministic `<pptx_stem>_previews/` directory | **Guarantee** | `ls <pptx_stem>_previews/slide_1.*` (PNG or TXT) |
| PNG previews require LibreOffice | **Known dependency** | Falls back to `.txt` if LibreOffice absent — not a failure |
| Applied revisions are semantically equivalent to user instructions | **Not guaranteed** | Free-form instructions appended as `<!-- REVISION: -->` comments for manual/Claude review |

### Full pipeline verification script

```bash
# ── Step 1: parse-ppt-template ─────────────────────────────────────────────
python skills/parse-ppt-template/parse_template.py template.pptx --output out.md \
  > parse_out.txt 2>&1
echo "parse exit=$?"                                               # Must be 0

# Structural conformance checks
grep -q "^template:" out.md             && echo "PASS: template_path in frontmatter"
grep -qP "ph:\d+ type:\w+" out.md       && echo "PASS: placeholder idx annotations found"
grep -q "Verification passed" parse_out.txt \
                                        && echo "PASS: built-in verification passed"

# ── Step 2: generate-ppt ───────────────────────────────────────────────────
python skills/generate-ppt/generate_ppt.py out.md --output out.pptx \
  > gen_out.txt 2>&1
echo "generate exit=$?"                                            # Must be 0
grep -q "\[agent4ppt\] PPTX generated" gen_out.txt \
                                        && echo "PASS: success marker present"

# Roundtrip fidelity: slide count in PPTX == section count in markdown
python -c "
from pptx import Presentation
import re
p = Presentation('out.pptx')
md = open('out.md').read()
body = md.split('---\n', 2)[-1]
slides_md = len(re.split(r'\n---\s*\n', body.strip()))
assert len(p.slides) == slides_md, f'Mismatch: pptx={len(p.slides)} md={slides_md}'
print('PASS: slide count matches')
"

# ── Step 3: revise-ppt (after non-interactive or completed interactive run) ─
grep -q "Revision complete" rev_out.txt  && echo "PASS: revision completed"
python -c "
import json
state = json.load(open('out_revision_state.json'))
assert 'markdown_path' in state,        'missing markdown_path'
assert 'pptx_path' in state,            'missing pptx_path'
assert isinstance(state.get('iteration_count'), int), 'iteration_count must be int'
print('PASS: revision state JSON valid, iteration_count =', state['iteration_count'])
"
```

---

## Environment variables

| Variable | Description |
|----------|-------------|
| `LANG` | POSIX locale string used to auto-detect output language (e.g. `ko_KR.UTF-8` → Korean). Standard system variable; no need to set explicitly on Korean-locale systems. |

---

## License

MIT © Jinwang Mok

---
> Source: [JinwangMok/agent4ppt](https://github.com/JinwangMok/agent4ppt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
