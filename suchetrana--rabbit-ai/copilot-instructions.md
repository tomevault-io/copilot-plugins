## rabbit-ai

> > This file persists project context across Claude sessions.

# CLAUDE.md — Rabbitt Sales Insight Automator

> This file persists project context across Claude sessions.
> Read this file at the start of every session before making any changes.

---

## 📦 Project Overview

**Project Name:** Rabbitt Sales Insight Automator  
**Role:** AI Cloud DevOps Engineer  
**Stack:** React + Vite (Frontend) · Node.js + Express (Backend)  
**Goal:** Upload CSV/XLSX sales data → AI generates summary → Email delivered to recipient

---

## 🗂️ Project Structure

```
rabbitt-sales-insight/
├── frontend/                    # React + Vite → deploy on Vercel
│   ├── src/
│   │   ├── components/          # UploadForm, FileDropzone, StatusBanner
│   │   ├── hooks/               # useUpload.js
│   │   ├── services/            # api.js (Axios instance)
│   │   ├── App.jsx
│   │   └── main.jsx
│   ├── Dockerfile
│   └── .env.example
│
├── backend/                     # Node.js + Express → deploy on Render
│   ├── src/
│   │   ├── config/              # env.js (dotenv + Zod validation)
│   │   ├── middlewares/         # rateLimiter, helmet, validate, errorHandler
│   │   ├── routes/              # upload.route.js
│   │   ├── controllers/         # upload.controller.js
│   │   ├── services/
│   │   │   ├── parser.service.js    # CSV/XLSX → JSON
│   │   │   ├── ai.service.js        # Groq (Llama 3) integration
│   │   │   └── mailer.service.js    # Resend email delivery
│   │   ├── docs/                # swagger.js (OpenAPI spec)
│   │   └── app.js
│   ├── server.js
│   ├── Dockerfile
│   └── .env.example
│
├── docker-compose.yml
├── .github/workflows/ci.yml     # PR → lint + build
└── README.md
```

---

## 🎨 Google Stitch MCP Integration

**What is Google Stitch?**  
Google Stitch (`https://stitch.withgoogle.com`) is an AI-powered UI design tool that generates high-quality HTML/CSS UI screens from text prompts. The MCP server integration allows Claude and other AI agents to pull design context directly into the development workflow.

**Why use it here?**  
Use Stitch MCP to generate and iterate on the frontend UI (upload form, status states, email input) without manually writing all the CSS from scratch. Claude can fetch design DNA (colors, fonts, layout structure) from a Stitch screen and apply it consistently across components.

---

### ⚙️ Stitch MCP Setup (One-Time)

#### Option A — API Key (Recommended, simplest)

