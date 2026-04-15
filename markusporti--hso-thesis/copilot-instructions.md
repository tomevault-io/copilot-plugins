## hso-thesis

> This project is a **Typst** package and template designed for Bachelor's, Master's, and seminar papers at **Hochschule Offenburg (HSO)**. It provides a standardized layout following official guidelines and supports multiple faculties.

# GEMINI Context: hso-thesis

This project is a **Typst** package and template designed for Bachelor's, Master's, and seminar papers at **Hochschule Offenburg (HSO)**. It provides a standardized layout following official guidelines and supports multiple faculties.

## Project Overview

- **Purpose:** Provide a modern, programmable alternative to LaTeX templates for HSO students.
- **Main Technologies:** [Typst](https://typst.app/) (v0.12.0+), YAML (for data), CSL (for citations).
- **Architecture:** 
  - **Modular Components:** Reusable Typst fragments (declaration, glossary, etc.) in `src/components/`.
  - **Faculty Styles:** Specific formatting for EMI, MV, and W faculties in `src/styles/`.
  - **Localization:** Support for German ("de") and English ("en") via `src/locales/`.
  - **Context-Driven:** Uses a `ctx` object to propagate metadata and configuration throughout the rendering pipeline.

## Building and Running

As a Typst project, there is no traditional build step. The following commands are relevant:

- **Compilation:** 
  ```bash
  typst compile template/main.typ
  ```
- **Local Installation:** Copy the repository content to your local Typst package directory:
  - Windows: `%LOCALAPPDATA%\typst\packages\local\hso-thesis\0.1.0\`
  - macOS: `~/Library/Application Support/typst/packages/local/hso-thesis/0.1.0/`
  - Linux: `~/.local/share/typst/packages/local/hso-thesis/0.1.0/`
- **Project Initialization:**
  ```bash
  typst init @local/hso-thesis:0.1.0
  ```

## Development Conventions

- **State Management:** Metadata is collected in a `thesis-info` object. Avoid global state; prefer passing the `ctx` dictionary.
- **Component Design:** Components in `src/components/` should be functional and receive a `ctx` argument.
- **Localization:** Never hardcode strings. Use `ctx.strings.[key]` and update `src/locales/*.toml`.
- **Naming:** Follow kebab-case for Typst files and variable names (e.g., `thesis-info`, `lib.typ`).

### Actionable Workflows:

- **Adding a New Component:**
  1. Create a new file in `src/components/`.
  2. Import it in `src/lib.typ` and add it to the `components` dictionary.
  3. Integrate it into the `chapters` list of the relevant styles in `src/styles/`.

- **Adding a New Style:**
  1. Create a new file in `src/styles/`.
  2. Implement `setup`, `style-content`, and the `style` function.
  3. Register the style in `src/lib.typ` in the `styles` dictionary.

## Key Files

- `typst.toml`: Defines the package metadata and template entry point.
- `src/lib.typ`: The primary API. Exports `thesis`, `thesis-info`, and faculty constants.
- `src/styles/`: Contains `emi.typ`, `mv.typ`, and `w.typ` which define the visual identity of each faculty.
- `src/locales/`: TOML files containing translated strings for UI elements.
- `template/`: A complete boilerplate for a thesis, including sample chapters and configuration.

## Usage Guidelines

- Always use the `thesis-info` helper to ensure all mandatory metadata is present.
- The `thesis` function is the main entry point; it handles the ordering of pages, front matter, and body content based on the selected `style`.
- Page numbering is controlled via `src/utils.typ` (`thesis-page-numbering`) to switch between Roman and Arabic numerals.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MarkusPorti)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MarkusPorti)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
