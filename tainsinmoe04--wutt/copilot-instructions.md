## wutt

> > "ဒီနေ့ ဘာဝတ်ရမလဲ?" — Myanmar's first AI outfit advisor

# WUTT — AI Personal Stylist (Myanmar)
> "ဒီနေ့ ဘာဝတ်ရမလဲ?" — Myanmar's first AI outfit advisor

## Project Overview
Web app: User uploads wardrobe photos + inputs occasion/weather/body size → OpenAI Vision analyzes → outfit recommendation output.

## Tech Stack
- Frontend: HTML + CSS (vanilla) + JavaScript (vanilla)
- Backend: Python FastAPI
- Database: SQLite (MVP) → PostgreSQL later
- Image Storage: Cloudinary
- AI: OpenAI GPT-4o Vision API
- Weather: OpenWeatherMap API
- Auth: JWT (python-jose)
- Deploy: Render.com

## File Structure
```
wutt/
├── CLAUDE.md
├── .env                    # NEVER commit this
├── .gitignore
├── render.yaml
├── .claude/
│   ├── skills/
│   │   ├── styling.md      # AI outfit logic skill
│   │   └── backend.md      # FastAPI patterns skill
│   └── hooks/
│       └── safety-check.sh # Block dangerous commands
├── frontend/
│   ├── index.html          # Landing + Auth pages
│   ├── dashboard.html      # Main app
│   ├── style.css           # WUTT design system
│   └── app.js              # Frontend logic
└── backend/
    ├── main.py             # FastAPI entry point
    ├── database.py         # SQLite + SQLAlchemy
    ├── models.py           # DB models
    ├── requirements.txt
    └── routes/
        ├── auth.py         # Register/Login/JWT
        ├── profile.py      # Body size + preferences
        ├── wardrobe.py     # Cloudinary upload
        └── stylist.py      # OpenAI Vision logic
```

## Database Schema
```sql
users: id, email, password_hash, created_at
profiles: id, user_id, height_cm, skin_tone, style_preference, location_city, updated_at
wardrobes: id, user_id, cloudinary_url, cloudinary_public_id, category, color, description, uploaded_at
style_sessions: id, user_id, occasion, weather_desc, temperature_c, location, ai_response, created_at
```

## API Endpoints
```
POST /auth/register
POST /auth/login
GET  /profile/{user_id}
PUT  /profile/{user_id}
POST /wardrobe/upload
GET  /wardrobe/{user_id}
DELETE /wardrobe/{item_id}
POST /stylist/recommend
GET  /stylist/history/{user_id}
```

## Design System — WUTT Colors
```
Base Light:   #FFFFFF
Base Dark:    #0A0A0A
Primary:      #88A2FF  (Periwinkle — buttons, CTA)
Deep:         #253A82  (Navy — headers, dark bg)
Energy:       #E3FC87  (Neon Green — AI results, highlights)
Blush:        #FFB2F7  (Pink — tags, badges)
Sky:          #C0E0FF  (Light Blue — input fields)
Lavender:     #AB9DFF  (Purple — secondary elements)
```

## Coding Standards
- Python: 4-space indent, type hints always, docstrings on all functions
- JS: 2-space indent, async/await preferred, no jQuery
- CSS: BEM naming convention, CSS variables for all colors
- All API responses: `{"status": "success/error", "data": {}, "message": ""}`
- Error handling: always try/catch in JS, try/except in Python

## Commands
```bash
# Backend
cd backend && pip install -r requirements.txt
uvicorn main:app --reload --port 8000

# Frontend
cd frontend && python -m http.server 3000

# Test API
curl http://localhost:8000/health
```

## Security Rules
- NEVER hardcode API keys — use .env only
- NEVER commit .env file
- All passwords: bcrypt hash only
- JWT tokens: 24hr expiry
- Cloudinary: server-side upload only (never expose secret to frontend)
- CORS: restrict to frontend domain only in production

## Known Patterns / Don't Repeat These Mistakes
- OpenAI Vision: send images as base64 OR url — not both
- Cloudinary upload: use `upload_preset` for unsigned, `api_secret` for signed (use signed/server-side)
- FastAPI CORS: add middleware BEFORE routes
- SQLite: use `check_same_thread=False` for FastAPI
- JWT auth: backend sets httpOnly cookie (SameSite=None in prod) AND returns token in response body. Frontend stores token in localStorage and sends via `Authorization: Bearer` header for cross-origin API calls. `get_current_user` checks header first, then cookie fallback.
- NEVER expose JWT to third-party scripts — localStorage is used as a cross-origin necessity, not a preference

## What Claude Should NOT Do
- Never modify .env file
- Never push directly to main branch
- Never delete wardrobe images without user confirmation
- Never expose API keys in frontend JS
- Never skip input validation on API endpoints

---
> Source: [tainsinmoe04/wutt](https://github.com/tainsinmoe04/wutt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
