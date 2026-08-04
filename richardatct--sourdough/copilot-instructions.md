## sourdough

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a single-page web application that presents an interactive focaccia bread recipe. The application allows users to:
- Scale the recipe (0.25x to 4x)
- Adjust dough hydration (78-84%)
- Switch between commercial yeast and sourdough starter variations
- View dynamically updated ingredient amounts and instructions

## Architecture

The project consists of a single HTML file (`focaccia_recipe.html`) that includes:
- Embedded CSS for styling (lines 7-403)
- Embedded JavaScript for interactivity (lines 728-1027)
- No external dependencies or build process required

Key JavaScript functions:
- `setYeastType()`: Switches between commercial yeast and sourdough variations
- `setScale()`: Updates recipe quantities based on scaling factor
- `setHydration()`: Adjusts water content for dough consistency
- `updateRecipe()`: Core function that recalculates all ingredient amounts and updates the DOM

## Development Commands

Since this is a static HTML file with no dependencies:
- **Run locally**: Open `focaccia_recipe.html` directly in a web browser
- **Test changes**: Refresh the browser after saving edits
- **No build/lint/test commands required**

## Code Conventions

- All functionality is contained within the single HTML file
- CSS uses BEM-style naming for clarity (e.g., `.scale-btn`, `.ingredients-table`)
- JavaScript uses vanilla JS with no frameworks
- IDs are used for elements that need dynamic updates
- Inline styles in JavaScript for dynamic styling changes

---
> Source: [RichardAtCT/Sourdough](https://github.com/RichardAtCT/Sourdough) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
