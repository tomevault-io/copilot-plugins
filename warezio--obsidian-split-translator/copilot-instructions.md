## obsidian-split-translator

> The Obsidian Split Translator Plugin provides real-time translation capabilities directly within the Obsidian note-taking application. It allows users to translate either the entire current note or a selected portion of text. The plugin leverages an unofficial Google Translate API to perform translations, with configurable source and target languages. To maintain the integrity of code snippets, it includes a `MarkdownMasker` that temporarily masks code blocks and inline code before sending text to the translation service. The translated content is streamed into a dedicated split view within Obsidian, and the plugin attempts to synchronize scrolling between the original note and the translated view.

# Project: Obsidian Split Translator Plugin

## Project Overview

The Obsidian Split Translator Plugin provides real-time translation capabilities directly within the Obsidian note-taking application. It allows users to translate either the entire current note or a selected portion of text. The plugin leverages an unofficial Google Translate API to perform translations, with configurable source and target languages. To maintain the integrity of code snippets, it includes a `MarkdownMasker` that temporarily masks code blocks and inline code before sending text to the translation service. The translated content is streamed into a dedicated split view within Obsidian, and the plugin attempts to synchronize scrolling between the original note and the translated view.

**Key Technologies:**
*   TypeScript
*   Obsidian API
*   esbuild (for bundling)
*   Unofficial Google Translate API

## Building and Running

This project uses `esbuild` to compile TypeScript into JavaScript for the Obsidian plugin environment.

**Build Commands:**

*   **Development Mode (with watch):** Compiles `main.ts` into `main.js`, bundles dependencies, and watches for changes, automatically recompiling on file modifications.
    ```bash
    npm run dev
    # or
    esbuild main.ts --bundle --outfile=main.js --format=cjs --external:obsidian --watch
    ```
*   **Production Build:** Compiles `main.ts` into `main.js` and bundles dependencies for release.
    ```bash
    npm run build
    # or
    esbuild main.ts --bundle --outfile=main.js --format=cjs --external:obsidian
    ```

**To run the plugin in Obsidian:**
1.  Perform a production build (`npm run build`).
2.  Copy `main.js`, `manifest.json`, and `styles.css` into your Obsidian vault's plugin folder (e.g., `.obsidian/plugins/obsidian-split-translator/`).
3.  Enable the plugin in Obsidian's settings.

## Development Conventions

*   **Language:** TypeScript is used for development.
*   **Build Tool:** `esbuild` is used for bundling and transpilation.
*   **Code Style:** Follows standard TypeScript/JavaScript practices.
*   **Obsidian API Usage:** Interacts with Obsidian's API for commands, settings, and view management.
*   **Translation Handling:** Text content is chunked and masked (for code blocks) before translation to ensure efficiency and accuracy.
*   **Settings:** Plugin settings (source/target languages) are managed through Obsidian's native settings interface and are persisted.

## Usage

**Method 1: Translate the current note**
```
Cmd/Ctrl + P → "LibreTranslate: Translate current note"
```

**Method 2: Translate selected text**
```
Select text → Cmd/Ctrl + P → "LibreTranslate: Translate selection"
```

**Configuring Languages:**
Users can select source and target languages from the plugin's settings tab within Obsidian. Default source is "Auto-detect", and default target is "English".

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/warezio) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
