## adiagarwalrock

> This repository is a monorepo containing multiple iterations and versions of the user's personal portfolio website. It showcases different technologies and design paradigms, ranging from a retro desktop simulation to a modern AI-integrated interface and a legacy Python backend.

# Project Context: adiagarwalrock

## Overview
This repository is a monorepo containing multiple iterations and versions of the user's personal portfolio website. It showcases different technologies and design paradigms, ranging from a retro desktop simulation to a modern AI-integrated interface and a legacy Python backend.

## Sub-Projects

The repository is divided into three main applications:

### 1. Desktop Portfolio (`desktop-portfolio/`)
A **Vue 3 + Vite** application designed to simulate a desktop operating system environment (resembling Windows 98/XP).

*   **Technology Stack:**
    *   **Framework:** Vue 3, Vite, TypeScript
    *   **State Management:** Pinia
    *   **Styling:** `98.css`, `xp.css`, `winbox`
    *   **Key Libraries:** `clippyjs`, `turndown`, `vue-command`
*   **Structure:** Follows a standard Vite project structure with source code located in `src/`.
*   **Development Commands:**
    *   Install dependencies: `npm install`
    *   Start development server: `npm run dev`
    *   Build for production: `npm run build`
    *   Type check: `npm run type-check`

### 2. Modern Portfolio (`portfolio/`)
A newer **Vue 3 + Vite** application, likely an "AI Studio" style app, featuring integration with Google's Gemini AI.

*   **Technology Stack:**
    *   **Framework:** Vue 3, Vite, TypeScript
    *   **AI Integration:** `@google/genai`
    *   **Icons:** `lucide-vue-next`
*   **Structure:** Uses a **flat structure** where source files (`App.vue`, `main.ts`, `components/`) are located directly in the project root (`portfolio/`), rather than a `src/` directory.
*   **Environment Variables:**
    *   Requires a `.env.local` file with `GEMINI_API_KEY`.
*   **Development Commands:**
    *   Install dependencies: `npm install`
    *   Start development server: `npm run dev`
    *   Build for production: `npm run build`

### 3. Legacy Python App (`legacy_python/`)
A **Python Flask** application, representing an older version of the portfolio or a backend service.

*   **Technology Stack:**
    *   **Framework:** Flask, Gunicorn
    *   **Templating:** Jinja2
    *   **Deployment:** Geared towards Heroku-like environments (presence of `Procfile`, `runtime.txt`).
*   **Structure:**
    *   Entry point: `app.py`
    *   App logic: `flask_app/` package
    *   Static assets: `static/`
    *   Templates: `templates/`
*   **Development Commands:**
    *   Install dependencies: `pip install -r requirements.txt`
    *   Run locally: `python app.py`
    *   Run with Gunicorn: `gunicorn app:app`
*   **Configuration:**
    *   Uses `.env` file for secrets (loaded via `python-dotenv`).
    *   Requires `APP_SECRET` and `ADMIN_URL` environment variables.

## Global Conventions

*   **Monorepo:** The root directory acts as a container. There is no global orchestration tool (like Lerna or Nx) visible; projects are independent.
*   **Version Control:** Git is used. Standard `.gitignore` exists at the root and within sub-projects.
*   **Personal Profile:** The root `README.md` is a GitHub Profile README and does not document the code. Refer to sub-project READMEs for specific instructions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adiagarwalrock)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/adiagarwalrock)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
