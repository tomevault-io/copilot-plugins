## trans-summary

> This document provides context for AI coding assistants to understand the architecture, design patterns, and workflows of the `trans-summary` project.

# AI Context & Architecture Guide for Gemini

This document provides context for AI coding assistants to understand the architecture, design patterns, and workflows of the `trans-summary` project.

## 1. Architecture Overview

The project follows a **Core-Wrapper** pattern to separate business logic from user interfaces.

- **Core Logic (`pipeline.py`)**: Contains the implementation of all major processing steps (formatting, summarizing, generating). It handles API interactions, file I/O, and logic. It is UI-agnostic.
- **Shared Utilities (`transcript_utils.py`)**: Provides robust primitives for logging, API calls (with retry/validation), text normalization, and regex extraction.
- **Configuration (`config.py`)**: Central source of truth for file paths, model selection, and constants.
- **Interfaces**:
  - **GUI (`ts_gui.py`)**: Tkinter-based interface that calls `pipeline.py` functions in threads.
  - **CLI Wrappers (`transcript_*.py`)**: Simple scripts that parse arguments and call `pipeline.py` functions.

## 2. Data Pipeline Flow

Files move through specific states, denoted by suffixes and directories:

1.  **Ingestion**: `source/*.txt` (Raw text)
    - _Action_: `format_transcript`
2.  **Formatting**: `formatted/* - formatted.md` (Clean Markdown)
    - _Action_: `add_yaml`
3.  **Metadata**: `formatted/* - yaml.md` (Formatted + Frontmatter)
    - _Action_: `summarize_transcript`
4.  **Extraction**: `projects/* - structural-themes.md`, `interpretive-themes.md`, `topics.md`, `key-terms.md`, `lenses-ranked.md`, `bowen-references.md`, `emphasis-items.md`
5.  **Synthesis**: `projects/* - summary-generated.md`, `abstract-generated.md`
6.  **Publication**: `projects/*.html`, `projects/*.pdf`

## 3. Key Design Patterns

### API Interaction

- **Centralized Calls**: All Claude API calls go through `transcript_utils.call_claude_with_retry`.
- **7-Level Validation**: Responses are rigorously validated (message type, role, stop reason, content existence, text validity, token usage) to prevent silent failures.
- **Truncation Safety**: The system automatically detects and rejects responses truncated by token limits.
- **Streaming**: Long generations use streaming to prevent timeouts.

### Text Processing

- **Regex over JSON**: The pipeline prefers robust Regex extraction (see `transcript_utils.extract_section`) over strict JSON parsing for large text blocks, as it is more forgiving of LLM formatting minor errors.
- **Fuzzy Matching**: Validation scripts (`transcript_validate_*.py`) use `difflib.SequenceMatcher` to verify that generated quotes exist in the source text, handling minor punctuation/whitespace differences.

### Prompt Management

- **External Files**: Prompts are stored as Markdown files in `prompts/`.
- **Template Injection**: `pipeline._fill_prompt_template` handles Jinja-like variable replacement (e.g., `{{transcript}}`).

## 4. Critical Files Map

| File                          | Responsibility                                                          |
| ----------------------------- | ----------------------------------------------------------------------- |
| `pipeline.py`                 | **The Brain.** Orchestrates all logic. If adding a feature, start here. |
| `config.py`                   | **The Settings.** Paths, models, constants.                             |
| `transcript_utils.py`         | **The Toolbox.** Logging, API wrappers, text helpers.                   |
| `html_generator.py`           | **The Publisher.** Logic for HTML/PDF creation.                         |
| `summary_validation.py`       | **The Critic.** Deep semantic validation for summaries.                 |
| `abstract_validation.py`      | **The Critic.** Deep semantic validation for abstracts.                 |
| `ts_gui.py` | **The Face.** Tkinter GUI implementation.                               |

## 5. Validation Strategy

The project emphasizes correctness. Every generation step has a corresponding validation step:

- **Format**: `validate_format` checks word preservation.
- **Headers**: `validate_headers` checks header relevance via LLM.
- **Abstract/Summary**: `validate_coverage` checks if required topics are present.
- **Quotes**: `validate_emphasis_items` ensures quotes are not hallucinations.

## 6. Common Tasks for AI Agents

- **Adding a new pipeline step**:

  1. Define prompt in `prompts/`.
  2. Add logic to `pipeline.py`.
  3. Add CLI wrapper `transcript_new_step.py`.
  4. Update `ts_gui.py` to include a button.

- **Updating Prompts**:

  - Modify files in `prompts/`.
  - Ensure placeholders (e.g., `{{transcript}}`) match what `pipeline.py` provides.

- **Changing Models**:
  - Update `DEFAULT_MODEL` or `AUX_MODEL` in `config.py`.

- **Testing & QA**:
  - Always run `pytest tests` after making logic changes.
  - Run `ruff check .` to ensure code style compliance.
  - Add new tests in `tests/` if introducing new logic.

## 7. Workflows

- **Request Logging**:
  - Automatically track user requests and task completion status in `REQUEST_LOG.md`.
  - Update this log at the start and end of significant tasks.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dbgnvan2)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dbgnvan2)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
