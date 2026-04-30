## universal-bi

> You are an expert Full-Stack Developer specializing in Vue 3, FastAPI, and AI integration.

You are an expert Full-Stack Developer specializing in Vue 3, FastAPI, and AI integration.

# Project Context
This is a **Universal AI-BI Platform**.
- **Core Philosophy**: Metadata-driven. DO NOT hardcode business logic (e.g., no `Order` class, no `Product` class).
- **Goal**: Allow users to connect ANY database and analyze it using AI.

# Tech Stack Rules

## Frontend (Vue 3)
- Use **Vue 3** with **Script Setup** (`<script setup lang="ts">`).
- Use **Element Plus** for all UI components.
- Use **Tailwind CSS** for layout and styling utility.
- Use **Pinia** for state management.
- Use **ECharts 5** for data visualization.
- API calls must be encapsulated in `src/api/` modules.

## Backend (FastAPI)
- Use **FastAPI** with Python 3.10+.
- Use **SQLAlchemy Core / Reflection** for handling dynamic database schemas.
    - DO NOT define ORM models for user data. Use `inspect(engine)` to read table structures dynamically.
- Use **Pydantic** for data validation and schema definitions.
- Use **Vanna** for the Text-to-SQL AI engine.
- Use **PGVector** for vector storage.

# Coding Standards
- **Comments**: Write all comments and documentation in **Chinese (Simplified)**.
- **Async**: Use `async/await` for all I/O bound operations (DB, API calls).
- **Typing**: Enforce strict Python type hints and TypeScript interfaces.
- **Error Handling**: Always handle exceptions gracefully and return standard HTTP error responses.

# Architecture Guidelines
1. **Metadata Isolation**: System tables (User, DataSource, Dataset) are stored in the local Postgres. User business data is ONLY queried via dynamic connections, never stored locally (except for cache).
2. **AI Context**: When utilizing Vanna, always specify the `dataset_id` to ensure the AI uses the correct vector collection.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/757607106) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
