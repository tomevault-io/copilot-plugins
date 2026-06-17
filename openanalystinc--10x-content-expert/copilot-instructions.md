## 10x-content-expert

> This is a comprehensive content creation and file management system powered by Claude. It helps create, edit, and manage content across multiple formats.

# 10X Content Expert - Claude Instructions

## Project Overview
This is a comprehensive content creation and file management system powered by Claude. It helps create, edit, and manage content across multiple formats.

## CRITICAL: Output Path Rules

**ALL generated files MUST go to the `output/` folder structure:**

| File Type | Output Location |
|-----------|-----------------|
| PDF files | `output/pdf/` |
| PPTX files | `output/pptx/` |
| DOCX files | `output/docx/` |
| XLSX files | `output/xlsx/` |
| LinkedIn content | `output/content/social/linkedin/` |
| Twitter content | `output/content/social/twitter/` |
| Email content | `output/content/emails/` |
| Blog content | `output/content/blogs/` |
| Presentations content | `output/content/presentations/` |
| Email sequences | `output/content/sequences/` |
| Hooks/CTAs | `output/content/hooks/` |
| Campaigns | `output/content/campaigns/` |
| Analysis results | `output/analysis/` |
| Content plans | `output/plans/` |
| Working/temp files | `output/working/` |
| MEGA downloads | `output/mega-downloads/` |
| Transcripts | `output/transcripts/` |
| Canvas exports | `output/canvas/` |
| Logs | `output/logs/` |

## Output Path Constants

When generating ANY file output, use these paths:

```
PROJECT_ROOT = C:\Users\Anit\Downloads\10x-Content-Expert

OUTPUT_PATHS = {
    "pdf": "output/pdf/",
    "pptx": "output/pptx/",
    "docx": "output/docx/",
    "xlsx": "output/xlsx/",
    "linkedin": "output/content/social/linkedin/",
    "twitter": "output/content/social/twitter/",
    "instagram": "output/content/social/instagram/",
    "emails": "output/content/emails/",
    "blogs": "output/content/blogs/",
    "presentations": "output/content/presentations/",
    "sequences": "output/content/sequences/",
    "hooks": "output/content/hooks/",
    "campaigns": "output/content/campaigns/",
    "analysis": "output/analysis/",
    "plans": "output/plans/",
    "working": "output/working/",
    "mega": "output/mega-downloads/",
    "transcripts": "output/transcripts/",
    "canvas": "output/canvas/",
    "logs": "output/logs/"
}
```

## File Naming Convention

Use this pattern for generated files:
```
[YYYY-MM-DD]_[topic/name]_[type].[extension]
```

Examples:
- `2026-01-28_arjun_maheshwari_linkedin_profile.pdf`
- `2026-01-28_product_launch_email_sequence.md`
- `2026-01-28_brand_analysis.json`

## Folder Structure

```
10x-Content-Expert/
├── input/                    # User files to edit (READ-ONLY)
├── references/               # Learning materials (READ-ONLY)
│   ├── transcripts/
│   ├── examples/
│   ├── brand-voice/
│   └── templates/
├── samples/                  # Visual references (READ-ONLY)
├── output/                   # ALL OUTPUTS GO HERE
│   ├── pdf/                  # Generated/edited PDFs
│   ├── pptx/                 # Generated/edited PowerPoints
│   ├── docx/                 # Generated/edited Word docs
│   ├── xlsx/                 # Generated/edited Excel files
│   ├── content/              # Written content
│   │   ├── social/
│   │   │   ├── linkedin/
│   │   │   ├── twitter/
│   │   │   └── instagram/
│   │   ├── emails/
│   │   ├── blogs/
│   │   ├── presentations/
│   │   ├── sequences/
│   │   ├── hooks/
│   │   └── campaigns/
│   ├── analysis/             # Analysis results
│   ├── plans/                # Content plans
│   ├── working/              # Temp/working files
│   ├── mega-downloads/       # MEGA cloud downloads
│   ├── transcripts/          # Generated transcripts
│   ├── canvas/               # TLDraw exports
│   └── logs/                 # Operation logs
├── scripts/                  # Python utilities
└── .claude/                  # Skills and agents
```

## Safety Rules

### NEVER Modify
- `input/` - Original files
- `references/` - Learning materials
- `samples/` - Design references

### ALWAYS Output To
- `output/` - All generated content goes here

### The Copy Pattern
When editing user files:
1. Copy original from `input/` to `output/working/`
2. Make ALL edits on the copy
3. Save final result to appropriate `output/[type]/` folder
4. Original in `input/` stays untouched

## Python Script Output

