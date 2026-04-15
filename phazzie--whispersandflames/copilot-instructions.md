## whispersandflames

> This document provides instructions for Gemini, the AI coding assistant, on how to contribute to the "Whispers and Flames" project.

# Gemini Coding Assistant Instructions for "Whispers and Flames"

This document provides instructions for Gemini, the AI coding assistant, on how to contribute to the "Whispers and Flames" project.

## 1. Project Overview

**Whispers and Flames** is a Next.js application designed to foster intimacy and connection through guided, AI-driven conversations. Users create or join private rooms, select conversation themes and an intensity level, and answer a series of questions. At the end, an AI-generated summary of their conversation is provided.

**Core Goal:** Assist in developing, refactoring, and improving the application while adhering to the established architecture and coding style.

## 2. Tech Stack

- **Framework**: Next.js 14 (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS with shadcn/ui components
- **Backend & Database**: Firebase (Firestore for real-time game state, Firebase Auth for users)
- **Generative AI**: Google's Gemini model via Genkit

**Strict Adherence:** Do not introduce new frameworks or libraries (e.g., Angular, Vue, other CSS frameworks) without explicit instruction. The current stack is definitive.

## 3. Key Project Files & Structure

- `src/app/page.tsx`: The home/landing page.
- `src/app/game/[roomCode]/page.tsx`: The main entry point for a game session. It manages game state and renders the current step.
- `src/app/game/[roomCode]/steps/*.tsx`: Components for each step of the game flow (Lobby, Categories, Spicy, Game, Summary).
- `src/lib/firebase.ts`: Firebase initialization.
- `src/lib/game-types.ts`: Contains all TypeScript types for the game logic (GameState, Player, etc.).
- `src/lib/constants.ts`: Defines game constants like categories and spicy levels.
- `src/ai/flows/*.ts`: Genkit flows for generating questions and summaries.
- `src/ai/genkit.ts`: Genkit initialization and configuration.
- `aiprompting.md`: The master prompt and persona definition for "Ember," the question-generation AI.
- `agents.md`: High-level overview of the AI agent personas used in the app.
- `package.json`: Project dependencies. Any new packages added here will be auto-installed.

## 4. DigitalOcean MCP Server

This project is configured to use the `@digitalocean/mcp` tool, which allows the AI assistant to manage DigitalOcean resources using natural language commands.

- **Configuration:** The configuration is stored in the `.mcprc.json` file in the project root.
- **Authentication:** It requires the `DIGITALOCEAN_API_TOKEN` environment variable to be set with a valid DigitalOcean API token.
- **Capabilities:** The MCP server is configured to manage the following services:
  - `apps`: Manage DigitalOcean App Platform applications.
  - `droplets`: Manage DigitalOcean Droplets (virtual machines).
  - `insights`: Monitor resources and endpoints.
  - `spaces`: Manage DigitalOcean Spaces object storage.
  - `databases`: Manage managed database clusters.
  - `marketplace`: Manage DigitalOcean Marketplace applications.

This allows the assistant to perform tasks such as deploying applications, managing servers, and monitoring services directly through natural language commands.

## 5. Coding & Development Guidelines

### General

- **Clarity & Conciseness:** Write clean, readable, and self-documenting code.
- **Component-Based:** Create small, reusable components where possible. Default to server components unless client-side interactivity (`'use client'`) is necessary.
- **TypeScript First:** Use TypeScript for all new code. Leverage the types defined in `src/lib/game-types.ts`.
- **No Comments in JSON:** Do not add comments to `package.json` or `tsconfig.json`.

### Next.js

- **App Router:** Use the `app/` directory structure.
- **Server Actions:** Use server actions (`'use server'`) for backend logic callable from client components, as seen in `src/app/game/actions.ts`.

### Firebase

- **Client-Side SDK:** All Firestore and Auth operations are performed on the client side (`'use client'`).
- **Real-time Updates:** Use `onSnapshot` for real-time game state synchronization. Ensure listeners are properly unsubscribed in `useEffect` cleanup functions to prevent memory leaks.
- **Data Modeling:** The game state is stored in a single Firestore document per game room, located in the `games` collection.
- **Security:** Security rules are managed outside of your direct changes but assume they exist. Write code that respects user ownership and authentication.

### Genkit (AI)

- **Flows:** AI logic is encapsulated in Genkit flows located in `src/ai/flows/`.
- **Schemas:** Use Zod (`z`) to define input and output schemas for all flows. This ensures type safety and predictable AI outputs.
- **Prompt Engineering:** The primary prompt for question generation is in `aiprompting.md`. Refer to this for AI behavior changes.

## 6. Responding to User Requests

- **Clarify Ambiguity:** If a user request is unclear, ask clarifying questions before writing code.
- **Explain Your Plan:** Before providing a code change, briefly explain what you are going to do and why.
- **Use the `<changes>` Block:** All file modifications MUST be enclosed in a `<changes>` XML block. Provide the full, final content of each file being modified. Do not use diffs or partial snippets.
- **Summarize After:** After providing the code, give a brief confirmation of what was accomplished.

Your primary goal is to be a collaborative and highly effective coding partner, helping the user build and refine their application efficiently and correctly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Phazzie)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Phazzie)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
