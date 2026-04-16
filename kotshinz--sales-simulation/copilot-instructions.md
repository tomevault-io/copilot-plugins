## sales-simulation

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Japanese sales simulation application (営業シミュレーション) that provides AI-powered role-playing scenarios for sales training. The system uses Dify AI platform for intelligent conversation and provides feedback on sales performance.

## Development Commands

### Starting the Application
```bash
# Start both frontend and backend with Docker Compose
docker-compose up --build

# Alternative: Start services individually
cd backend && python app.py  # Backend on port 8000
cd react-app && npm start    # Frontend on port 3000
```

### Frontend Development
```bash
cd react-app
npm install          # Install dependencies
npm start           # Development server (port 3000)
npm run build       # Production build
npm test            # Run tests with Jest/Testing Library
```

### Backend Development
```bash
cd backend
pip install -r requirements.txt  # Install Python dependencies
python app.py                    # Start Flask server (port 8000)
```

## Architecture

### Frontend (React + TypeScript)
- **Framework**: React 19 with TypeScript
- **UI Library**: Material-UI (@mui/material) v7 with Emotion styling
- **Routing**: React Router v6
- **Forms**: React Hook Form with Zod validation  
- **HTTP Client**: Dual approach - Axios (http.ts) and native fetch (chatApi.ts)
- **State Management**: Context API for auth, singleton ChatStore for conversation tracking

### Backend (Python Flask)
- **Framework**: Flask 3.0 with CORS support
- **AI Integration**: Dify platform via REST API
- **Database**: SQLite with JSON data files for simulation scenarios
- **Authentication**: JWT-based (currently disabled in frontend)

### Key API Endpoints
- `POST /ask` - Send messages to AI simulation
- `GET /history` - Retrieve conversation history
- `GET /simulations` - Get simulation scenarios
- `POST /review` - Get AI feedback on conversations

## Project Structure

### Frontend (`/react-app/src/`)
```
├── components/        # Reusable UI components
│   ├── auth/         # Login/Register forms
│   └── layout/       # Layout components (DashboardLayout, Chatboard)
├── pages/            # Route-level components
│   ├── Login/
│   ├── Dashboard/
│   ├── SimulationSetup/
│   ├── SimulationChat/    # Main chat interface
│   ├── SimulationResult/
│   └── ExistingCases/
├── contexts/         # React Context providers (AuthContexts)
├── api/             # API communication layers
│   ├── chatApi.ts   # Dify chat integration (fetch-based)
│   ├── http.ts      # Axios client with interceptors
│   └── simulations.ts # Simulation data API
├── types/           # TypeScript type definitions
├── state/           # Application state management
└── themes/          # Material-UI theme configuration
```

### Backend (`/backend/`)
```
├── app.py           # Main Flask application
├── db/              # Database files and utilities
│   ├── database.db  # SQLite database
│   ├── *.json       # Simulation scenario data
│   └── make_init_db.py # Database initialization
└── requirements.txt # Python dependencies
```

## Development Patterns

### API Communication
The codebase uses two HTTP client patterns:
1. **chatApi.ts**: Native fetch API for Dify integration with detailed error handling
2. **http.ts + api layers**: Axios-based client with interceptors for general API calls

### Component Architecture
- Pages are route-level containers in `/pages/[PageName]/`
- Each page exports from `index.ts` for clean imports
- Reusable components in `/components/` organized by domain
- TypeScript interfaces centralized in `/types/`

### State Management
- **Authentication**: React Context (`AuthContexts.tsx`)
- **Chat Conversations**: Singleton ChatStore for conversation_id mapping
- **API State**: React hooks pattern (useSimulationsQuery)

### Styling
- Material-UI with custom theme in `/themes/`
- Emotion for styled components
- Consistent color palette in `themes/colors.ts`

## Key Features

### Chat System
- Persistent conversations using Dify conversation_id
- Message history retrieval and display
- Real-time AI responses in Japanese
- Voice input support (isVoice flag in messages)

### Simulation Workflow
1. **Setup** (`/setup`) - Configure simulation parameters
2. **Chat** (`/categories/:orgId/chats/:chatId`) - Role-play conversation
3. **Review** (`/review`) - AI-generated performance feedback
4. **Results** (`/sim/:id/result`) - Detailed analysis

### Database Schema
- `simulation_status` - Active simulations
- `contact_persons` - Customer contact information  
- `customers` - Company/client data
- JSON files contain product catalogs and scenario templates

## Environment Configuration

### Required Environment Variables
```bash
# Backend (.env file)
DIFY_API_BASE=https://api.dify.ai    # Dify platform URL
DIFY_API_KEY=your_api_key            # Main simulation API key
DIFY_REVIEW_API_KEY=your_review_key  # Review/feedback API key
ENDPOINT=your_db_endpoint            # Database API endpoint

# Frontend
REACT_APP_API_BASE=http://127.0.0.1:8000  # Backend URL
```

## Testing

Frontend uses Jest + React Testing Library:
```bash
cd react-app
npm test           # Run tests
npm test -- --coverage  # With coverage report
```

## Common Tasks

### Adding New Pages
1. Create folder in `/react-app/src/pages/NewPage/`
2. Add component `NewPage.tsx` and export in `index.ts`
3. Add route in `App.tsx`
4. Update type definitions if needed

### Extending Chat Features
- Chat logic is in `SimulationChat.tsx`
- API calls use `chatApi.ts` with conversation tracking via `ChatStore`
- Message types defined in `types/index.ts`

### Database Modifications
- Update SQLite schema in `backend/db/`
- Modify API endpoints in `backend/app.py`
- Update TypeScript types to match backend changes

## Notes

- The application is primarily in Japanese with some English code comments
- Authentication system exists but is currently disabled (see commented code in App.tsx)
- Dify AI platform integration is central to the chat functionality
- Docker Compose setup enables easy development environment

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KotShinZ) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