When creating Python scripts that generate files:

```python
import os

# Define project root and output paths
PROJECT_ROOT = r"C:\Users\Anit\Downloads\10x-Content-Expert"

OUTPUT_PATHS = {
    "pdf": os.path.join(PROJECT_ROOT, "output", "pdf"),
    "pptx": os.path.join(PROJECT_ROOT, "output", "pptx"),
    "docx": os.path.join(PROJECT_ROOT, "output", "docx"),
    "xlsx": os.path.join(PROJECT_ROOT, "output", "xlsx"),
    "content": os.path.join(PROJECT_ROOT, "output", "content"),
    "analysis": os.path.join(PROJECT_ROOT, "output", "analysis"),
    "plans": os.path.join(PROJECT_ROOT, "output", "plans"),
    "working": os.path.join(PROJECT_ROOT, "output", "working"),
}

# Always use these paths for output
def get_output_path(file_type, filename):
    """Get the correct output path for a file type."""
    base_path = OUTPUT_PATHS.get(file_type, OUTPUT_PATHS["working"])
    os.makedirs(base_path, exist_ok=True)
    return os.path.join(base_path, filename)
```

## MEGA Integration

- MEGA downloads go to: `output/mega-downloads/`
- Login credentials are stored via `mega-login` command
- Never download files unless explicitly requested
- For browsing/exploring, use `mega-ls` commands only

## Content Creation Workflow

1. **Analyze** - Review references and gather context
2. **Plan** - Create content plan in `output/plans/`
3. **Create** - Generate content to appropriate `output/content/` subfolder
4. **Review** - User reviews and provides feedback
5. **Refine** - Update content based on feedback

## Available Skills

Use `/[skill-name]` to invoke:
- `/content` - Main content creation orchestrator
- `/canva` - Canva design operations
- `/mega` - MEGA cloud file management
- `/setup` - Initial setup and configuration

## Quick Reference

```
PDF output     → output/pdf/
PPTX output    → output/pptx/
DOCX output    → output/docx/
LinkedIn posts → output/content/social/linkedin/
Emails         → output/content/emails/
Analysis       → output/analysis/
Plans          → output/plans/
```

---

## SKILL ROUTING GUIDE (Decision Tree)

Use this table to pick the RIGHT skill for any request:

### "I want to create content..."

| User Says | Use This Skill | Sample Script |
|-----------|---------------|---------------|
| Write an email | `email-copywriter` | `scripts/samples/sample_email_content.py` |
| Write a LinkedIn post | `social-media-writer` | `scripts/samples/sample_social_content.py` |
| Write a Twitter thread | `social-media-writer` | `scripts/samples/sample_social_content.py` |
| Write a blog post | `blog-article-writer` | `scripts/samples/sample_blog_content.py` |
| Create headlines/hooks | `hook-generator` | `scripts/samples/sample_hook_generation.py` |
| Create email sequence | `nurture-sequence` | `scripts/samples/sample_nurture_sequence.py` |
| Create presentation content | `presentation-content` | `scripts/samples/sample_presentation_content.py` |
| Repurpose content | `content-repurposer` | `scripts/samples/sample_repurpose_content.py` |
| Not sure what to create | `content-manager` | - |

### "I want to edit a file..."

| User Says | Use This Skill | Sample Script |
|-----------|---------------|---------------|
| Edit a PDF | `local-pdf-editor` | `scripts/samples/sample_pdf_edit.py` |
| Edit a PowerPoint | `local-pptx-editor` | `scripts/samples/sample_pptx_edit.py` |
| Edit a Word doc | `local-docx-editor` | `scripts/samples/sample_docx_edit.py` |
| Edit a spreadsheet | `local-xlsx-editor` | `scripts/samples/sample_xlsx_edit.py` |
| Edit a file (any type) | `local-file-manager` | Routes to correct editor |

### "I want to use Canva..."

| User Says | Use This Skill |
|-----------|---------------|
| Show my Canva designs | `canva-explorer` |
| Export a design | `canva-export` |
| Edit an image design | `canva-image-editor` |
| Edit a presentation | `canva-presentation` |
| Edit a video | `canva-video` |
| Organize folders | `canva-folder-organizer` |
| Upload assets | `canva-asset-manager` |
| Set up brand kit | `canva-brand-kit` |
| Generate design content | `canva-content-generator` |
| Not sure (Canva) | `canva-manager` |

### "I want to analyze/plan..."

| User Says | Use This Skill | Sample Script |
|-----------|---------------|---------------|
| Analyze my references | `content-analyzer` | `scripts/samples/sample_content_analysis.py` |
| Define brand voice | `brand-voice-manager` | - |
| Which framework to use? | `content-frameworks` | - |

