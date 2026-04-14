## founderlings

> This project is the landing page for the **Founderlings** program, the first 1,000 founding members of the **Beginner Investor Hub**. It is a high-performance, visually rich Next.js application designed with a custom "mechanica" aesthetic.

# Founderlings - Gemini CLI Context

This project is the landing page for the **Founderlings** program, the first 1,000 founding members of the **Beginner Investor Hub**. It is a high-performance, visually rich Next.js application designed with a custom "mechanica" aesthetic.

## Project Overview

- **Purpose:** Capture early adopters (first 1,000 "Founderlings") for pre-launch access to the **Beginner Investor Hub**.
- **Context:** This landing page is part of a larger ecosystem consisting of **14+ microservices** organized around a financial education and investment platform with AI-powered personalization.
- **Value Proposition:** Founderlings help **train the LLM** and receive early access plus **lifetime free premium for a one-time fee of $29.99**.
- **Framework:** Next.js 15 (App Router)
- **Language:** TypeScript
- **Styling:** Tailwind CSS + **mechanica** design system (custom industrial/mechanical aesthetic).
- **Backend/API:** Next.js Route Handlers (App Router API routes).
- **Integrations:**
    - **Google Sheets:** Signups are sent to a Google Apps Script URL (defined in `GOOGLE_SCRIPT_URL`).
    - **Vercel Analytics:** Enabled via `@vercel/analytics`.

## Tech Stack & Architecture

- **Frontend:** React 18, Next.js 15, Tailwind CSS 3.
- **State Management:** React `useState` and `useEffect` for local component state.
- **Data Fetching:** Standard `fetch` API for internal route calls (`/api/count`, `/api/signup`).
- **Design System:** The `mechanica` system is defined in `app/globals.css` using CSS variables for colors (moonlight blue, brushed copper, birch wood), typography, and mechanical-inspired spacing/animations.

## Building and Running

### Development
```bash
npm install
npm run dev
```
Open [http://localhost:3000](http://localhost:3000)

### Production Build
```bash
npm run build
npm run start
```

### Linting
```bash
npm run lint
```

## Project Structure

- `app/`: Next.js App Router directory.
    - `api/`: Route handlers for signups and count tracking.
    - `globals.css`: Core design system definitions (mechanica).
    - `page.tsx`: The main landing page entry point.
- `components/`: Modular UI components (Header, Benefits, CTA, etc.).
- `types/`: TypeScript definitions (global and image types).
- `public/`: Static assets.

## Development Conventions

- **Component Design:** Prefer functional components with hooks. Components should be modular and kept in the `components/` directory.
- **Styling:** Use Tailwind CSS utility classes. For "mechanica" specific styles, refer to the custom classes and variables in `app/globals.css` (e.g., `.mechanica-button-mechanical`, `.mechanica-card-wood`).
- **API Routes:** Follow the App Router convention using `route.ts` files. Ensure proper error handling and type safety (e.g., using `unknown` instead of `any` in catch blocks).
- **Environment Variables:**
    - `GOOGLE_SCRIPT_URL`: Required for the signup feature to work.

## Key Files to Reference

- `app/globals.css`: Contains the entire `mechanica` design system specification.
- `app/page.tsx`: The primary orchestrator for the landing page UI and state.
- `app/api/signup/route.ts`: Core integration logic for the member signup process.
- `tailwind.config.ts`: Tailwind configuration (standard, with minimal extensions).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Beginnerinvestorhub)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Beginnerinvestorhub)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
