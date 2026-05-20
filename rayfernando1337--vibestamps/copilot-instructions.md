## project

> Project overview: Goals, tech stack (Next.js, shadcn/ui, Bun), design system details, and development guidelines

## Goal

This project aims to create a web application that takes SubRip Text (`.srt`) files as input, processes them using the Google Gemini language model via the Vercel AI SDK, and generates meaningful timestamps or summaries based on the content.

## Tech Stack

- **Framework:** Next.js (App Router)
- **Language:** TypeScript
- **Styling:** Tailwind CSS v4
- **UI Components:** shadcn/ui
- **AI Integration:** Vercel AI SDK
- **LLM:** Google Gemini
- **Package Manager:** Bun

## Design System

We are using **shadcn/ui** for our component library. Components are added individually as needed.

- **Documentation:** [https://ui.shadcn.com/docs/components/](mdc:https:/ui.shadcn.com/docs/components)
- **Adding Components:** Use the CLI with Bun:
  ```bash
  bunx --bun shadcn@latest add <component-name>
  ```

## Development Rules

1.  **Package Management:** Always use `bun` for installing, removing, or managing dependencies (`bun add`, `bun install`, `bun remove`, etc.).
2.  **UI Components:** Prefer components from `shadcn/ui` where possible. Install them using the command above.
3.  **Environment Variables:** Store sensitive information like API keys in environment variables (`.env.local`) and do not commit them to version control.
4.  **Code Style:** Follow standard TypeScript and React best practices. Ensure code is formatted (consider adding a formatter like Prettier later).
5.  **Tailwind CSS v4:** Use Tailwind CSS v4 for styling. Refer to the [Tailwind CSS documentation](https://tailwindcss.com/docs) for more information.

---
> Source: [RayFernando1337/vibestamps](https://github.com/RayFernando1337/vibestamps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