### "I want to use MEGA/other..."

| User Says | Use This Skill |
|-----------|---------------|
| Browse MEGA files | `mega-manager` |
| Transcribe audio/video | `mega-transcriber` |
| Create visual canvas | `tldraw-canvas` |

---

## ACTUAL SCRIPT LOCATIONS (Do NOT hallucinate paths)

**All scripts are relative to project root: `C:\Users\Anit\Downloads\10x-Content-Expert`**

### Canva Scripts (in `scripts/`)
```
scripts/canva_client.py      ← Canva API client (import this)
scripts/auth_check.py        ← Check Canva authentication
scripts/oauth_flow.py        ← Run OAuth flow
scripts/list_designs.py      ← List all designs
scripts/list_folders.py      ← List folder structure
scripts/export_design.py     ← Export a design
```

### Local File Editing Scripts (in `scripts/local/`)
```
scripts/local/safe_copy.py   ← Safe copy before editing (ALWAYS use)
scripts/local/pdf_utils.py   ← PDF analysis and manipulation
scripts/local/pptx_utils.py  ← PPTX analysis and manipulation
scripts/local/docx_utils.py  ← DOCX analysis and manipulation
scripts/local/xlsx_utils.py  ← XLSX analysis and manipulation
```

### Content Analysis Scripts (in `scripts/content/`)
```
scripts/content/analyze_transcript.py  ← Analyze transcripts
scripts/content/analyze_brand_voice.py ← Analyze brand voice
scripts/content/extract_themes.py      ← Extract themes
scripts/content/find_quotes.py         ← Find quotable moments
```

### MEGA Scripts (in `scripts/mega/`)
```
scripts/mega/mega_browse.py  ← Browse MEGA storage
scripts/mega/transcribe.py   ← Transcribe with Whisper
```

### Canvas Scripts (in `scripts/canvas/`)
```
scripts/canvas/generate_canvas.py  ← Generate TLDraw canvas
```

### Sample Scripts (in `scripts/samples/`) — COPY AND MODIFY THESE
```
scripts/samples/sample_email_content.py       ← Email creation template
scripts/samples/sample_social_content.py      ← Social media template
scripts/samples/sample_blog_content.py        ← Blog post template
scripts/samples/sample_pdf_edit.py            ← PDF editing template
scripts/samples/sample_pptx_edit.py           ← PPTX editing template
scripts/samples/sample_docx_edit.py           ← DOCX editing template
scripts/samples/sample_xlsx_edit.py           ← XLSX editing template
scripts/samples/sample_content_analysis.py    ← Content analysis template
scripts/samples/sample_hook_generation.py     ← Hook/headline template
scripts/samples/sample_presentation_content.py ← Presentation template
scripts/samples/sample_nurture_sequence.py    ← Email sequence template
scripts/samples/sample_canva_operations.py    ← Canva API template
scripts/samples/sample_repurpose_content.py   ← Content repurposing template
```

### WRONG PATHS (Do NOT use these — they don't exist)
```
skills/canva-explorer/scripts/     ← WRONG (does not exist)
skills/canva-export/scripts/       ← WRONG (does not exist)
skills/[any-name]/scripts/         ← WRONG (does not exist)
```

---

## RUNNING PYTHON SCRIPTS

**Always use the virtual environment Python:**

```bash
# Windows (ALWAYS use this)
.venv\Scripts\python.exe scripts/[path]/script_name.py

# macOS/Linux
.venv/bin/python scripts/[path]/script_name.py
```

**NEVER use bare `python` or `python3` — always use `.venv\Scripts\python.exe`**

---

## UNIVERSAL DO / DON'T RULES

### DO
- Save ALL outputs to `output/` folder
- Use the naming convention: `YYYY-MM-DD_topic_type.extension`
- Copy files from `input/` to `output/working/` before editing
- Use `.venv\Scripts\python.exe` to run Python scripts
- Check `references/` folder for context before creating content
- Use sample scripts from `scripts/samples/` as starting templates
- Confirm with user before making irreversible changes

### DON'T
- NEVER modify files in `input/`, `references/`, or `samples/`
- NEVER use script paths starting with `skills/[name]/scripts/`
- NEVER use bare `python` — always use `.venv\Scripts\python.exe`
- NEVER skip the safe-copy step when editing files
- NEVER create files outside the `output/` folder
- NEVER guess Canva API endpoints — use existing scripts in `scripts/`

---
> Source: [OpenAnalystInc/10x-Content-Expert](https://github.com/OpenAnalystInc/10x-Content-Expert) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
