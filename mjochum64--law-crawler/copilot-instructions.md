## law-crawler

> This document outlines the best practices for interacting with a Gemini-based AI assistant within this project.

# Gemini AI Interaction Guidelines

This document outlines the best practices for interacting with a Gemini-based AI assistant within this project.

## Core Mandates

- **Adhere to Conventions**: Rigorously adhere to existing project conventions when reading or modifying code. Analyze surrounding code, tests, and configuration first.
- **Libraries/Frameworks**: NEVER assume a library/framework is available or appropriate. Verify its established usage within the project before employing it.
- **Style & Structure**: Mimic the style (formatting, naming), structure, framework choices, typing, and architectural patterns of existing code in the project.
- **Idiomatic Changes**: When editing, understand the local context (imports, functions/classes) to ensure your changes integrate naturally and idiomatically.
- **Comments**: Add code comments sparingly. Focus on *why* something is done, especially for complex logic, rather than *what* is done.
- **Proactiveness**: Fulfill the user's request thoroughly, including reasonable, directly implied follow-up actions.
- **Confirm Ambiguity/Expansion**: Do not take significant actions beyond the clear scope of the request without confirming with the user.

## Project-Specific Context

- **Language**: Python
- **Dependencies**: Managed in `pyproject.toml`. Use `pip install .` to install.
- **Main Logic**: The core crawling logic is in `src/crawler.py`.
- **Data**: Downloaded XML files are stored in the `data/` directory.
- **Goal**: The primary goal is to download XML court decisions from "rechtsprechung-im-internet.de".

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mjochum64)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mjochum64)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
