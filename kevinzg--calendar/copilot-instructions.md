## calendar

> This document outlines the plan for creating a website to generate calendar PDFs with user-customizable special dates.

# Gemini Calendar PDF Generator

This document outlines the plan for creating a website to generate calendar PDFs with user-customizable special dates.

## 1. Project Setup

The project is already set up as a SvelteKit application. We will build upon this existing structure. `pnpm` is used as the package manager.

## 2. UI for Calendar Customization

- **File:** `src/routes/+page.svelte`
- **Component:** Create a Svelte component that allows users to:
    - Select the year and month for the calendar.
    - Add special dates with a title and an icon.
    - A "Generate PDF" button.

## 3. PDF Generation

Users will save the page as PDF and optionally print using the browser print dialog.

## 4. Special Date Icons

The project already contains some Svelte components for icons in `src/lib/assets`. We will use these components to allow the user to select an icon for their special dates.

- **Components:**
    - `src/lib/assets/Cake.svelte`
    - `src/lib/assets/Circle.svelte`
    - `src/lib/assets/Heart.svelte`
    - `src/lib/assets/Star.svelte`
    - `src/lib/assets/Tree.svelte`
- **Functionality:**
    - Create a component that displays these icons for the user to select.
    - When a user adds a special date, they can choose one of these icons to be displayed on that date in the PDF.

## 5. Deployment

The project already has a GitHub workflow for deployment (`.github/workflows/deploy.yaml`). Once the features are implemented, we can deploy the application to a hosting provider like Vercel or Netlify.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kevinzg)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kevinzg)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
