## agent-jude

> Agent Jude is an AI-powered document processing tool that generates product requirements and user stories from DOCX templates. It uses OpenAI's GPT-4 Turbo to populate placeholder-based DOCX templates with structured content, presented through a Streamlit web interface.

# CLAUDE.md - Agent Jude

## Project Overview

Agent Jude is an AI-powered document processing tool that generates product requirements and user stories from DOCX templates. It uses OpenAI's GPT-4 Turbo to populate placeholder-based DOCX templates with structured content, presented through a Streamlit web interface.

**Author:** Thien Nghiem
**License:** MIT (2024)

## Tech Stack

- **Language:** Python 3
- **UI Framework:** Streamlit
- **AI Integration:** OpenAI API (GPT-4 Turbo - `gpt-4-turbo-preview`)
- **Document Processing:** python-docx (DOCX manipulation), mammoth (DOCX-to-HTML conversion)

## Project Structure

```
agent-jude/
├── main.py                    # Main Streamlit application (entry point)
├── requirements.txt           # Python dependencies
├── demo.docx                  # Demo DOCX template with placeholders
├── FPT_Software_Logo.png      # Logo asset used in demo template
├── example_image.png          # Sample image
├── test_docx_generation.py    # Script to generate demo.docx template
├── test_docx_to_html.py       # Script to test DOCX-to-HTML conversion
├── test_pdf_viewer.py         # Script to test PDF viewer widget
├── archived/                  # Previous application versions (v0.0, v1.0, v2.0)
│   ├── main_v0.0.py           # v0: Basic DOCX editor
│   ├── main_v1.0.py           # v1: Added OpenAI integration
│   ├── main_v2.0.py           # v2: Added table generation
│   └── generate_resume.py     # Experimental LaTeX/PDF resume generator
└── temp/                      # Runtime temp directory (created automatically)
```

## Architecture

### Core Data Flow

1. User uploads a DOCX template containing `[targeted_text]` and `[targeted_table]` placeholders
2. User describes product requirements in natural language
3. OpenAI generates content (plain text for docs, JSON for tables)
4. Placeholders in the DOCX are replaced with generated content
5. Modified DOCX is converted to HTML for in-browser preview (via mammoth)
6. User downloads the final modified DOCX

### Key Functions in `main.py`

| Function | Purpose |
|----------|---------|
| `load_docx(document)` | Loads a DOCX file into a python-docx Document object |
| `copy_document(doc)` | Deep-copies a Document's XML body into a new Document |
| `generate_content(doc, nl_instruction)` | Calls OpenAI to generate text, replaces `[targeted_text]` placeholders |
| `generate_table(doc, nl_instruction)` | Calls OpenAI for JSON-structured table data, replaces `[targeted_table]` with a formatted table |
| `save_docx(doc)` | Serializes a Document to a BytesIO buffer |
| `docx_to_html(docx_path, html_path)` | Converts DOCX to HTML using mammoth for preview |
| `main()` | Streamlit UI entry point and workflow orchestration |

### Template Placeholder Convention

- `[targeted_text]` - Replaced with AI-generated product documentation (plain text)
- `[targeted_table]` - Replaced with a structured user story table (Section Name + Description columns)

### Session State

Streamlit session state keys:
- `st.session_state.product_doc` - Stores the Document after text generation
- `st.session_state.modified_doc` - Stores the Document after table generation

## Development Setup

### Prerequisites

- Python 3.x
- An OpenAI API key

### Installation

```bash
pip install -r requirements.txt
```

### Secrets Configuration

Create `.streamlit/secrets.toml` (this file is gitignored):

```toml
OPENAI_API_KEY = "sk-..."
```

### Running the Application

```bash
streamlit run main.py
```

### Generating the Demo Template

```bash
python test_docx_generation.py
```

This produces `demo.docx` with both `[targeted_text]` and `[targeted_table]` placeholders.

## Dependencies

From `requirements.txt`:
- `openai` - OpenAI API client
- `python-docx` - DOCX file creation and manipulation
- `streamlit` - Web application framework
- `mammoth` - DOCX to HTML conversion

## Testing

Test files are standalone scripts (no test framework configured):
- `test_docx_generation.py` - Generates the demo DOCX template
- `test_docx_to_html.py` - Tests mammoth DOCX-to-HTML conversion
- `test_pdf_viewer.py` - Tests the Streamlit PDF viewer widget

Run individually with `python <test_file>.py`.

## Conventions and Guidelines

- **No formal linter or formatter** is configured. Follow existing code style (PEP 8 conventions).
- **No CI/CD pipeline** exists. Changes are tested manually.
- **Secrets must never be committed.** The `.gitignore` excludes `.streamlit/secrets.toml`.
- **Temp files** go in the `./temp/` directory, which is created at runtime.
- **Archived versions** in `archived/` are kept for reference and should not be modified.
- The OpenAI model is hardcoded as `gpt-4-turbo-preview` in `main.py`.
- All AI responses use `temperature=0.5` and `stream=False`.
- Table generation uses OpenAI's JSON response format (`response_format={"type": "json_object"}`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GrandPurpleOcelot) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
