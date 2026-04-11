## tswebui

> `ts-web-ui` is a library of Framework-Agnostic Web Components (Custom Elements) designed for building data-driven user interfaces. It focuses on configuration-driven development, allowing developers to define complex UI elements like forms and tables using JSON objects.

# Project Overview

`ts-web-ui` is a library of Framework-Agnostic Web Components (Custom Elements) designed for building data-driven user interfaces. It focuses on configuration-driven development, allowing developers to define complex UI elements like forms and tables using JSON objects.

The project also includes a standalone visual editor (`ts-form-editor`) to facilitate the creation of form configurations.

## Architecture

*   **Core Components:** Built as native Web Components (Vanilla JS/ES Modules), making them compatible with any framework (React, Vue, Angular, or no framework).
*   **Form Editor:** A React application (TypeScript + Tailwind CSS) for visually designing forms.
*   **Bundling:** Uses [Vite](https://vitejs.dev/) to bundle the individual components into ES modules located in `dist/`.
*   **Documentation:** Uses [Storybook](https://storybook.js.org/) for component development, testing, and documentation.

## Key Components

The library exports the following main components:

*   **`ts-form`**: A powerful form generator that renders dynamic forms based on a JSON layout and field configuration. Supports validation, tabs, and various input types.
*   **`ts-table`**: A data table component with support for pagination, sorting, and row selection.
*   **`ts-window`**: A window/dialog component.
*   **`ts-form-wrapper`**: A wrapper component for the form.

## Directory Structure

*   `packages/`: Contains the source code for the individual components.
    *   `ts-form/`: Source for the `ts-form` component.
    *   `ts-table/`: Source for the `ts-table` component.
    *   `ts-window/`: Source for the `ts-window` component.
    *   `ts-form-editor/`: A separate React application for visually creating `ts-form` JSON configurations.
*   `storybook/`: Contains Storybook stories (`.stories.js`) and documentation (`.mdx`) for the components.
*   `dist/`: The build output directory containing the bundled JavaScript files.
*   `docs/`: The static build of the Storybook documentation.

## Building and Running

### Main Library & Storybook

1.  **Install Dependencies:**
    ```bash
    npm install
    ```

2.  **Run Storybook (Development):**
    Starts the Storybook dev server on port 3000.
    ```bash
    npm run storybook
    ```

3.  **Build Library:**
    Bundles the components into the `dist/` directory using `vite.bundle.config.js`.
    ```bash
    npm run bundle
    ```

4.  **Build Storybook:**
    Builds the static Storybook documentation to `docs/`.
    ```bash
    npm run build-storybook
    ```

### Form Editor Application

The form editor is a separate application located in `packages/ts-form-editor`.

1.  **Navigate to directory:**
    ```bash
    cd packages/ts-form-editor
    ```

2.  **Install Dependencies:**
    ```bash
    npm install
    ```

3.  **Run Development Server:**
    ```bash
    npm run dev
    ```

4.  **Build:**
    ```bash
    npm run build
    ```

## Development Conventions

*   **Language:** The core components are written in JavaScript (ES Modules). The editor is written in TypeScript.
*   **Documentation:** Some documentation (like `ts-form-readme.md`) is in Czech, but the codebase and configuration keys generally use English.
*   **Styling:** The editor uses Tailwind CSS. The core components likely use standard CSS or Shoelace design tokens (referenced in `ts-form-readme.md`).
*   **Package Management:** The root uses `npm`. The `ts-form-editor` also has its own `package-lock.json`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/janbkrejci)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/janbkrejci)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
