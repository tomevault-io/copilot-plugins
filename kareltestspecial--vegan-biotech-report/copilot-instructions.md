## vegan-biotech-report

> This document provides an overview of the "Vegan BioTech Report" project, its architecture, key technologies, and development conventions to serve as instructional context for future interactions.

# GEMINI.md: Vegan BioTech Report Project Context

This document provides an overview of the "Vegan BioTech Report" project, its architecture, key technologies, and development conventions to serve as instructional context for future interactions.

## Project Overview

The "Vegan BioTech Report" is an automated platform designed to generate news and biotechnology reports. It leverages AI to create textual content (newsletters and longreads) and associated cover images. The generated content is then used to build a multi-language static website using Hugo, which is subsequently deployed. The project includes a robust Python-based pipeline for content generation, curation, and management, orchestrated via GitHub Actions for automation and deployment.

### Key Technologies

*   **Hugo**: Static site generator used to build the final website.
*   **Python**: Powers the entire content generation, AI integration, and content management pipeline.
*   **Gemini AI**: Utilized for drafting content and generating images using the modern `google-genai` SDK (Gemini 3.0 compliant).
*   **GitHub Actions**: Central to the project's automation, handling content generation, site building, and deployment workflows.
*   **`python-dotenv`**: For managing environment variables (e.g., API keys) from `.env` files.

## Project Structure

The project is organized into several key directories:

*   **`.github/workflows/`**: Contains YAML files defining the GitHub Actions workflows for various automated tasks.
*   **`content/`**: Stores all Markdown content generated for the Hugo website, categorized into `newsletters` and `longreads`.
*   **`pipeline/`**: Houses Python scripts for the content generation pipeline.
*   **`prompts/`**: Contains text files used as prompts for the AI models.
*   **`reports/`**: Documentation and internal reports.
*   **`static/`**: Static assets, including `images/` and `css/custom.css`.
*   **`themes/ananke/`**: The Hugo theme, included as a Git submodule.

## Building and Running

### Python Content Generation Pipeline

1.  **Install Python Dependencies**:
    ```bash
    pip install -r requirements.txt
    ```
2.  **Run the Full Pipeline**:
    ```bash
    python3 -m pipeline.run_pipeline
    ```
    *Note: The pipeline uses `google-genai` and includes a dynamic topic selection mechanism that scans `content/longreads/` to avoid repeating recent topics.*

### Hugo Local Development Server

1.  **Serve the Website Locally**:
    ```bash
    hugo server
    ```

### Deployment

Deployment is managed through **GitHub Actions**. The primary workflows are:
*   `1-auto-generate-and-deploy.yml`: End-to-end generation, building, and deployment.
*   `4-build-and-deploy-no-content-generation.yml`: Deploys the current state of the repository without creating new content.
*   `5-manage-content-status.yml`: Interface for archiving or making content live.

## Development Conventions

*   **Python Scripts**: Run as modules (e.g., `python3 -m pipeline.script_name`).
*   **Gemini 3.0 Compliance**: AI interactions must use robust parsing (e.g., checking `response.candidates[0].content.parts`) to handle "thought signatures" and avoid crashes.
*   **Topic Avoidance**: The `run_pipeline.py` script automatically synchronizes `last_topics.json` with the existing content in `content/longreads/` before selecting a new topic.
*   **Multi-language Support**: Configured via `languages.json` (currently English and Dutch).
*   **Automation**: GitHub Actions ensure consistent updates and deployment.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KarelTestSpecial)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/KarelTestSpecial)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
