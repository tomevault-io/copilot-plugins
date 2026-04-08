## google-search-assistant-extension

> This project is a Chrome browser extension named **Search Flow**. It is built using **TypeScript**, HTML, and CSS. The primary goal of the extension is to enhance and streamline the user's search workflow by providing features like quick site-specific searches, multi-platform searching, and a visual query builder for advanced search operators.

# Gemini Code Assistant Context

## Project Overview

This project is a Chrome browser extension named **Search Flow**. It is built using **TypeScript**, HTML, and CSS. The primary goal of the extension is to enhance and streamline the user's search workflow by providing features like quick site-specific searches, multi-platform searching, and a visual query builder for advanced search operators.

The project is structured to separate source files from distributable build files:
- `src/`: Contains all the source TypeScript (`.ts`), HTML, and CSS files.
- `dist/`: Contains the compiled and packaged files, ready to be loaded as an unpacked extension in Chrome.
- `assets/`: Contains static assets like icons.

## Building and Running

This project uses `npm` for dependency management and running scripts.

- **Installation:**
  ```shell
  npm install
  ```

- **Building the extension:**
  The following command cleans the `dist` directory, compiles the TypeScript files, and copies all necessary assets (`.html`, `.css`, images, etc.) to the `dist` folder.
  ```shell
  npm run build
  ```

- **Running the extension:**
  1.  Open Google Chrome and navigate to `chrome://extensions`.
  2.  Enable "Developer mode" using the toggle in the top-right corner.
  3.  Click on "Load unpacked".
  4.  Select the `dist` directory from this project.

## Development Conventions

- **Code Formatting:** The project uses **Prettier** for automatic code formatting. It is recommended to format files before committing.
- **Commit Messages:** Commits must follow the **Conventional Commits** specification (e.g., `feat(popup): add new button`). This is enforced by `commitlint` and `husky` on each commit.
- **TypeScript:** All new logic should be written in TypeScript. The configuration is defined in `tsconfig.json`, which compiles to ESNext modules and targets modern browsers.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zae-hyeong)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zae-hyeong)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
