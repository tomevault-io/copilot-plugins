## new-resume

> This project is a high-performance, AI-integrated portfolio and resume application for a Cloud Architect/DevOps Engineer. It leverages Next.js 15, React 19, and Google's Genkit for advanced AI orchestration.

# GEMINI.md - Nexus Project (Next.js 15 + Genkit)

This project is a high-performance, AI-integrated portfolio and resume application for a Cloud Architect/DevOps Engineer. It leverages Next.js 15, React 19, and Google's Genkit for advanced AI orchestration.

## 🚀 Project Overview

- **Frontend:** Next.js 15 (App Router), React 19, TypeScript.
- **Styling:** Tailwind CSS with **shadcn/ui** components (Radix UI).
- **AI Core:** **Genkit** with `googleai/gemini-2.5-flash`.
- **Primary Tool:** **Project Synthesis Lab** (`/ai-tool`) - An AI flow that transforms technical specifications into business-oriented value propositions.
- **Prompt Engineering:** Uses the **RCTF framework** (Role, Context, Task, Format) with templates stored in `prompt-hub/`.

## 🛠️ Building and Running

### Development
```bash
# Start Next.js development server (port 9002)
npm run dev

# Start Genkit Developer UI (for testing AI flows)
npm run genkit:dev
```

### Production
```bash
# Build the application
npm run build

# Start production server
npm run start
```

### Quality Control
```bash
# Run linting
npm run lint

# Run TypeScript type checking
npm run typecheck
```

## 📂 Key Directory Structure

- `src/ai/`: Genkit configuration, model definitions, and AI flows.
  - `flows/`: Business logic for AI operations (e.g., `generate-project-summary.ts`).
  - `genkit.ts`: Central Genkit instance with Gemini plugin.
- `src/app/`: Next.js App Router pages.
  - `ai-tool/`: Route for the Project Synthesis Lab.
- `src/components/`:
  - `ai/`: Specialized AI-interactive components.
  - `sections/`: Home page sections (Hero, Projects, TechSkills, etc.).
  - `ui/`: shadcn/ui base components.
- `prompt-hub/`: Organized Markdown prompt templates using the RCTF framework.

## 🧠 AI Development Conventions

1.  **Flows:** Define AI logic in `src/ai/flows/`. Use Zod schemas for input/output validation to ensure structured responses from the LLM.
2.  **Prompts:** Store complex prompt templates in `prompt-hub/` or define them in-code using `ai.definePrompt`. Prefer the RCTF framework for consistency.
3.  **Genkit UI:** Use `npm run genkit:dev` to iterate on prompts and flows in a sandboxed environment before integrating into the UI.
4.  **Environment Variables:** Requires `GOOGLE_GENAI_API_KEY` for AI features to function.

## 🎨 UI & Styling Standards

- Follow the established **shadcn/ui** patterns.
- Use `lucide-react` for iconography.
- Maintain the dark-themed, professional aesthetic defined in `src/app/globals.css`.
- Use the `cn()` utility from `src/lib/utils.ts` for conditional Tailwind classes.

## 🧪 Testing Strategy

- Currently, manual testing via Genkit UI and Next.js dev server.
- Add unit tests for utility functions in `src/lib/` and AI flows using Jest or Vitest as the project matures (TODO).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PanupongDeve)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/PanupongDeve)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
