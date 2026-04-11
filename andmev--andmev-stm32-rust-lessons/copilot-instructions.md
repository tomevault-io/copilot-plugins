## andmev-stm32-rust-lessons

> This project is a multilingual static educational site for learning STM32 microcontroller programming with Rust. It's built for performance, accessibility, and ease of content management.

# Project Context: STM32 Rust Lessons

This project is a multilingual static educational site for learning STM32 microcontroller programming with Rust. It's built for performance, accessibility, and ease of content management.

## 🚀 Tech Stack

- **Framework:** [Astro 5](https://astro.build) (Static Site Generation)
- **Styling:** [TailwindCSS 4](https://tailwindcss.com) (Vite-based integration)
- **UI Components:** [Preact](https://preactjs.com) (for interactive islands)
- **Content:** [MDX](https://mdxjs.com) (Markdown + JSX)
- **Testing:** [Vitest](https://vitest.dev)
- **I18n:** Custom auto-detection system based on Content Collections
- **Deployment:** GitHub Pages

## 🛠️ Key Commands

| Command                           | Action                                         |
| :-------------------------------- | :--------------------------------------------- |
| `npm run dev`                     | Start development server at `localhost:4321`   |
| `npm run build`                   | Build for production (local/generic)           |
| `GITHUB_PAGES=true npm run build` | Build specifically for GitHub Pages deployment |
| `npm run test`                    | Run tests in watch mode                        |
| `npm run lint`                    | Run ESLint and check Astro files               |
| `npm run format`                  | Format codebase with Prettier                  |
| `npm run type-check`              | Validate TypeScript types                      |

## 🏗️ Architecture & Conventions

### Content Collections (`src/content/`)

- **lessons**: `src/content/{lang}/lessons/*.mdx`. Schema includes `title`, `description`, `order`, `category`, and SEO metadata.
- **pages**: `src/content/{lang}/{home,about}.mdx`. Static pages for each language.
- Managed via `src/content/config.ts` using `glob` loaders.

### Multilingual System

- **Routing**: Root `/` serves English. Other languages use `/[lang]/` (e.g., `/es/`, `/uk/`).
- **Detection**: Languages are auto-detected by scanning the `src/content/` directory.
- **Utilities**: `src/utils/languages.ts` handles language mapping and detection.
- **Persistence**: The language picker (`LanguagePickerIsland.tsx`) uses `transition:persist` to maintain state.

### Deployment (GitHub Pages)

- **Base Path**: `astro.config.mjs` prepends `/andmev-stm32-rust-lessons` when `GITHUB_PAGES=true`.
- **Links**: Always use the `withBase()` helper (from `src/utils/url.ts`) for internal links in layouts/components.

### Styling & Design

- **Tailwind 4**: Configured via `@tailwindcss/vite` plugin.
- **Typography**: Uses `@tailwindcss/typography` with a custom `.lesson-prose` class for MDX content.
- **Fonts**: Instrument Serif (Headings), JetBrains Mono (Code), PT Sans (Body).

## CRITICAL: Required MCP Tool Usage

**BEFORE making ANY changes to this project, you MUST follow this three-step workflow:**

### Step 1: Initial Analysis with `sequential-thinking` MCP

**ALWAYS** use the `mcp__sequential-thinking__sequentialthinking` tool to:

- Analyze the user's request and break it down
- Understand the full context and implications
- Think through potential solutions and approaches
- Identify which libraries and technologies are involved
- Determine what documentation you need to fetch

### Step 2: Fetch Documentation with `context7` MCP

**ALWAYS** consult the `context7` MCP tools to get up-to-date documentation for:

**Core Technologies:**

- `tailwindcss` (v4.1.18) - Including @layer directives, @apply usage, and utility classes
- `astro` (v5.16.6) - Content collections, routing, MDX integration
- `@astrojs/mdx` (v4.3.13) - MDX processing and configuration
- `@astrojs/preact` (v4.1.3) - Preact integration with Astro
- `@tailwindcss/typography` (v0.5.19) - Prose class and typography utilities
- `preact` (v10.28.1) - Component development

**Plugins & Tools:**

- `rehype-pretty-code` - Code syntax highlighting
- `rehype-slug` - Heading ID generation
- `rehype-external-links` - External link handling
- `remark-github-blockquote-alert` - Blockquote alerts

**Process:**

1. First use `mcp__context7__resolve-library-id` with the library name and your query
2. Then use `mcp__context7__query-docs` with the returned library ID
3. Do NOT call these tools more than 3 times per library per question

### Step 3: Synthesis with `sequential-thinking` MCP (Again!)

**ALWAYS** use `mcp__sequential-thinking__sequentialthinking` tool AGAIN after fetching docs to:

- Synthesize all the information from the documentation
- Integrate the fresh knowledge with the codebase understanding
- Develop a complete, step-by-step implementation plan
- Identify potential issues or edge cases
- Ensure the approach aligns with best practices from the docs

### Step 4: Save State with `memory` MCP

**ALWAYS** use the `save_memory` tool at the end of the task to:

- Store a concise summary of the project state (achievements, technical approach).
- Record failed attempts or rejected libraries to **prevent looping**.
- Define the immediate "Next Steps" for future sessions.
- **CRITICAL:** This ensures you do not repeat work or make the same mistakes in future interactions.

**Example Workflow:**

```
User asks: "Update the lesson layout styling"

STEP 1: Use sequential-thinking to analyze:
   - What files are involved? (global.css, lesson template)
   - What technologies are being used? (Tailwind, Typography plugin)
   - What do I need to know? (How @layer works, how prose class works)

STEP 2: Use context7 to fetch docs:
   - Query tailwindcss docs about @layer and @apply behavior
   - Query @tailwindcss/typography docs about prose class and max-width
   - Query astro docs if layout changes affect routing

STEP 3: Use sequential-thinking AGAIN to synthesize:
   - Now I understand @layer components can be overridden by utilities
   - I know prose class adds max-width: 65ch by default
   - I can use "max-w-none" utility to override it
   - The parent wrapper has max-w-[min(82vw,68rem)] constraint
   - Best approach: use "@apply prose max-w-none" in CSS
   - This will give full width within parent container
   - Implementation plan: Edit global.css line 275, add max-w-none

STEP 4: Implement the solution with complete understanding

STEP 5: Save memory: "Updated lesson layout using @apply prose max-w-none in global.css to override default max-width. Verified against Tailwind docs."
```

### Why This Matters

- **Sequential-thinking (1st)**: Understand the problem and identify knowledge gaps
- **Context7**: Fetch the LATEST, accurate documentation
- **Sequential-thinking (2nd)**: Synthesize everything into a complete implementation plan
- **Memory MCP**: Preserves context to avoid repetitive loops and redundant work
- **Together**: These tools ensure deep understanding and continuity, preventing mistakes and lost progress

## 📁 Key Directories

- `src/components/`: Astro and Preact UI components.
- `src/layouts/`: Page skeletons (e.g., `BaseLayout.astro`).
- `src/pages/`: File-based routing (mostly dynamic `[lang]` routes).
- `src/styles/global.css`: Main CSS entry point with Tailwind layers and theme variables.
- `src/utils/`: Core logic for I18n, URL handling, and validation.

## 🧠 Memory MCP Usage

### ROLE & OBJECTIVE

You are an advanced AI assistant integrated with a Memory MCP. Your primary goal is to ensure seamless continuity across different conversation sessions and to work efficiently by retaining context of completed work.

### MEMORY MANAGEMENT PROTOCOLS

1.  **Proactive State Saving:**
    - At the end of a significant task, code implementation, or reasoning chain, you must proactively store a concise summary of the current project state into your long-term memory.
    - Always capture: What was achieved, the specific technical approach used, and the immediate "Next Steps" required to resume work.

2.  **Anti-Looping & Error Avoidance:**
    - Explicitly record "What didn't work" (failed attempts, rejected libraries, bugs encountered).
    - Before proposing a solution, consult your memory to ensure you are not suggesting a method that was previously attempted and discarded.

3.  **Session Handoff:**
    - When the user indicates the session is ending (or after completing a major milestone), generate a "Checkpoint Memory." This should be a snapshot that allows the user to say "Continue" in a new chat and have you pick up exactly where you left off without needing re-explanation.

4.  **Shared Memory & Context Synchronization:**
    - **CRITICAL:** Treat the Memory MCP as a **shared resource** between all AI agents (Gemini, Claude, etc.).
    - Before starting work, assume the previous context might have been set by a different agent.
    - Write memory entries in a neutral, standardized format that any agent can understand.
    - **Objective:** Enable seamless switching between Gemini and Claude while maintaining a unified project history and context.

### OPERATIONAL RULES

- **Always check memory first:** Before answering complex queries, scan for existing project context.
- **Update, don't just add:** If the project status changes, update the existing memory context rather than creating conflicting duplicate entries.
- **Be explicit:** When saving to memory, use clear tags like [COMPLETED], [FAILED], [NEXT_STEPS] to make retrieval easier.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andmev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/andmev)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
