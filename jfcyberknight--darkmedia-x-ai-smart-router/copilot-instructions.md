## darkmedia-x-ai-smart-router

> **AI Smart Router** is a high-performance, resilient API gateway deployed on **Vercel**. It provides a single point of entry for multiple AI providers (Gemini, Groq, DeepSeek, Mistral, Ollama, etc.), implementing **automatic load balancing** (via random shuffling) and **transparent fallback** in case of quota limits (429) or service outages (500/503).

# ♊ GEMINI.md — Darkmedia-X AI Smart Router

## 📌 Project Overview
**AI Smart Router** is a high-performance, resilient API gateway deployed on **Vercel**. It provides a single point of entry for multiple AI providers (Gemini, Groq, DeepSeek, Mistral, Ollama, etc.), implementing **automatic load balancing** (via random shuffling) and **transparent fallback** in case of quota limits (429) or service outages (500/503).

- **Core Mission**: Ensure 100% availability for AI-powered applications by routing requests through a pool of available providers.
- **Main Features**:
  - `POST /api/chat`: AI conversation with intelligent routing.
  - `POST /api/image`: Image generation (DALL-E 3 via OpenRouter/Fal.ai).
  - `POST /api/normalize`: Text-to-JSON extraction using high-efficiency models.
  - **Dynamic Fallback**: If a provider fails or hits a rate limit, the router immediately tries the next available one.
  - **Environment Sync**: Automated scripts to sync `.env` variables with Vercel and local templates.

---

## 🛠️ Building and Running

### Prerequisites
- **Node.js**: >= 18
- **Package Manager**: `pnpm` (MANDATORY per `.instructions-ai/00-system-directives.md`)
- **CLI**: `vercel` CLI installed globally (`npm install -g vercel`)

### Key Commands
| Action | Command |
|--------|---------|
| **Install** | `pnpm install` |
| **Development** | `pnpm run dev` (Starts `vercel dev`) |
| **Deploy (Prod)** | `pnpm run deploy` |
| **Sync Env Vars** | `pnpm run env:sync` (Updates `.env.example` and Vercel) |
| **Generate Secret** | `python ai-smart-router/scripts/generate-api-secret.py` |
| **Run Tests** | `pnpm test` (Exécute les tests unitaires et d'intégration) |
| **Test Deployed API** | `pnpm run test:deployed` (Vérifie la santé de l'API live sur Vercel) |
| **Test Providers** | `pnpm run test:providers` (Verifies API keys and routing) |
| **Test Live API** | `pnpm run test:api:prod` |
| **RAG Indexing** | `pnpm run rag:index` (Indexe .instructions-ai dans Qdrant) |
| **RAG Push Workflow** | `pnpm run rag:push` (Git push + Indexation automatique) |

---

## 📐 Architecture & Structure

- **`ai-smart-router/functions/`**: Vercel Serverless Functions (Entry points).
- **`ai-smart-router/lib/`**: Business logic.
  - `router.js`: The heart of the system (shuffling and fallback logic).
  - `auth.js`: Security layer using `API_SECRET`.
  - `providers/`: Modular adapters for each AI service (Gemini, Groq, etc.).
- **`ai-smart-router/scripts/`**: DevOps utilities for environment management and pre-deployment checks.
- **`.instructions-ai/`**: Source of truth for all engineering and behavioral standards.

---

## 📜 Development Conventions

### 1. Package Management
- **pnpm ONLY**: Usage of `npm` or `yarn` is strictly prohibited. All dependencies must be managed via `pnpm`.

### 2. Code Quality & Style
- **0-Warning Tolerance**: Code must pass all linting and type-checking without warnings.
- **Standard**: Follow `03-javascript-typescript.md` and `09-documentation-standards.md`.
- **Formatting**: Use Prettier (`pnpm run format`).
- **Naming**: Use clear, descriptive names. French is the primary language for documentation and comments unless otherwise specified.

### 3. Git Workflow
- **Conventional Commits**: Messages must follow the `feat:`, `fix:`, `chore:`, etc., convention.
- **Pre-push Hook**: Automatically runs `env:sync` to ensure Vercel and the local environment are aligned.

### 4. Security
- **API_SECRET**: All requests to the API must be authenticated via an `Authorization: Bearer <API_SECRET>` header.
- **Secret Protection**: Never commit `.env` or hardcode API keys. Use the provided sync scripts.

### 5. Testing
- **Native Test Runner**: Use `node --test` for fast, dependency-free testing.
- **Mandatory Validation**: New features or providers **MUST** include tests in `ai-smart-router/tests/` and a validation script in `ai-smart-router/scripts/`.

---

## 🚀 Adding a New Provider
1. Create `ai-smart-router/lib/providers/new-provider.js` exporting a `generate` function.
2. Register the provider in `ai-smart-router/lib/router.js` within the `PROVIDERS` array.
3. Update `.env.example` with the required environment variables.
4. Run `pnpm run test:providers` to verify integration.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jfcyberknight)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jfcyberknight)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
