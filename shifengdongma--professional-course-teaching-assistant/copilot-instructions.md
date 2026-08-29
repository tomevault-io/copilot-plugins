## professional-course-teaching-assistant

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PCTS (专业课教学辅助系统) is an AI-powered teaching assistant system for professional courses. It's a full-stack application with a Flask backend and Vue 3 frontend, featuring AI chat powered by a locally fine-tuned LLM.

## Development Commands

### Backend (from `backend/` directory)

```bash
# Install dependencies
pip install -r requirements.txt

# Run development server
python run.py
# Server runs on http://0.0.0.0:5000

# Database migrations
python manage_migrations.py  # Uses Flask-Migrate
```

### Frontend (from `frontend/` directory)

```bash
# Install dependencies
npm install

# Run development server
npm run dev
# Server runs on http://0.0.0.0:5173

# Build for production
npm run build

# Preview production build
npm run preview
```

### Database Setup

The database scripts are in `dataset/` directory:
- `db_create.py` - Initialize database tables
- Individual `create_*_table.py` files for each table

## Architecture

### Backend Structure (`backend/`)

```
backend/
├── app/
│   ├── __init__.py        # Flask app factory, blueprint registration
│   ├── models/            # SQLAlchemy ORM models (user, chat, ebook, note, tool, ai_classroom)
│   ├── routes/            # API endpoints (auth, user, chat, ebook, note, tool, ai_classroom)
│   ├── services/
│   │   └── model_service.py  # LLM loading & inference (uses unsloth)
│   └── utils/             # Utility functions
├── config.py              # Database, JWT, model path configuration
├── migrations/            # Flask-Migrate migrations
└── uploads/               # User uploaded files (ebooks)
```

**API Blueprints:**
- `/api/auth` - Authentication (register, login, logout)
- `/api/user` - User profile management
- `/api/chat` - Chat sessions and messages
- `/api/ebooks` - Ebook upload and management
- `/api/notes` - Note CRUD operations
- `/api/tools` - Teaching tools list
- `/api/ai-classroom` - AI classroom features

### Frontend Structure (`frontend/src/`)

```
src/
├── main.js               # App entry point
├── App.vue               # Root component
├── router/index.js       # Vue Router configuration with auth guards
├── utils/
│   ├── auth.js           # Token management
│   └── request.js        # Axios wrapper with auth headers
├── components/           # Reusable components (ChatBox, EbookReader, NoteEditor, ToolCard)
└── pages/                # Route components organized by feature
    ├── Auth/             # Login, Register
    ├── Home/             # Dashboard, index
    ├── Chat/             # ChatHome, ChatSession
    ├── Ebook/            # EbookShelf, EbookReaderPage
    ├── Note/             # NoteList, NoteEditorPage
    ├── Tools/            # ToolList
    ├── User/             # UserProfile
    ├── Settings/         # Settings
    └── AIClassroom*/     # AI classroom pages
```

## Key Technologies

**Backend:**
- Flask 2.0.1 with Flask-SQLAlchemy, Flask-JWT-Extended, Flask-Migrate
- MySQL database via PyMySQL
- Unsloth + Transformers for LLM inference (4-bit quantization)

**Frontend:**
- Vue 3 with Vite 3
- Element Plus for UI components
- Pinia for state management
- Vue Router with hash history
- Axios for HTTP requests
- Marked for Markdown rendering

## Configuration

Backend config is in `backend/config.py`:
- Database connection (MySQL with socket)
- JWT settings
- Model path for fine-tuned LLM
- DeepSeek API fallback

Frontend dev server proxies `/api` requests to the backend (configured in `vite.config.js`).

## Database Tables

- `users` - User accounts with profile fields
- `chat_sessions` - Chat session metadata
- `chat_messages` - Individual messages in sessions
- `ebooks` - Uploaded ebook files
- `notes` - User notes
- `tools` - Teaching tool links (system-managed)
- `ai_classroom_*` - AI classroom related tables

---
> Source: [shifengdongma/professional-course-teaching-assistant](https://github.com/shifengdongma/professional-course-teaching-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
