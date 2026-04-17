## nlba

> This document outlines specific guidelines for the Gemini CLI agent when working on the Natural Language Bash Assistant (NLBA) project.

# Gemini CLI Agent Guidelines for NLBA Project

This document outlines specific guidelines for the Gemini CLI agent when working on the Natural Language Bash Assistant (NLBA) project.

## General Rules

*   **Confirmation for New Tasks:** Always ask for explicit user confirmation before starting any new, significant task or feature implementation. This includes, but is not limited to, adding new features, refactoring large sections of code, or making architectural changes.

## Project-Specific Information

*   **Testing:** To run tests, use the command `uv run pytest` from the project root directory.
*   **Executable:** The `nlba` command can be run directly from the terminal after installing the package in editable mode (`uv pip install -e .`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ArtyomKa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
