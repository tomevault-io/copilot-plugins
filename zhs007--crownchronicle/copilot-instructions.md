## crownchronicle

> This document contains essential guidelines for the Gemini agent to ensure safe, efficient, and consistent work on this project.

# Gemini Agent Instructions for the Crown Chronicle Project

This document contains essential guidelines for the Gemini agent to ensure safe, efficient, and consistent work on this project.

## 1. Project Overview

*   **`crownchronicle`**: A monorepo for a game content creation suite.
*   **`editor`**: A Next.js application within the workspace that serves as a GUI for creating game content (characters, events) using an AI-driven, conversational workflow.
*   **`core`**: The core game logic library.
*   **`gameconfig`**: The YAML-based game data source of truth.

## 2. Critical Commands

This project is an **npm workspace**. All commands for a specific workspace **must** be run from the **project root** using the `--workspace` flag.

### Building the Editor

To build the `editor` application, **always** use the following command. It includes the necessary `NODE_ENV` override to prevent critical build failures:

```bash
NODE_ENV=production npm run build --workspace=editor
```

### Running the Editor in Dev Mode

```bash
npm run dev --workspace=editor
```

## 3. Environment & Configuration

*   **`NODE_ENV`**: The Next.js build process is extremely sensitive to the `NODE_ENV` environment variable. It **must** be set to `production` for builds. Local environment configurations should not override this for build commands.
*   **Configuration Files**: The `editor` project uses `next.config.ts`. Any stray `next.config.js` files are likely erroneous and should be investigated or removed.

## 4. Core Architectural Patterns

*   **AI-Driven Workflow**: The central feature of the `editor` is an AI-driven conversational interface. The AI's role is to be a "knowledgeable editor," guiding the user through a multi-step process to create game data based on real Chinese history. The AI, not the backend, drives the workflow logic. This is managed via a `WorkflowContext` object.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zhs007)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zhs007)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
