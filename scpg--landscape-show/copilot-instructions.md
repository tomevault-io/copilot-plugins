## landscape-show

> Before beginning any task, I will first review the `AI.md` file in the root directory and follow its checklist. This file contains the primary project overview, technology stack, architecture, and status, and serves as the "single source of truth" for project context.

# Gemini AI Assistant Guidelines - Landscape Show

## 1. Primary Directive

Before beginning any task, I will first review the `AI.md` file in the root directory and follow its checklist. This file contains the primary project overview, technology stack, architecture, and status, and serves as the "single source of truth" for project context.

## 2. My Core Understanding

Based on my analysis, **Landscape Show** is a YAML-driven visualization tool with a decoupled frontend and backend. The most critical feature is the **bidirectional synchronization** between a visual React Flow diagram and a Monaco-powered text editor.

*   **Data Flow (Diagram -> YAML)**: User drags a node -> `onPositionChange` fires -> `useLandscapeStore` calls `updatePositions` API -> Backend's `yaml_service` updates the specific system's `position` in the `.yaml` file.
*   **Data Flow (YAML -> Diagram)**: User types in Monaco editor -> `onChange` fires -> `handleYamlChange` in `App.tsx` debounces the input -> `YAMLParser.parse` is called -> `useLandscapeStore`'s `updateLandscape` action is dispatched -> The `currentLandscape` object is updated, triggering a re-render of the `DiagramCanvas`.

## 3. My Approach to Assisting

*   **Context First**: I will always begin by referencing `AI.md` to ensure my actions align with the project's established architecture and goals.
*   **API-Oriented Enforcement**: I will ensure all new frontend functionality that interacts with the backend does so exclusively through the REST API, adhering to the principle outlined in `AI.md`.
*   **Convention-Driven**: I will strictly adhere to the established coding styles. New backend logic will use the service pattern; new frontend state will be managed in the Zustand store.
*   **Safety and Precision**: I will prioritize data integrity. Changes will respect the backend Pydantic validators and frontend TypeScript types. When modifying files, I will use precise locators and provide sufficient context to avoid unintended changes.
*   **Incremental Changes**: I will break down complex tasks into smaller, logical, and verifiable steps.
*   **Clarity**: I will explain my plan before executing it and summarize the results.

I am ready to assist with the next phase of the project.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/scpg)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/scpg)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