1. Go to [https://stitch.withgoogle.com](https://stitch.withgoogle.com)
2. Click your **profile icon** (top-right) → **Stitch Settings**
3. Go to **API Keys** section → click **Create Key**
4. Copy the generated API key

Add to your MCP client config (`claude_desktop_config.json` on Windows at `%APPDATA%\Claude\`):

```json
{
  "mcpServers": {
    "stitch": {
      "command": "npx",
      "args": ["-y", "stitch-mcp"],
      "env": {
        "STITCH_API_KEY": "your_stitch_api_key_here"
      }
    }
  }
}
```

Restart Claude Desktop after saving.

#### Option B — Google Cloud ADC (gcloud CLI)

```bash
# 1. Login
gcloud auth login

# 2. Set your project
gcloud config set project YOUR_PROJECT_ID
gcloud auth application-default set-quota-project YOUR_PROJECT_ID

# 3. Enable Stitch API
gcloud beta services mcp enable stitch.googleapis.com --project=YOUR_PROJECT_ID
```

MCP config (no API key needed):

```json
{
  "mcpServers": {
    "stitch": {
      "command": "npx",
      "args": ["-y", "stitch-mcp"],
      "env": {
        "GOOGLE_CLOUD_PROJECT": "YOUR_PROJECT_ID"
      }
    }
  }
}
```

> **WSL2 Note:** Browser-based OAuth may not auto-open in WSL2. Copy the OAuth URL from the terminal and paste it manually into your Windows browser.

---

### 🛠️ Available Stitch MCP Tools

Once the MCP server is connected, Claude has access to these tools:

| Tool | Description |
|------|-------------|
| `list_projects` | View all your Stitch design projects |
| `get_project` | Get details of a specific project |
| `list_screens` | List all screens in a project |
| `get_screen_code` | Download HTML/CSS for a specific screen |
| `get_screen_image` | Get screenshot of a screen as base64 |
| `extract_design_context` | Extract Design DNA — colors, fonts, layout |
| `generate_screen` | Generate a new screen from a text prompt |
| `build_site` | Map multiple screens to routes and build a site |

---

### 🎯 How to Use Stitch MCP in This Project

#### Generate the Upload UI screen
```
Use Stitch MCP to generate a UI screen for:
"A sales data upload tool. Single page with a file dropzone 
that accepts CSV and XLSX, an email input field, and a submit 
button. Professional dark theme. Show loading and success states."
```

#### Extract Design DNA from an existing screen
```
Use Stitch MCP extract_design_context on screen <screen-id> 
from project <project-id> and apply the same color palette, 
fonts, and layout structure to the React UploadForm component.
```

#### Fetch screen HTML and convert to React
```
Use Stitch MCP get_screen_code for screen <screen-id>, 
then convert the HTML/CSS into a React component using 
Tailwind CSS classes. Place it in frontend/src/components/UploadForm.jsx
```

---

## 🔒 Security Implementation

| Layer | Method |
|-------|--------|
| HTTP Headers | `helmet` middleware — 14 security headers |
| Rate Limiting | `express-rate-limit` — 10 req/15min per IP on `/api/upload` |
| File Validation | Multer: MIME type whitelist (`text/csv`, `application/vnd.openxmlformats...`), 10MB max |
| Input Validation | Zod schema — email format enforced before processing |
| CORS | Whitelist only Vercel frontend URL in production |
| Secrets | All keys via `.env`, validated on startup with Zod |

---

## 🌍 Environment Variables

### Backend (`backend/.env.example`)
```env
PORT=5000
NODE_ENV=development

# AI - Groq
GROQ_API_KEY=your_groq_api_key_here

# Email - Resend
RESEND_API_KEY=your_resend_api_key_here
FROM_EMAIL=noreply@yourdomain.com

# Security
CORS_ORIGIN=http://localhost:5173
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX=10
```

### Frontend (`frontend/.env.example`)
```env
VITE_API_BASE_URL=http://localhost:5000
```

### Stitch MCP (not committed, set in MCP config)
```env
STITCH_API_KEY=your_stitch_api_key_here
# OR
GOOGLE_CLOUD_PROJECT=your_gcp_project_id
```

---

## 🐳 Running Locally with Docker

```bash
# Clone the repo
git clone https://github.com/your-username/rabbitt-sales-insight.git
cd rabbitt-sales-insight

# Copy env files
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env
# Fill in your API keys in both .env files

# Start everything
docker-compose up --build
```

- Frontend: http://localhost:5173  
- Backend API: http://localhost:5000  
- Swagger Docs: http://localhost:5000/api-docs  

---

## 🚀 Deployment

| Service | Platform | Trigger |
|---------|----------|---------|
| Frontend | Vercel | Push to `main` |
| Backend | Render | Push to `main` |
| CI/CD | GitHub Actions | Pull Request to `main` |

---

## 📋 CI/CD Pipeline (`.github/workflows/ci.yml`)

Triggers on every PR to `main`:
1. Lint frontend (ESLint)
2. Lint backend (ESLint)
3. Build frontend (`vite build`)
4. Docker build validation (both services)

---

## 🧠 AI Flow (End-to-End)

```
User uploads CSV/XLSX + enters email
        ↓
Multer validates file (type + size)
        ↓
parser.service.js → CSV/XLSX parsed to JSON
        ↓
ai.service.js → Groq (Llama 3) generates narrative summary
        ↓
mailer.service.js → Resend sends email with summary
        ↓
Frontend shows success state
```

---

## 📝 Coding Conventions

- **Commits:** Conventional Commits format (`feat:`, `fix:`, `chore:`, `docs:`, `refactor:`)
- **Branches:** `feature/`, `fix/`, `chore/` prefixes
- **Code style:** No unnecessary abstraction — keep services thin and focused
- **Error handling:** All async routes use `try/catch` with centralized `errorHandler` middleware
- **Swagger:** Every endpoint must have JSDoc `@swagger` annotations

---

## 🔗 Reference Links

- [Google Stitch](https://stitch.withgoogle.com)
- [Stitch MCP Docs](https://stitch.withgoogle.com/docs/mcp/setup)
- [stitch-mcp npm package](https://github.com/Kargatharaakash/stitch-mcp)
- [Groq Console](https://console.groq.com)
- [Resend Dashboard](https://resend.com)
- [Render Dashboard](https://render.com)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/suchetrana)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/suchetrana)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
