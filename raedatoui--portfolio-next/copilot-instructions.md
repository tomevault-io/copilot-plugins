## portfolio-next

> **CRITICAL: You are STRICTLY FORBIDDEN from executing ANY shell commands.**

# Gemini Context & Instructions

## 🛑 STRICT OPERATING RULES (READ FIRST) 🛑

**CRITICAL: You are STRICTLY FORBIDDEN from executing ANY shell commands.**

*   **ABSOLUTELY NO SHELL COMMANDS:** Do not run `npm`, `node`, `ls`, `mkdir`, `touch`, or any other shell command.
*   **ABSOLUTELY NO GIT COMMANDS:** Do not run `git status`, `git add`, `git commit`, `git push`, or `git diff`. The user handles ALL version control.
*   **ABSOLUTELY NO LINTING/BUILD COMMANDS:** Do not run `biome`, `tsc`, or `next build`.
*   **FILE OPERATIONS ONLY:** You may ONLY read (`read_file`) and write (`write_file`, `replace`) files.
*   **VIOLATION PROTOCOL:** If a plan involves running a command, **STOP**. Rewrite the plan to only involve file edits, or ask the user to run the command.

This file provides high-level context and instructions for AI agents (Gemini) working on this project.

## Project Overview

This is a personal portfolio website for Raed Atoui, built with the latest web technologies. It features a unique interactive audio-visual experience using `Tone.js` for sound synthesis and HTML5 Canvas for real-time pixelation effects.

**Tech Stack:**
*   **Framework:** Next.js 16 (App Router)
*   **UI Library:** React 19
*   **Styling:** Tailwind CSS v4
*   **Language:** TypeScript 5.9 (Strict Mode)
*   **Audio:** Tone.js
*   **Validation:** Zod (for content JSON)
*   **Tooling:** Biome (Linting & Formatting)

## Architecture

### 1. Content-Driven Design
The site's content is decoupled from the code, stored in JSON files within `src/content/`.
*   **Source of Truth:** `src/lib/LoadContent.ts` uses **Zod** to validate these JSON files at runtime and infer TypeScript types.
*   **Legacy:** `src/types/content.ts` contains commented-out legacy interfaces. Do not use them. Rely on the inferred types from `src/lib/LoadContent.ts`.

### 2. Interactive Audio-Visual System
The core "wow" factor of the site lies in the interaction between mouse movement, canvas rendering, and generated audio.
*   **`src/lib/Nervous.ts`:** Handles audio synthesis using `Tone.PolySynth`. It generates random walk patterns (Box-Muller transform) to map notes to pixelation levels.
*   **`src/components/Projects/ProjectCard.tsx`:** A Client Component that renders project thumbnails on a Canvas. It handles the pixelation effect using the Canvas 2D API (disabling image smoothing) and synchronizes with the `Nervous` audio system on hover.
*   **Experimental WebGL:** The project contains experimental WebGL components for advanced visual effects.
    *   **`src/components/ShaderBackground/`**: A background component using custom shaders (FBM noise, domain warping) for atmospheric effects.
    *   **`src/components/Projects/ProjectCard2.tsx`**: An alternative WebGL-based implementation of the project card pixelation using shaders (see `src/components/Projects/shader.ts`).

### 3. Project Structure
*   `src/app/`: Next.js App Router pages and layouts.
*   `src/components/`: React components, organized by feature (e.g., `Header`, `Bio`, `Projects`, `ShaderBackground`).
*   `src/content/`: JSON data files.
*   `src/lib/`: Utility logic (`LoadContent.ts`, `Nervous.ts`).

## conventions & Best Practices

*   **Strict TypeScript:** No `any`. Ensure all types are properly defined or inferred.
*   **Imports:** Use the `@/` alias to refer to the `src` directory (e.g., `import { ... } from '@/components/...'`).
*   **Styling:** Use Tailwind CSS utility classes.
*   **Components:**
    *   Default to Server Components where possible.
    *   Use `'use client'` only for components requiring interactivity (state, hooks, browser APIs like `window` or `AudioContext`).
*   **Audio Safety:** Audio context requires user interaction to start. Handle this gracefully. The `Nervous` class manages context resumption.

## Important Files

*   `biome.json`: Configuration for linting and formatting.
*   `src/lib/LoadContent.ts`: **CRITICAL**. Defines the schema for all site content. Modify this if adding new fields to JSON files.
*   `src/lib/Nervous.ts`: Logic for the generative audio system.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/raedatoui)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/raedatoui)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
