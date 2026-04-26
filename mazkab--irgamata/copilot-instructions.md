## irgamata

> Irgamata is a professional career platform with AI-powered features:

# CLAUDE.md - Irgamata Project Context

## Project Overview
Irgamata is a professional career platform with AI-powered features:
- **I-CV**: AI-powered CV/Resume builder (uses DeepSeek API)
- **I-Interview**: AI interview simulator (uses ElevenLabs voice + Gemini evaluation)
- **I-Hire**: Professional profile dashboard based on interview history

## Tech Stack
- **Frontend**: React + TypeScript + Vite, Tailwind CSS, shadcn/ui components
- **Backend**: Python FastAPI, PostgreSQL (asyncpg), JWT auth
- **AI**: Google Gemini 2.5 Flash (evaluation/reports), DeepSeek (CV), ElevenLabs (voice)
- **Deployment**: Docker Compose → GitHub Actions CI/CD → VPS (production.ehw.my.id)
- **Infra**: Nginx reverse proxy, GHCR image registry

## Key Architecture
- `web/src/pages/` — Page components (HirePage, InterviewPage, CvPage, etc.)
- `web/src/components/` — Reusable UI components (BlurText, FadeContent, shadcn)
- `web/src/contexts/AuthContext` — JWT auth context
- `web/src/lib/api.ts` — API URL helper
- `backend/main.py` — All FastAPI endpoints
- `backend/database.py` — PostgreSQL pool & table definitions
- `backend/auth.py` — JWT token + password hashing
- `backend/cv_service.py` — CV improvement logic

## Important Patterns
- **Auth**: JWT Bearer token, `get_current_user` dependency
- **API**: All endpoints under `/api/` prefix
- **Hire Page**: Has hardcoded DUMMY_PROFILE fallback data (Mazka Buana Hidayat) that shows when no real interview sessions exist. This ensures the dashboard always has showcase data.
- **Interview flow**: prepare → create-agent (ElevenLabs) → voice convo → evaluate (Gemini) → auto-save to DB
- **Deployment**: Push to `main` triggers GitHub Actions → build Docker images → deploy to VPS

## Recent Changes
- **Hire Page**: Enhanced dummy showcase data with:
  - 3 interview sessions (Tokopedia, Gojek, Google Indonesia) with full strengths/weaknesses/improvements
  - Pre-filled AI report (Grade B, executive summary, career advice, recommended focus)
  - CV Profile Summary section (profile info, 15 skills, 2 experiences, education, 3 projects, 2 certifications, languages)
  - Each session card now shows strengths, weaknesses, and improvement suggestions
- **Interview Page**: "Pasang Ekstensi Sistem" button now links to Google Drive folder
- Fallback logic: if API returns 0 sessions OR errors, dummy data + report are displayed
- **CV Editor**: Added 6th wizard step "Download" with PDF generation
  - 3 templates: Professional (serif ATS), Modern (violet sidebar), Minimalist (ultra-clean)
  - Client-side PDF via html2pdf.js (dynamic import, code-split)
  - Template preview panel before download
  - Files: `CvDownloadStep.tsx`, `cv-pdf-templates.tsx`, `cv-template-config.tsx`
- **Backend CV Improve**: DeepSeek→Gemini automatic fallback on failure
- **Backend Logging**: Detailed logs at each stage of CV improve (extract, API call timing, response status)
- **CV Chat Interface**: Replaced form wizard (scratch flow) with AI chat interface
  - Backend: `/api/cv/chat` endpoint using DeepSeek + Gemini fallback
  - Frontend: `CvChatView.tsx` — chat panel + live PDF preview sidebar
  - `CvLivePreview.tsx` — animated completeness ring (0-100), template selector, scaled PDF preview
  - `cv-chat-messages.tsx` — chat bubble components
  - `cv-completeness.ts` — weighted completeness scoring
  - Improve flow still uses the original form wizard (`CvEditorView`)
- **I-Hire → i-Dashboard**: Renamed across Navbar, LandingPage, App, HirePage, PusatResponsPage, KetentuanPage. Route changed from `/hire` to `/dashboard`
- **CV Improve — Job Targeting**: Added target_job field to `/api/cv/improve` endpoint
  - Frontend: job role input + industry selector with 12 presets
  - Backend: `build_cv_improvement_prompt` injects job-targeting instructions into AI prompt
  - AI reframes summary, skills, and highlights for the target role without fabricating facts

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MazkaB) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
