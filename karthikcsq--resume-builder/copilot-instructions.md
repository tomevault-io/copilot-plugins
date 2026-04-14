## resume-builder

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

### Local Development
```bash
# Install dependencies
pip install -r requirements.txt

# Render resume and CV from YAML (outputs to current directory)
python render_tex.py

# Publish to all destinations (website, RAG, OneDrive, git push)
python publish.py

# Convert YAML to JSON for API testing
python get_yaml_json.py > resume.json
```

### Docker
```bash
# Build container with TeX Live
docker build -t resume-builder .

# Render using Docker (no local LaTeX needed)
docker run --rm -v ${PWD}:/app -w /app resume-builder
```

### FastAPI Web Service
```bash
# Run API server on port 10000
python app.py
# or
uvicorn app:app --host 0.0.0.0 --port 10000

# Health check
curl http://localhost:10000/health
```

## Architecture Overview

### Core Data Flow
The system converts structured YAML resume data into LaTeX documents and PDFs through a multi-stage pipeline:

```
YAML → Filter by target → Escape LaTeX chars → Jinja2 render → .tex → pdflatex → .pdf
```

### Key Components

**render_tex.py** - Core rendering engine
- `create_tex()`: Pure rendering function (YAML string → LaTeX string)
- `render_target()`: Full pipeline (render → write → compile PDF → cleanup)
- `filter_for_target()`: Recursively filters items by `show_on` field
- `escape_all()`: Escapes LaTeX special characters throughout data structure

**app.py** - FastAPI web service with 6 endpoints
- `/render` - YAML input → both resume and CV PDFs
- `/render_json` - JSON input → both PDFs
- `/render_json_pdf` - JSON → single PDF (most efficient for single doc)
- `/get_tex` - JSON → LaTeX source only (no PDF)
- `/download/{request_id}/{doc_type}` - Retrieve generated PDFs
- `/health` - Service health check

**publish.py** - Publication workflow
1. Renders CV and resume from YAML
2. Copies to 4 destinations (website public, RAG docs, utils, OneDrive)
3. Runs `create-pinecone.py` in python-rag folder for RAG indexing
4. Commits and pushes to personalsite git repo

**copy_utils.py** - File distribution helpers
- `copy_to_public()`, `copy_to_rag()`, `copy_to_utils()` - Website destinations
- `copy_to_documents()` - OneDrive with timestamped filenames
- `run_git_commands()` - Automated git add/commit/push

### Template System

**Jinja2 with Custom Delimiters** (to avoid LaTeX syntax conflicts):
- Variables: `<<<variable>>>` instead of `{{variable}}`
- Blocks: `<% for/if %>` instead of `{% for/if %}`
- Comments: `<# comment #>` instead of `{# comment #}`

**Two Templates**:
- `templates/resume.tex.j2` - Tight spacing for one-page format
- `templates/cv.tex.j2` - More generous spacing

Both share identical structure (header, education, skills, experience, projects) but differ in spacing parameters.

### Target Filtering

Items can specify `show_on: ["resume", "cv"]` to control visibility:
- Missing `show_on` → appears in all targets
- `["resume"]` → resume only
- `["cv"]` → CV only
- `["resume", "cv"]` → both (explicit)

Filtering happens recursively before template rendering via `filter_for_target()`.

## YAML Data Structure

The single source of truth (`resume_truth.yaml` or `karthik_thyagarajan_truth.yaml`) contains:
- `name` - Full name
- `contact` - List of contact methods (email, phone)
- `links` - List of `{label, url}` objects (GitHub, LinkedIn, etc.)
- `education` - List with `institution`, `degree`, `gpa`, `dates`, `coursework`, `show_on`
- `skills` - List with `category`, `bullets`, `show_on`
- `experience` - List with `role`, `company`, `location`, `work_type`, `start_date`, `end_date`, `bullets`, `show_on`
- `projects` - List with `title`, `tools`, `date`, `link`, `bullets`, `show_on`

## External Integrations (publish.py)

The publish script integrates with:
- **Personal Website**: Copies PDFs to `~/CodingFiles/PersonalWebsite/personalsite/public/`
- **RAG System**: Copies YAML to `~/CodingFiles/PersonalWebsite/personalsite/python-rag/rag-docs/`
- **Pinecone**: Runs `create-pinecone.py` in python-rag folder for vector indexing
- **OneDrive**: Copies to `~/OneDrive/Documents/` with timestamps
- **Git**: Automated commit/push to personalsite repository

Hard-coded paths assume this personal file structure.

## Important Implementation Details

1. **Stateless API Design**: Each API request gets UUID-based temp directory under `/app/tmp/`
2. **Immediate Cleanup**: Files deleted after streaming PDF response
3. **Graceful Degradation**: If pdflatex not found, only .tex files generated
4. **LaTeX Character Escaping**: All special chars (`\`, `&`, `%`, `$`, `#`, `_`, `{`, `}`, `~`, `^`) escaped before rendering
5. **Separation of Concerns**: `create_tex()` is pure (rendering only), `render_target()` handles I/O and compilation
6. **Reusable Functions**: Both render functions can be imported for custom workflows

## File Paths

When modifying file operations:
- Templates: `templates/*.tex.j2`
- Output files: `resume_output.tex/pdf`, `cv_output.tex/pdf` (in working directory)
- Temp directory (API): `/app/tmp/{uuid}/`
- Personal paths (publish.py): Under `Path.home() / "CodingFiles" / ...`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/karthikcsq)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/karthikcsq)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
