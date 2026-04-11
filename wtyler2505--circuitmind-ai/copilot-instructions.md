## circuitmind-ai

> CircuitMind AI is an intelligent, futuristic workspace for electronics prototyping. It features AI-generated wiring diagrams, interactive component inventory management, and educational pinout exploration. This is a personal project optimized for correctness, craftsmanship, and local-first workflows.

# CircuitMind AI

## Project Overview

CircuitMind AI is an intelligent, futuristic workspace for electronics prototyping. It features AI-generated wiring diagrams, interactive component inventory management, and educational pinout exploration. This is a personal project optimized for correctness, craftsmanship, and local-first workflows.

## User Context & Collaboration

- **User:** Tyler
- **Communication Style:** Clear, plain language. Avoid jargon; define acronyms. Prefer concrete examples and step-by-step explanations.
- **Priorities:** Correctness > Speed. No deadlines.
- **Privacy:** Local-only. No telemetry without explicit request.

## Tech Stack

- **Framework:** React 19, Vite
- **Language:** TypeScript
- **Styling:** Tailwind CSS (v4), PostCSS
- **3D Graphics:** Three.js
- **AI:** Google GenAI SDK (`@google/genai`), `client-vector-search`
- **Testing:** Vitest, Playwright, React Testing Library

## Key Commands

```bash
npm run dev          # Start development server (Requires GEMINI_API_KEY in .env.local)
npm run build        # Build for production
npm run preview      # Preview production build
npm run test         # Run unit tests (Vitest)
npm run test:watch   # Run tests in watch mode
npm run test:visual  # Run visual tests (Playwright)
npm run lint         # Lint code
```

## Architecture

### State Management (React Context)

- **`DiagramContext`**: Manages the wiring diagram state and undo/redo history.
- **`InventoryContext`**: Manages the library of electronic components.
- **`ConversationContext`**: Handles chat sessions and history.
- **`AssistantStateContext`**: Tracks AI status (thinking, idle, etc.).
- **`VoiceAssistantContext`**: Manages voice input/output.
- **`LayoutContext`**: Controls UI layout and sidebars.

### Key Files & Directories

- **`types.ts`**: **CRITICAL**. Contains all core interfaces (`ElectronicComponent`, `WiringDiagram`, `ChatMessage`). Read this first.
- **`App.tsx`**: Root orchestration and layout structure.
- **`contexts/*.tsx`**: Domain state providers.
- **`services/geminiService.ts`**: Main entry point for Gemini AI integration.
- **`hooks/useAIActions.ts`**: Central hub for dispatching AI actions.
- **`conductor/index.md`**: **Conductor Index**. Single source of truth for product vision, tech stack, and active tracks.
- **`CLAUDE.md`**: Detailed architectural documentation, pitfalls, and patterns.

## Development Conventions

- **Coding Style:** TypeScript + React. Use PascalCase for components, `useX` for hooks.
- **Testing:** Verification is mandatory. Tests live near code in `__tests__` directories.
- **Secrets:** `GEMINI_API_KEY` must be in `.env.local`. Never commit secrets.
- **Component Sync:** Updates to components must sync to BOTH `InventoryContext` (source of truth) and `DiagramContext`.

## AI Model Strategy (Gemini)

- **Wiring Diagrams:** `gemini-2.5-pro` (Prioritizes accuracy)
- **General Chat:** `gemini-2.5-flash` (Prioritizes speed)
- **Complex Reasoning:** `gemini-2.5-pro`
- **Image Generation:** `imagen-3.0-generate-001`
- **Video Generation:** `veo-2.0-generate-001`

## Analysis & Exploration Tools

Utilize these tools for deep codebase understanding, refactoring, and quality assurance.

### Codebase Discovery

- **`ast-grep` (sg):** Use for structural code search and rewriting. Prefer over `grep` for finding complex patterns like "all React components using a specific hook".
- **`tree`:** Use to visualize directory structures. Use `-I "node_modules|dist|.git"` to keep output clean.
- **`fd`:** Use for fast file finding. Faster and more intuitive than `find`.
- **`rg` (ripgrep):** Use for lightning-fast text searching. **MANDATORY** for initial context gathering.

### Code Quality & Complexity

- **`lizard`:** Use to analyze cyclomatic complexity. Run on `services/` or `components/` to identify refactoring candidates.
- **`shellcheck`:** Use to validate any `.sh` scripts in the project.
- **`eslint` / `prettier`:** Strictly follow project linting and formatting rules via `npm run lint` and `npm run format`.

### Performance & System

- **`hyperfine`:** Use to benchmark critical paths (e.g., 3D generation logic or worker startup).
- **`btop` / `glances`:** Monitor system resources during heavy 3D rendering or AI inference tasks.

### Utilities

- **`bat`**: Use for syntax-highlighted file viewing in the terminal.

- **`fzf`**: Use for fuzzy finding across logs or file lists.

- **`jq` / `yq`**: Use for processing JSON/YAML configuration and metadata files.

- **`lazygit`**: Use for interactive git management when complex staging/squashing is required.

## Advanced Reasoning & Problem Solving (Clear-Thought)

Utilize these cognitive frameworks to navigate complex architectural decisions, persistent bugs, and high-fidelity creative tasks.

### Structural & Systems Analysis

- **`systems_thinking`**: **MANDATORY** for analyzing cross-context state synchronization (e.g., Diagram vs. Inventory vs. Simulation). Use to map second-order effects of state changes.

- **`mental_model`**: Use "First Principles" to rebuild legacy logic from scratch or "Inversion" to identify how the 3D engine might fail before writing code.

- **`causal_analysis`**: Use to trace the root cause of non-deterministic race conditions in Three.js workers or async AI stream parsing.

### Execution & Debugging

- **`sequential_thinking`**: Use for planning multi-step refactors. Every step must be verified before proceeding to the next.

- **`debugging_approach`**: Use when a bug survives the first fix attempt. Force a structured "Observation -> Hypothesis -> Experiment" loop.

- **`scientific_method`**: Use for performance tuning. Define clear metrics (e.g., "Worker Latency < 50ms"), hypothesize optimizations, and measure results using `hyperfine`.

### Design & Innovation

- **`creative_thinking`**: Use for designing futuristic UI/UX elements that don't have standard web patterns (e.g., neural-link visualizers).

- **`analogical_reasoning`**: Use to bridge electronics engineering concepts with 3D graphics logic (e.g., treating Three.js Groups as PCB sub-assemblies).

### Strategic Decisions

- **`decision_framework`**: Use for high-stakes technical choices (e.g., React 19 vs. older paradigms). Weigh pros/cons against "Correctness > Speed" mandate.

- **`tree_of_thought` / `graph_of_thought`**: Use to explore multiple implementation paths for complex features like the "Semantic Layout Engine" before committing to a single file change.

- **`ethical_analysis`**: Use when implementing privacy-sensitive features or AI data handling to ensure "Local-only" mandate is strictly preserved.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wtyler2505)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wtyler2505)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
