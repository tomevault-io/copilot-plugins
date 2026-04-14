## alpinebot

> AlpineBot is an AI-powered chatbot dedicated to providing accurate information about Switzerland using public data sources. It features a minimalist design and a Retrieval-Augmented Generation (RAG) architecture.

# AlpineBot Context

## Project Overview
AlpineBot is an AI-powered chatbot dedicated to providing accurate information about Switzerland using public data sources. It features a minimalist design and a Retrieval-Augmented Generation (RAG) architecture.

**Tech Stack:**
- **Frontend:** React 18 (deployed to Azure Web App)
- **Backend:** Azure Functions (Python 3.12)
- **AI/ML:** Azure OpenAI (GPT-4)
- **Database:** Azure Database for PostgreSQL & Azure Redis Cache
- **Infrastructure:** Terraform (IaC)
- **CI/CD:** GitHub Actions

## Directory Structure
- **`frontend/app/`**: The React Single Page Application (SPA).
  - `src/`: Source code (components, pages, styles).
  - `package.json`: Dependencies and scripts.
- **`backend/`**: Azure Functions Python app.
  - `function_app.py`: Main entry point for functions.
  - `requirements.txt`: Python dependencies.
- **`infra/`**: Terraform configuration for all Azure resources.
  - `main.tf`: Main infrastructure definition.
- **`modules/`**: Reusable Terraform modules (App Service, Cognitive Services, DBs, etc.).
- **`.github/workflows/`**: CI/CD definitions.

## Development Guidelines

### Prerequisites
- **Node.js:** v18+ (for Frontend)
- **Python:** v3.12 (for Backend)
- **Azure Functions Core Tools:** v4.x (for local backend debugging)
- **Git:** For version control

### 1. Frontend (React)
Located in `frontend/app/`.

- **Install Dependencies:**
  ```bash
  cd frontend/app
  npm install
  ```
- **Run Locally:**
  ```bash
  npm start
  ```
  *Note: Requires `REACT_APP_FUNCTION_APP_URL` in `.env.local` to point to a running backend (e.g., `http://localhost:7071` or a dev Azure URL).*
- **Build:**
  ```bash
  npm run build
  ```
- **Test:**
  ```bash
  npm test
  ```

### 2. Backend (Azure Functions)
Located in `backend/`.

- **Install Dependencies:**
  ```bash
  cd backend
  pip install -r requirements.txt
  ```
- **Run Locally:**
  ```bash
  func start
  ```
  *Note: Requires `local.settings.json` with `AZURE_OPENAI_API_KEY` and `AZURE_OPENAI_ENDPOINT`.*

### 3. Infrastructure (Terraform)
Located in `infra/`.

- **IMPORTANT:** Infrastructure is managed **exclusively** via GitHub Actions. Do not run `terraform apply` locally unless strictly necessary for debugging read-only plans.
- Define resources in `infra/main.tf` and `modules/`.

## Deployment
Deployment is automated via GitHub Actions:
- **Frontend:** `deploy-app.yaml` builds and deploys the React app to Azure Web App.
- **Backend:** `deploy-function.yaml` (implied) packages and deploys Python functions.
- **Infrastructure:** `deploy-infra.yaml` applies Terraform changes.

## Key Configuration Files
- **`docs/specifications.md`**: Detailed functional and non-functional requirements.
- **`backend/function_app.py`**: Core logic for the Chatbot API (`/api/chat`).
- **`frontend/app/package.json`**: Frontend scripts and dependencies.
- **`infra/main.tf`**: Entry point for infrastructure definitions.

## Code Style & Conventions
- **Design:** "Swiss Minimalist" - clean, white/light gray, elegant typography (Space Grotesk).
- **Auth:** Google OAuth via Azure App Service "Easy Auth" (handled at infrastructure level).
- **Testing:** TDD is encouraged. Unit tests for React components and Python functions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fpittelo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fpittelo)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
