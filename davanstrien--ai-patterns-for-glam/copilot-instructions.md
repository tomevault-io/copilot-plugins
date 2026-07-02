## ai-patterns-for-glam

> This is a Quarto book documenting AI/ML patterns and methodologies emerging from National Library of Scotland consulting work. It serves both NLS and the broader GLAM community by focusing on pattern-based documentation that survives technology changes.

# AI Patterns for GLAM - Quarto Book Project

## Overview
This is a Quarto book documenting AI/ML patterns and methodologies emerging from National Library of Scotland consulting work. It serves both NLS and the broader GLAM community by focusing on pattern-based documentation that survives technology changes.

**Target Audiences**: Library leadership, project coordinators, technical staff, general interest
**Approach**: Progressive disclosure using Quarto features (panel-tabset for multi-audience content)

## Repository Information
- **Local Path**: `/Users/davanstrien/Documents/nls-work/ai-patterns-for-glam`
- **GitHub**: `git@github.com:davanstrien/ai-patterns-for-glam.git` (private initially)
- **Publishing**: Hugging Face Space (controlled deployment via GitHub Actions)
  - Source code: GitHub (private)
  - Published book: HF Space only (public)
  - Deployment: Manual trigger OR automatic on `publish` branch
  - See `PUBLISHING.md` for setup instructions

## Book Structure (_quarto.yml)

1. **Beyond Chatbots: Finding AI Opportunities** - Discovery phase/framework content
2. **Design Patterns** - High-level patterns with multi-audience tabs
3. **Technology Evaluations** - Technical demonstration chapters:
   - `ocr-evaluation.qmd` - OCR technology comparison
   - `vlm-structured-generation.qmd` - VLM tutorial with Sloane index cards (verbose, includes LM Studio setup)
   - `advisor-index-cards.ipynb` - Practical extraction demo with **emphasis on evaluation strategies** (less verbose)
4. **Infrastructure Considerations** - Hardware, deployment
5. **Appendices** - Assessment forms, production pipeline details

## Current Work: Advisor Index Cards Chapter

### Background
Working with NLS curators on extracting structured metadata from historical index cards (Advocate's Library collection). Real consulting engagement with specific catalog requirements.

### Schema Design (Final)
After iteration, settled on simple 6-field structure + 2 metadata fields:

```python
class IndexCardEntry(BaseModel):
    surname: str  # Family name as written
    forenames: Optional[str]  # Given names
    epithet: Optional[str]  # Title, occupation, or role
    ms_no: str  # Manuscript number (kept as string for complex formats like "33.1.7")
    description: str  # Document description with date
    folios: str  # Folio reference

    # Quality tracking
    failed_to_parse: bool = False  # Flag for illegible/damaged cards
    notes: Optional[str] = None  # Handwritten corrections, ambiguities, issues
```

**Key Design Decisions**:
- Simple over complex - curator specified 6 fields, we added 2 for quality
- All strings except boolean flag (preserves ambiguity like "ca. 1783")
- Optional fields where appropriate (forenames, epithet, notes)
- Separate `failed_to_parse` bool for easy filtering
- `notes` field for human-in-the-loop workflow

**What We Tried That Didn't Work**:
- ❌ `raw_text` field for full OCR transcription - resulted in worse structured extraction (model got lazy)
- ✅ Focused structured extraction only

### Extraction Prompt Strategy
- **Field-focused** (not JSON-focused) - relies on `response_format` schema for structure
- Two-part: field definitions with examples + extraction guidelines
- Emphasis on "extract exactly as written" to reduce hallucination
- Low temperature (0.1) for consistency

### Sample Data
- Sample images: `/Users/davanstrien/Downloads/indexes 2/DSC00168-173.JPG`
- Reference Excel: `/Users/davanstrien/Downloads/advindexes.xlsx` (159 manually cataloged entries)

### Technical Setup
- **Model**: qwen/qwen2.5-vl-7b via LM Studio
- **API**: OpenAI client → localhost:1234
- **Method**: `client.beta.chat.completions.parse()` with Pydantic response_format
- **Temperature**: 0.1 for extraction

## Evaluation Framework (advisor-index-cards.ipynb)

The chapter emphasizes **4 evaluation strategies**:

1. **Manual Ground Truth** - Gold standard, establish baselines (~50-100 cards)
2. **Model-as-Judge** - Stronger model evaluates outputs, scales to full dataset
3. **Consistency Checks** - Automated validation rules (MS number format, date ranges, folio patterns)
4. **Confidence Scoring** - Prioritize review using model uncertainty (if available)

**Recommendation**: Combine approaches - manual baseline, automated checks, model-as-judge sampling, confidence-based review prioritization.

## Chapter Philosophy

### vlm-structured-generation.qmd (Tutorial)
- Verbose, educational
- Explains VLM concepts
- Shows LM Studio setup
- Walks through basics

### advisor-index-cards.ipynb (Practical Demo)
- Less verbose
- Assumes reader knows VLM basics
- References previous chapter for setup
- Focus: real extraction + comprehensive evaluation
- Heavy on practical considerations (edge cases, failure modes, catalog export)

## TODO Placeholders in advisor-index-cards.ipynb
- [ ] Load and display sample images
- [ ] Implement extraction function
- [ ] Run extraction on 5-10 samples
- [ ] Manual ground truth comparison
- [ ] Model-as-judge evaluation implementation
- [ ] Consistency validation rules
- [ ] Confidence score analysis (if available)
- [ ] Batch processing results
- [ ] Edge case analysis
- [ ] Export to catalog formats

## Development Notes

### Working with .ipynb in Quarto
- Switched from .qmd to .ipynb for better VSCode experience
- Quarto renders Jupyter notebooks natively
- YAML frontmatter in first markdown cell

### Virtual Environment
- Python environment: `.venv/` (created with `uv venv`)
- Key packages: datasets, pillow, tqdm, huggingface-hub, pandas, openpyxl, pydantic, openai

### Common Commands
```bash
# Preview book
quarto preview

# Render book
quarto render

# Install dependencies
uv pip install <package>
```

## Related Projects

This book is part of the broader NLS consulting work. See parent directory CLAUDE.md (`/Users/davanstrien/Documents/nls-work/CLAUDE.md`) for:
- India Papers dataset conversion
- Handbooks dataset
- Overall NLS engagement context

## Pattern Documentation Strategy

**Goal**: Document patterns that survive technology changes

Examples:
- **Structured Document Processing** (index card digitization) ← current work
- **Visual Assessment at Scale** (conservation condition assessment)
- **Legacy Format Liberation** (dataset conversion)
- **Workflow Augmentation** (cataloging support)

Each pattern includes multi-audience content:
- Business case (leadership)
- Workflow integration (coordinators)
- Technical implementation (staff)
- Conceptual explanation (general)

---
> Source: [davanstrien/ai-patterns-for-glam](https://github.com/davanstrien/ai-patterns-for-glam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
