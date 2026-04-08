## bank-agent

> This project is an intelligent financial assistant that automates the tracking of personal finances by processing bank notification emails from a user's Gmail inbox. It leverages the Google Gmail API to fetch transaction emails and extracts structured transaction data (expenses, payroll, withdrawals) using **Gemini 2.0 Flash Lite** with structured output.

# Gemini Project: Bank Notification Agent

## Project Overview

This project is an intelligent financial assistant that automates the tracking of personal finances by processing bank notification emails from a user's Gmail inbox. It leverages the Google Gmail API to fetch transaction emails and extracts structured transaction data (expenses, payroll, withdrawals) using **Gemini 2.0 Flash Lite** with structured output.

Key technologies and components:

* **Python:** The primary programming language.
* **Google Gmail API:** Used to access and fetch bank notification emails.
* **Google OAuth2:** For secure authentication with Gmail.
* **Gemini 2.0 Flash Lite:** For processing email content and extracting structured transaction data using structured output with return schema.
* **Pydantic:** For defining structured output models and application settings.
* **`uv`:** A fast dependency manager for Python.
* **`pyproject.toml`:** Project configuration and dependency declaration.

## Building and Running

### Prerequisites: Google Cloud Setup

Before running the project, you must configure a Google Cloud Project with OAuth access to your Gmail. Refer to the original `README.md` for detailed steps on:

1. Creating a Google Cloud Project and enabling the Gmail API.
2. Configuring the OAuth Consent Screen (User Type: External, add your Gmail as a test user).
3. Creating OAuth client ID credentials (Desktop app type) and downloading `credentials.json` to the project root.
4. Obtaining a Gemini API key from Google AI Studio and configuring it in `.env`.

### Installation & Execution

The project uses `uv` for dependency management.

#### Option 1: Using `uv` (Recommended)

```bash
# Initialize the environment and install dependencies
uv sync

# Run the project
uv run main.py
```

#### Option 2: Using Plain Python & Pip

```bash
# Create a virtual environment
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# Install dependencies
pip install google-api-python-client google-auth-httplib2 google-auth-oauthlib google-genai pydantic pydantic-settings sqlalchemy

# Run the project
python main.py
```

### Authentication

The first time you run the script, a browser window will open for Google account sign-in and OAuth authorization. A `token.json` file will be created to store your session for subsequent runs.

## Development Conventions

* **Dependency Management:** The project utilizes `uv` for efficient dependency management, with `pyproject.toml` defining the project's metadata and dependencies.
* **Google API Integration:** Follows standard practices for integrating with Google APIs, including OAuth2 for authentication and `google-api-python-client` for interacting with the Gmail API.
* **GenAI Integration:** Uses the `google-genai` library with structured output (`response_mime_type="application/json"` and `response_schema`) to enforce consistent JSON responses from Gemini 2.0 Flash Lite.
* **Project Structure:**
  * `main.py`: Entry point and pipeline orchestrator.
  * `models.py`: Pydantic models for GenAI structured output.
  * `services/gmail.py`: Gmail API interactions, including authentication and email fetching.
  * `services/genai.py`: Gemini 2.0 Flash Lite integration for transaction data extraction.
  * `config.py`: Application settings via Pydantic BaseSettings.
  * `constants.py`: Scopes and bank email address configuration.
  * `credentials-desktop.json` / `credentials-web.json`: OAuth App ID files from Google Cloud.
  * `token.json`: User session token (generated after first login).
  * `pyproject.toml`: Project configuration and dependencies.
  * `README.md`: Project documentation.

## Markdown Standards

When adjusting or creating `.md` files, follow these rules:

* **Rule:** `MD030` (list-marker-space)
* **Description:** Enforce a single space after list markers (e.g., `*`, `-`, `1.`).
* **Expected:** 1 space.
* **Actual Violation Example:** `1.  Item` (2 spaces).
* **Action:** Always use exactly one space after list markers.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ManasesLovera)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ManasesLovera)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
