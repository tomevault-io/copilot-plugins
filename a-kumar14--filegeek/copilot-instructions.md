## filegeek

> **Objective:** Transform FileGeek into a high-density "Knowledge Operating System" by reengineering the UI into a modular Bento Grid architecture, establishing a dynamic theme/font engine, and stabilizing multi-provider AI routing.

# Antigravity Task: FileGeek "Knowledge OS" Final Reengineering

**Objective:** Transform FileGeek into a high-density "Knowledge Operating System" by reengineering the UI into a modular Bento Grid architecture, establishing a dynamic theme/font engine, and stabilizing multi-provider AI routing.

---

### 1. Layout Overhaul: The Bento Grid & Discovery Dashboard
* **Main Layout Refactor (`MainLayout.js`)**: 
    * Replace the current `react-resizable-panels` with a **CSS Grid Bento Layout**.
    * **The Canvas (Center)**: 
        * **Active State**: Display the `FileViewer`.
        * **Discovery Dashboard (Empty State)**: When no file is selected, show a dashboard with Bento-style cards for:
            * **Mastery Heatmap**: Visual progress of known vs. remaining flashcards.
            * **Session Pulse**: Total interactions and active session duration.
            * **Resource Inventory**: Recent documents and generated artifacts.
    * **Activity Stream (Right)**: Merge `ChatPanel` and `ArtifactPanel` into a single unified "Reasoning Log".
* **Warp-Inspired Input**: Move the chat input to a **Floating Command Bar** at the bottom-center of the screen with a `//` command prefix.

### 2. Branding & Aesthetic (Brutalist Design)
* **Theme & Font Engine**:
    * **Context Update (`ThemeContext.js`)**: Refactor to support **CSS Variable Injection** for themes and fonts.
    * **Settings Update (`SettingsContent.js`)**: Add an `// APPEARANCE` section with dropdown menus for:
        * **Theme**: `BRUTALIST_DARK` (Default), `PAPER_WHITE`, `SOLARIZED`.
        * **Typography**: `JETBRAINS_MONO` (Default), `INTER_SANS`, `OPEN_DYSPLEXIC`.
    * **TopBar Integration**: Add a `PaletteIcon` to `TopBar.js` for quick-swapping presets.
* **Brand Realignment**: Globally replace all "JM Solutionss" branding with **FileGeek** and enforce the **Geek Green (`#00FF00`)** accent color.

### 3. Feature Cleanup & Workflow Stabilization
* **Remove Legacy Features**: Strip all logic and UI components related to "Research Notes" and `HighlightsContext` from `MainLayout.js`, `TopBar.js`, and `CommandPalette.js`.
* **AI Routing Stability (`ai_service.py`)**: 
    * Implement **Lazy Client Loading** for OpenAI and Gemini clients.
    * Ensure `answer_with_tools` dynamically selects the provider based on the `model_override` string without requiring server re-initialization.
* **Performance Hardening**: Ensure `PdfViewer.js` uses **Virtual Scrolling** to maintain 60fps performance on documents exceeding 100 pages.

### 4. Workflow Enhancements
* **Agentic Transparency**: Implement "Thinking Blocks" in the chat stream to display real-time status updates (e.g., `[VECTOR_SEARCHING...]`, `[GENERATING_QUIZ...]`).
* **Active Recall Integration**: Add a toggle in the `TopBar` for **Socratic Mode**, forcing the AI to provide page-referenced hints instead of direct answers.

---

**Execution Order:**
1. **Clean**: Remove the Research Notes system.
2. **Structure**: Implement the Bento Grid and the Warp-style Discovery Dashboard.
3. **Personalize**: Deploy the Theme/Font selection engine.
4. **Optimize**: Finalize the AI routing and PDF virtualization.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/A-Kumar14)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/A-Kumar14)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
