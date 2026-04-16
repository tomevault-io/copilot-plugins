## portfolio-website

> This project is a personal portfolio website for Manikanta Gonugondla, an Electrical & Electronics Engineer specializing in FPGA design, embedded systems, and AI integration. The site is built as a single-page React application featuring rich animations and a terminal-style loading sequence.

# Project Context: Portfolio Website

## Overview
This project is a personal portfolio website for Manikanta Gonugondla, an Electrical & Electronics Engineer specializing in FPGA design, embedded systems, and AI integration. The site is built as a single-page React application featuring rich animations and a terminal-style loading sequence.

## Tech Stack
*   **Framework:** React 18 (bootstrapped with Create React App)
*   **Animation:** `framer-motion` (used extensively for scroll reveals and layout transitions)
*   **Styling:** CSS (Global styles in `App.css`, component-specific styles likely inherited or modular)
*   **Testing:** Playwright (End-to-End testing configured in CI)
*   **CI/CD:** GitHub Actions (configured in `.github/workflows/test.yml`)

## Key Files & Structure
*   `src/App.js`: Main application entry point. Manages the initial "terminal" loading state and renders the main page layout.
*   `src/components/`: Directory containing all UI sections and utility components.
    *   `Hero.js`, `About.js`, `Experience.js`, `Projects.js`, `Contact.js`: Main content sections.
    *   `CustomCursor.js`, `DotGrid.js`, `FloatingParticles.js`: Visual effects components.
    *   `Navigation.js`: Main navigation bar.
*   `.github/workflows/test.yml`: Defines the CI pipeline, running sharded Playwright tests across multiple browsers (Chromium, Firefox, WebKit) and mobile viewports.

## Development & Build Commands
*   **Install Dependencies:** `npm install` (or `npm ci` for clean install)
*   **Start Dev Server:** `npm start` (Runs on `http://localhost:3000`)
*   **Build for Production:** `npm build` (Outputs to `build/`)
*   **Run Tests:**
    *   The project uses Playwright for testing.
    *   Run locally: `npx playwright test`
    *   Install browsers: `npx playwright install --with-deps`

## Coding Conventions
*   **Components:** Functional components using React Hooks (`useState`, `useEffect`).
*   **Animations:** Use `framer-motion`'s `motion.div` for animated elements. Common pattern involves `initial`, `whileInView`, and `viewport` props for scroll-triggered animations.
*   **Theme:** The UI features a dark, technical aesthetic ("terminal" loader, dot grids, particle networks) reflecting the user's engineering background.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Manikanta25055) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
