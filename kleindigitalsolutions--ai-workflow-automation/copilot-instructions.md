## ai-workflow-automation

> This file provides context and instructions for interacting with the `ai-workflow-automation` project.

# Gemini Context: AI Workflow Automation

This file provides context and instructions for interacting with the `ai-workflow-automation` project.

## Project Overview

This is a Python-based automation and analysis tool that combines **Playwright** for web scraping/browser automation with **Anthropic's Claude API** for intelligent data processing.

**Key Capabilities:**
*   **Browser Automation:** Scrapes dynamic websites using Playwright.
*   **Data Processing:** Cleans and normalizes extracted data.
*   **AI Analysis:** Uses Claude to summarize content, analyze products, and generate insights.
*   **Robustness:** Features error handling, retries, and automatic screenshotting on failure.

## Architecture & Key Files

*   **`automation.py`**: The main entry point and workflow orchestrator. Contains `WorkflowAutomation` class.
*   **`utils/`**:
    *   `playwright_helper.py`: Wrapper for Playwright operations (navigating, clicking, extracting).
    *   `ai_analyzer.py`: Interface for the Anthropic API.
    *   `data_cleaner.py`: Utilities for text sanitization and normalization.
*   **`prompts/`**: Text files containing prompt templates (`product_analysis_prompt.txt`, etc.).
*   **`results/`**: Output directory for JSON results.
*   **`logs/`**: Execution logs and error screenshots.

## Setup & Configuration

### Prerequisites
*   Python 3.8+
*   Anthropic API Key

### Installation
1.  **Install Dependencies:**
    ```bash
    pip install -r requirements.txt
    playwright install chromium
    ```
2.  **Environment Setup:**
    Copy `.env.example` to `.env` and configure:
    ```bash
    cp .env.example .env
    ```
    **Required Variables:**
    *   `ANTHROPIC_API_KEY`: Your Claude API key.
    **Optional Variables:**
    *   `CLAUDE_MODEL`: AI model version (default: `claude-3-5-sonnet-20241022`).
    *   `TARGET_URL`: Default URL for the demo.
    *   `HEADLESS`: `true`/`false` for browser visibility.

## Common Tasks & Commands

### Run Default Workflow
Executes the product analysis demo on the configured `TARGET_URL` (default: `books.toscrape.com`).
```bash
python automation.py
```

### Run Custom Workflow
See `example_custom_workflow.py` (if available) or create a script to import `WorkflowAutomation`.
```python
from automation import WorkflowAutomation
import asyncio

async def main():
    app = WorkflowAutomation()
    await app.run_content_scraping_workflow("https://example.com")

asyncio.run(main())
```

### Testing
(If tests are added in the future, they would likely be run via `pytest` as it is listed in `requirements.txt`.)

## Development Conventions

*   **Async/Await:** The project heavily relies on Python's `asyncio` and `aiohttp` patterns. Ensure new I/O bound code is asynchronous.
*   **Type Hinting:** Use Python type hints for all function arguments and return values.
*   **Modular Design:** Keep scraping logic in `playwright_helper`, AI logic in `ai_analyzer`, and orchestration in `automation.py`.
*   **Logging:** Use the configured logger. Do not use bare `print()` statements for production logs.
*   **Error Handling:** Wrap Playwright actions in try/except blocks and utilize the screenshot capability for debugging.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KleinDigitalSolutions)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/KleinDigitalSolutions)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
