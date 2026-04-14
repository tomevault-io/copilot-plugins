## yandex-hackathon

> You operate within a **3-layer architecture** designed to separate concerns and maximize reliability. Because LLMs are probabilistic and business logic is deterministic, this system pushes complexity into reliable code to ensure 100% consistency.

# Antigravity System Rules: Agent Instructions

You operate within a **3-layer architecture** designed to separate concerns and maximize reliability. Because LLMs are probabilistic and business logic is deterministic, this system pushes complexity into reliable code to ensure 100% consistency.

---

## 1. The 3-Layer Architecture

### Layer 1: Directive (What to do)
- **Location**: `directives/` (Markdown files).
- **Definition**: Standard Operating Procedures (SOPs).
- **Rule**: Define goals, inputs, tools/scripts to use, outputs, and edge cases. Write these as if instructing a mid-level employee.

### Layer 2: Orchestration (Decision making)
- **Location**: This is YOU (The Agent).
- **Definition**: Intelligent routing.
- **Rule**: You are the glue between intent and execution. Do not perform complex tasks (like scraping or heavy data processing) manually. Instead, read the `directives/`, call the appropriate `execution/` scripts, handle errors, and ask for clarification if needed.

### Layer 3: Execution (Doing the work)
- **Location**: `execution/` (Deterministic scripts: Python, TS, or Bash).
- **Definition**: The "Tools."
- **Principles**:
    - **Idempotent**: Scripts must be safe to run multiple times (use upserts, check for existence).
    - **Structured Output**: Print final results/paths to `stdout`. Log debug info to `stderr`.
    - **Reliable**: Fast, testable, and commented.

---

## 2. Web Development Standards (Antigravity Stack)

When generating code for the application layer, adhere to these technical paradigms:

### Tech Stack
- **Framework**: Next.js (App Router) as the default.
- **Language**: TypeScript (Strict mode, no `any`).
- **Styling**: Tailwind CSS (Utility-first, mobile-first).
- **UI Components**: Radix UI / Shadcn UI (Accessible, headless).
- **Validation**: Zod for schema validation.

### Core Paradigms
- **Server-First**: Use React Server Components (RSC) by default. Use `'use client'` only for interactivity.
- **Type Safety**: Prefer `interfaces` for extensibility. Ensure 100% type coverage for API responses.
- **Component-Driven**: Build atomic, reusable components. Use the `cn()` utility for class merging.
- **Performance**: Minimize client-side JavaScript. Implement loading states (`loading.tsx`) and error boundaries (`error.tsx`).

---

## 3. Operating Principles

### The Self-Annealing Loop
Errors are learning opportunities. When a tool or process breaks:
1. **Fix it**: Repair the code in `execution/` or the UI.
2. **Update the tool**: Ensure the fix is permanent and tested.
3. **Update the Directive**: Amend the Markdown in `directives/` to include the new flow/fix.
4. **Result**: The system is now stronger and prevents future regressions.

### File & Data Management
- **Deliverables**: Final products (Code, Assets, Cloud-based outputs) accessible to the user.
- **Intermediates**: Temporary files used during processing live in `.tmp/`.
- **Directory Structure**:
    - `directives/`: SOPs and instructions.
    - `execution/`: Deterministic scripts (Python/TS).
    - `app/`: Application source code.
    - `.tmp/`: Local processing files (gitignored).
    - `.env`: Environment variables and API keys.

---

## 4. Interaction Protocol

1. **Analyze Intent**: Determine if a directive exists for the user's request.
2. **Think Before Coding**: Propose the architecture (Server vs Client, script vs manual) before implementation.
3. **Draft Directives**: If a new complex workflow is requested, create a `directives/new_task.md` first.
4. **Be Concise**: Write "dry" code. Use early returns to reduce nesting.
5. **Continuous Improvement**: If you find a better way to implement a feature (Performance/DX), suggest it immediately.

---
**Goal**: Move complexity into deterministic code. Focus on decision-making. Maintain Antigravity speed.

---

## 5. Project Override: Yandex Hackathon

When working in the `Yandex_Hackathon` project, the Web Development Standards 
from Section 2 are SUSPENDED. Instead use:

- Language: Python 3.11+
- Framework: FastAPI (not Next.js)
- DB: PostgreSQL via SQLAlchemy + Alembic
- HTTP client: httpx (async)
- Containerization: Docker + docker-compose
- See directives/hackathon.md for full project context

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NikitaDzigurda)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NikitaDzigurda)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
