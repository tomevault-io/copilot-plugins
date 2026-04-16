## grammatica-italiana

> An interactive web application designed to help English speakers learn Italian grammar. The tool features dynamic charts, switchable language support (IT/EN), and interactive exercises with automatic validation.

# Italian Grammar Learning Tool (Grammatica Italiana)

## Project Overview
An interactive web application designed to help English speakers learn Italian grammar. The tool features dynamic charts, switchable language support (IT/EN), and interactive exercises with automatic validation.

## Key Features
*   **Interactive Grammar Charts:**
    *   **Articles:** Comprehensive guide to Definite and Indefinite articles.
    *   **Verbs:** Deep dive into 8 different categories:
        *   Present Tense (including regular patterns and a clickable grid of 19 common irregulars + 6 -isc- verbs).
        *   Passato Prossimo (Avere vs Essere, with a list of irregular past participles).
        *   Imperfetto (Regular and common irregulars like essere, bere, dire, fare).
        *   Simple Future (Regular patterns and phonetic rules).
        *   Present Conditional.
        *   Present Subjunctive.
        *   Past Subjunctive.
        *   Reflexive Verbs (Construction rules and compound tense usage).
    *   **Prepositions:** Simple and Articulated preposition tables.
*   **Vocabulary Section:** Categorized word lists (Clothing, Health, etc.) with gender labels and translations.
*   **Exercises (Esercizi):** 
    *   Interactive modules for Lesson 1 (Italian 202).
    *   Features: Shuffled matching tasks, fill-in-the-blanks with individual checking, and writing prompts linked to the Brightspace Forum.
*   **Multi-language Support:** Full interface and explanation translations for both English and Italian students.

## Deployment
*   **Automatic Deployment:** The project is configured with GitHub Actions to automatically deploy to GitHub Pages on every push to the `main` branch. 
*   **Workflow:** Every code modification must be committed and pushed to trigger the update on: https://gi.beatricecarnelutti.com

## Architecture & Tech Stack
*   **Framework:** React 19
*   **Language:** TypeScript
*   **Build Tool:** Vite
*   **Styling:** Tailwind CSS v4
*   **Animation:** Framer Motion
*   **State Management:** 
    *   `ChartContext`: Manages navigation state (`ViewState`) across the app.
    *   `LanguageContext`: Manages global localization (IT/EN).

## Core File Structure
*   `src/App.tsx`: Main layout, navigation, and top-level routing.
*   `src/ChartContext.tsx`: Centralized view state and context provider.
*   `src/LanguageContext.tsx`: I18n provider and translation hook.
*   `src/translations.ts`: Master dictionary for all localized text.
*   `src/vocabularyData.ts`: Database for vocabulary items.
*   `src/ArticlesChart.tsx`: Logic for Italian articles.
*   `src/VerbsMenu.tsx`: Sub-navigation for the verbs section.
*   `src/Verbs...Chart.tsx`: Specialized components for each verb tense.
*   `src/exercises/Exercises.tsx`: Container for the exercises section.
*   `src/exercises/Lesson1Exercises.tsx`: Interactive exercise items and validation logic.

## Licensing
*   **Web App:** MIT License.
*   **Content:** Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bcarnelutti) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
