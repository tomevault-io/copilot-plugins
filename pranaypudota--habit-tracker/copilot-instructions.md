## habit-tracker

> This `GEMINI.md` file serves as the official, unified orchestration document for the Gemini AI Coding Assistant. It dictates the architectural constraints, styling preferences, and required skills for the Habit Tracker project.

# Gemini Agent Configuration (Habit & Expense Tracker)

This `GEMINI.md` file serves as the official, unified orchestration document for the Gemini AI Coding Assistant. It dictates the architectural constraints, styling preferences, and required skills for the Habit Tracker project. 

The agent must parse and abide by these rules before executing any task, writing code, or suggesting architectures, ensuring strict consistency across the project lifecycle.

---

## 1. Project Core Philosophy
1. **Local-First Baseline:** All data lives in a local SQLite file. Do NOT implement cloud synchronization, authentication, or external SaaS database dependencies.
2. **Zero Bloat:** Keep dependencies to an absolute minimum. Prioritize standard built-in functions over third-party packages whenever possible.
3. **High Performance Flow:** Minimize network chatter between frontend and backend. Use batched operations, optimal endpoint querying, and zero-network overhead navigation where applicable.

---

## 2. Frontend Stack & Guidelines
- **Core:** React 18 + TypeScript + Vite.
- **Styling:** TailwindCSS v4 with an emphasis on rich UI.
  - **Aesthetics:** Use vibrant, harmonious OKLCH-based color palettes, smooth dark modes, glassmorphism, and elegant component design. 
  - **Interactivity:** Utilize micro-animations, fluid hover states, and dynamic elements. Always aim for a "premium" feel over a simple MVP.
- **State Management:** Zustand for global state handling. Synchronize thoroughly with the backend.
- **Component Rules:** Modularize UI elements logically inside `src/components`. Maintain strict TypeScript types in `src/types/index.ts`.
- **Performance:** Follow Vite optimization practices. Rely on `lazy()` routing and standard code-splitting techniques natively.

---

## 3. Backend Stack & Guidelines
- **Core:** Python 3.11+ using FastAPI.
- **Package Management:** `uv` via `pyproject.toml`.
- **Database:** Local SQLite using SQLAlchemy 2.0 (async). Always handle async DB sessions efficiently.
- **Data Validation:** Pydantic V2 schemas (`app/schemas/`). Maintain strict typing all the way down.
- **Logging:** Use `loguru` exclusively. Ensure console output is visually clean, color-coded, and avoids unnecessary debug spam.
- **Architecture Details:** Keep code strictly organized by business concern (`models`, `schemas`, `services`, `routers`).

---

## 4. Required Agent Skills & Proficiencies
To maintain the repository efficiently, the agent must prioritize deploying these specific proficiencies over generalized coding tasks:

1. **Advanced UI & Component Engineering:**
   - Crafting modern interfaces natively (glassmorphism, OKLCH gradients).
   - Implementing responsive micro-animations entirely in standard CSS/Tailwind.
2. **High-Performance Architecture:**
   - Structuring batched data-fetching endpoints to push comprehensive "data snapshots".
   - Constructing RESTful paths that support aggressive global state caching.
3. **Local-First Database Operations:**
   - Writing secure, asynchronous SQLAlchemy SQLite queries.
   - Anticipating edge cases with local SQLite threading/locks natively.
4. **Observability & Debugging:**
   - Leveraging `loguru` middleware to intercept HTTP timings without clogging stdout.
5. **Technical Documentation & Planning:**
   - Reading and updating the `.planning/` directory logically before heavy refactors.
   - **Professional Tone:** NEVER use emojis in documentation files, inline comments, or markdown structures. Maintain a strictly professional and technical tone.
6. **Specialized Agency Personas:**
   - Leveraging a repository of 110+ specialized AI agents (e.g., `agency-backend-architect`, `agency-frontend-developer`, `agency-ui-designer`, `agency-seo-specialist`) to perform niche technical roles with high precision and cross-domain expertise.

---

## 5. Protocol & Execution Steps
1. **Analyze First:** Treat every user request with an architectural lens. Does this require a schema update? Is there a performance hit?
2. **No Placeholders:** Generate fully functioning, integrated code. Do not output snippet stubs like `# TODO: implement this logic`.
3. **Anticipate Scale:** Write code that expects future local expansions (e.g., CSV/JSON data exports, CSS-native heatmaps, heavy offline caching). All implementations should remain local, modular, and fast.
4. **Daily Upkeep:** Actively maintain `VersionChangeLog.md` and `TODO.md`. When generating a daily roundup or doing the "usual upkeep", log all architectural/logical changes into the `VersionChangeLog.md`, and ensure the `TODO.md` is accurately tracking pending tasks (organized by task, not date).
   - **Version Change Log Rule:** Entries must follow the "2 W's & 1 H" structure: **What** we changed, **Why** we changed it (what it solves), and **How** it affects the project. Logs should be detailed enough to be understood clearly but with minimal, concise word usage.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pranaypudota) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
