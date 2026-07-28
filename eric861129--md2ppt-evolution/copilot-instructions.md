## md2ppt-evolution

> A static web application (GitHub Pages) for converting Markdown to professional PowerPoint (PPTX) presentations. It is the spiritual successor to `BookPublisher_MD2Docx`, inheriting its UI/UX logic but targeting slide generation with advanced layout capabilities.

# MD2PPT - Gemini Development Context

## Project Overview
A static web application (GitHub Pages) for converting Markdown to professional PowerPoint (PPTX) presentations. It is the spiritual successor to `BookPublisher_MD2Docx`, inheriting its UI/UX logic but targeting slide generation with advanced layout capabilities.

## Tech Stack
- **Frontend:** React 18+, TypeScript, Vite.
- **Styling:** **Tailwind CSS** (Utility-first), with CSS Variables for dynamic theming (Dark/Light/Neon modes).
- **State Management:** **Zustand** (Editor content, slide configuration, global settings).
- **Core Libraries:** - `pptxgenjs` (Client-side PPTX generation).
    - `@monaco-editor/react` (Code editor experience).
    - `unified` / `remark-parse` / `remark-gfm` (Robust AST parsing for advanced layouts).
    - `mermaid` (Diagram rendering).
- **Testing:** Vitest, React Testing Library.

## Key Architectures
- **Registry Pattern:** Used for PPTX generation logic (`services/pptx/registry.ts`). Handlers for different Markdown nodes (headers, lists, code blocks, custom layouts) are registered in `services/pptx/builders/index.ts`.
- **Slide Object Model (SOM):** An intermediate data structure between Markdown AST and the final PPTX output, managed in `services/parser/som.ts`.
- **Dual Rendering:** 1. **Preview:** HTML/CSS mapping of the SOM using Tailwind (Left/Right split view).
    2. **Export:** PptxGenJS mapping of the SOM to actual slide objects.

## Branching & Workflow (Per CONTRIBUTING.md)
- **`main`**: Production (Deploys to GitHub Pages).
- **`dev`**: Integration branch.
- **Feature Branch:** `feature/FeatureName` or `dev_feature_yyyyMMdd_XXXX`.
- **Version Bumping:** - Minor for new features (e.g., new layout support).
    - Patch for bug fixes.
- **Version Bump Checklist (MUST Update All):** 1. **`package.json`**: `version` field.
    2. **`README.md`**: Title line & Version Badge.
    3. **`CHANGELOG.md`**: Add new section.
    4. **`src/constants/appInfo.ts`**: Update internal version constant.

## Coding Standards
- **Functional Components:** Use React Hooks exclusively. Avoid Class components.
- **Builder Pattern:** Follow existing patterns in `services/pptx/builders/` when adding new Markdown syntax support.
- **Type Safety:** Strict TypeScript usage. Define interfaces for all Prop types and Slide Models in `types/`.
- **Tailwind:** Use utility classes for layout; abstractions allowed for repeated component styles.

## Shared Context
- **Project Heritage:** Retains the `BookPublisher` sidebar/toolbar layout but with a modernized "Dark/Cyber" aesthetic by default.
- **Parsing Logic:** Uses `remark` to generate AST, then transforms to "Slides". Custom syntax (e.g., `::: split`) is handled via remark plugins or custom directives.
- **Styling Source:**
    - Slide Masters (Layouts) are defined in `services/pptx/masters.ts`.
    - UI Theme colors are defined in `tailwind.config.js` and `src/styles/theme.css`.
- **Documentation**:
    - `docs/PROJECT_OVERVIEW.md`: High-level feature summary.
    - `docs/ARCHITECTURE.md`: Technical architecture and workflows.

---
> Source: [eric861129/MD2PPT-Evolution](https://github.com/eric861129/MD2PPT-Evolution) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
