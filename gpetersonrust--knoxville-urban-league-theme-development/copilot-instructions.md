## knoxville-urban-league-theme-development

> This document provides a guide for the Gemini agent to effectively work on the Knoxville Urban League theme. It serves as an index to more specific instructions located in the `GEMINI/` directory.

# Gemini Project Guide

This document provides a guide for the Gemini agent to effectively work on the Knoxville Urban League theme. It serves as an index to more specific instructions located in the `GEMINI/` directory.

## General Agent Instructions

When working on this project, always refer to the specific instruction files within the `GEMINI/` directory based on the file type or task at hand.

-   **For SCSS/CSS related tasks:** Refer to `GEMINI/scss.md`
-   **For PHP related tasks:** Refer to `GEMINI/php.md` (To be created)
-   **For JavaScript related tasks:** Refer to `GEMINI/js.md` (To be created)

## Build Process

The frontend assets are managed with webpack and gulp. The relevant commands are defined in `webpack/package.json`.

-   **`npm run build`**: Compiles the SCSS and JavaScript assets. Run this after making any changes to the `webpack/src/` directory.
-   **`npm run watch`**: Watches for changes in the `webpack/src/` directory and automatically recompiles the assets.
-   **`npm run dev`**: Runs both `webpack --watch` and `gulp` for live reloading.

**To apply any changes to SCSS or JS, you must run `npm run build` inside the `webpack` directory.**

## File Structure Overview

-   **`dist/`**: Contains the compiled CSS and JS assets. **Do not edit files in this directory directly.**
-   **`includes/`**: Contains the core PHP logic for the theme, including post types, meta fields, and theme setup.
-   **`static/`**: Contains static HTML mockups for components. This is for reference and prototyping.
-   **`templates/`**: Contains the main WordPress template files.
-   **`webpack/`**: Contains the source files for the frontend assets.
    -   **`webpack/src/app/js`**: Main JavaScript files.
    -   **`webpack/src/app/scss`**: Main SCSS files.

## WordPress Integration

This is a WordPress theme. The PHP files in `includes/` and `templates/` work together to render the site. The compiled assets from `dist/` are enqueued in `includes/Modules/Assets.php`.

When adding new components, you may need to:

1.  Create the SCSS in `webpack/src/app/scss/components`.
2.  Add any necessary JavaScript to `webpack/src/app/js`.
3.  Run `npm run build` in the `webpack` directory.
4.  Potentially create a new PHP component in `includes/Components` to render the markup.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gpetersonrust)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gpetersonrust)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
