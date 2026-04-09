## portfolio-website

> - Name: Interactive Developer Portfolio

# Project Context
- Name: Interactive Developer Portfolio
- Stack: Next.js (App Router), TypeScript, React, Tailwind CSS, Three.js
- Architecture: Component-driven UI with a strict separation of concerns. Data is statically sourced from data/resume.json to populate sections like Education, Experience, Projects, and Hackathons. UI elements are built as functional components in the app/components/ directory. Next.js Server Components should be used by default, isolating Three.js WebGL rendering exclusively to Client Components

# Agent Constraints
- Test Coverage: All data-parsing utilities and core UI components must include Jest or Vitest unit tests.
- Code Style: Follow best practices for React development and future-proofing for later development. Use functional React components with Hooks. Enforce strict TypeScript typing for all component props, ensuring interfaces perfectly mirror the structured resume data. Prefer absolute imports (e.g., @/components/ExperienceTimeline) and avoid inline CSS in favor of Tailwind utility classes

## Extended Capabilities
ALWAYS read `.vibe/mcp-triggers.md` before executing complex tasks or using external tools.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AndrewChang-cpu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AndrewChang-cpu)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
