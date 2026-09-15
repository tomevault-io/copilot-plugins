## kp-astro-ai

> - **Project Name**: KP Astro AI

# Claude Code Settings

## Project Information
- **Project Name**: KP Astro AI
- **Type**: Full-stack web application (React + FastAPI)
- **Description**: KP Astrology chart generation with AI-powered predictions

## Development Commands

### Frontend (React)
```bash
npm run dev          # Start development server
npm run build        # Build for production
npm run preview      # Preview production build
npm run lint         # Run ESLint
```

### Backend (Python)
```bash
cd backend
python run.py        # Start FastAPI development server
pip install -r requirements.txt  # Install dependencies
```

### Docker
```bash
docker-compose up    # Start both services with Docker
docker-compose build # Build Docker images
```

## Architecture Notes
- Frontend: React 18 + Vite on port 3000
- Backend: FastAPI + uvicorn on port 8000
- Services: Geocoding, KP Astrology, AI Chat, Vimshottari Dasha
- External APIs: OpenAI, Nominatim (geocoding)

## Environment Variables
- `OPENAI_API_KEY` - Required for AI chat functionality (set in backend/.env)

## Key Directories
- `src/components/` - React components
- `backend/services/` - Python service modules
- `backend/main.py` - FastAPI application entry point

## Testing & Quality
- Run `npm run lint` before commits
- Backend uses FastAPI's built-in validation
- Swiss Ephemeris for astronomical calculations

## Deployment
- Docker-ready with multi-stage build
- Serves React build from FastAPI in production
- Health check endpoint: `/health`

---
> Source: [etandon-coatue/kp-astro-ai](https://github.com/etandon-coatue/kp-astro-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-15 -->
