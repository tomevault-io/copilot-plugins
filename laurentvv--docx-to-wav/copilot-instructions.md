## docx-to-wav

> This file provides foundational context and technical mandates for the `docx-to-wav` project.

# GEMINI.md - Project Context (v3.1)

This file provides foundational context and technical mandates for the `docx-to-wav` project.

## Project Mandates

1.  **Direct Audio Output**: Prefer direct MP3 generation for Google Cloud TTS to avoid `ffmpeg` dependencies in the cloud workflow.
2.  **SSML-First Architecture**: Always use SSML for text synthesis to ensure precise control over prosody, pauses, and pronunciation.
3.  **Content Separation**: Distinguish between narrative text and structural elements (Titles). Titles must have distinct audio characteristics.

## Architecture & Conventions

### Text Processing Flow
1.  **Extraction** (`utils.py`): 
    *   Extract paragraphs from `.docx`.
    *   Detect titles based on: Word styles (`Heading`, `Titre`), explicit keywords (`Chapitre`, `Acte`), Bold runs, or short uppercase content.
2.  **Normalization** (`utils.py`): Convert Microsoft Word typography (ellipsis, guillemets, etc.) to standardized ASCII. *Note: Curly apostrophes are intentionally preserved.*
3.  **Splitting** (`utils.py`): Break long segments into sentences (max 1500 chars) to prevent prosody degradation.
4.  **SSML Enrichment** (`convert_docx_google_tts.py`): 
    *   Inject `<break time="500ms"/>` between standard segments.
    *   Apply `<prosody rate="0.9">` and `<emphasis level="strong">` to Titles, followed by a `<break time="1s"/>`.

### Key Files
- `utils.py`: The single source of truth for text manipulation and extraction logic.
- `convert_docx_google_tts.py`: Orchestrates direct MP3 synthesis via Google Cloud.
- `config_google_tts.yaml`: Primary configuration for the Google engine (Voice selection, global speed).

## Development Conventions
- **Language**: Optimization is focused on **French (fr-FR)**.
- **Dependencies**: Use `uv` for all environment and script executions.

## Known Model Behaviors
- **Polyglot-1**: Default voice. Excellent stability and flow for long narrations.
- **Chirp3-HD-Algenib**: High-quality "human" grain.

## TODOs
- **Batching**: Monitor the 5000-character SSML limit during heavy batching in `convert_docx_google_tts.py`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/laurentvv) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
