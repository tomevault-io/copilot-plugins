## serious-shit

> This is the static website for **Serious S.H.I.T.** (Super Hyper Incredible Things), an experimental music, circuit-bending, and electronics brand by Yaniv Schonfeld. The site serves as a hub for workshops, gear modification services, and selling unique noise devices.

# Gemini Context: Serious S.H.I.T. Website

## Project Overview
This is the static website for **Serious S.H.I.T.** (Super Hyper Incredible Things), an experimental music, circuit-bending, and electronics brand by Yaniv Schonfeld. The site serves as a hub for workshops, gear modification services, and selling unique noise devices.

*   **Type:** Static Website (HTML, CSS, Vanilla JS).
*   **Platform:** Hosted on GitHub Pages.
*   **Languages:** Bilingual - Hebrew (RTL) and English (LTR).
*   **Aesthetic:** Glitch art, high contrast, "defective-on-purpose," raw, punk.

## Architecture & Conventions

### Tech Stack
*   **Frontend:** Standard HTML5, CSS3 (Variables), Vanilla JavaScript (ES6+).
*   **No Frameworks:** No React, Vue, or build tools. Pure web standards.
*   **Data:** JSON files in `content/` act as a lightweight CMS for galleries and workshops.

### Key Features
1.  **Bilingual System:**
    *   Content is duplicated in HTML with `.lang-en` and `.lang-he` classes.
    *   CSS controls visibility based on `body[data-lang="en|he"]`.
    *   JavaScript handles the toggle and persists preference in `localStorage` (`sshit-lang`).
2.  **Theme System:**
    *   Dark/Light mode via `data-theme` attribute on `<html>`.
    *   CSS Custom Properties handle colors.
    *   Persisted in `localStorage` (`sshit-theme`).
    *   **Default:** Light mode (forced default, ignores system preference initially).
    *   **Dark Mode:** Uses dark grey (`#121212`) background instead of pure black.
3.  **Booking Logic:**
    *   URL parameters (e.g., `?book=fuzz-pedal`) auto-select workshop options and scroll to the contact form.

### File Structure
*   `*.html`: Main page templates (index, mods, sales, workshops, etc.).
*   `css/`: Stylesheets. `style.css` is the main file; `glitch.css` likely handles specific effects.
*   `js/main.js`: Core logic for global state (lang/theme), gallery, and content fetching.
*   `content/*.json`: Data files for dynamic content sections (e.g., `site.json`, `workshops.json`).
*   `images/`: Assets organized by category.

## Development & Usage

### Running Locally
Since this is a static site, any static file server works.
```bash
# Python
python -m http.server 8000

# Node.js
npx serve
```

### Deployment
*   The project is deployed via GitHub Pages.
*   **Action:** Simply push changes to the `main` branch.

### Coding Style
*   **HTML:** Semantic, accessible. Ensure all text content is available in both languages.
*   **CSS:** Use CSS Variables for theming. Keep it raw but functional.
*   **JS:** Vanilla JS. Avoid adding dependencies unless absolutely necessary.
*   **Content:** Tone should be direct, honest, and slightly irreverent ("Nothing Is Holy").

## Important Context
*   **Yaniv Schonfeld:** The artist behind the project.
*   **Locations:** MEZO Haifa (Sirkin 3) - Main Workshop. Previous/CV: Matmon (TLV), Musrara (JLM).
*   **Core Offerings:**
    *   Workshops (ESP32 Synth, Pedal Building).
    *   Custom Mods (Circuit bending).
    *   Products (The Fuzilator PukeMachine).

## Helper Functions (Global)
`window.SSHIT` exposes:
*   `t(obj, lang)`: Extracts translated string from a content object (e.g., `{en: "Hi", he: "היי"}`).
*   `loadContent(file)`: Fetches JSON from the `content/` directory.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/willbearfruits)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/willbearfruits)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
