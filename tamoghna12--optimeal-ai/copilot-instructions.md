## optimeal-ai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FitSpice / Homeland Meals is a full-stack nutrition and recipe application focused on South Asian cuisine. It helps users track nutrition, convert traditional recipes to quick versions, and analyze food images using AI.

## Architecture

**Frontend** (`/frontend`):
- React 19 application with React Router for navigation
- Tailwind CSS for styling with Craco build configuration  
- Context-based state management for meal tray functionality
- Static JSON data files for recipes and ingredient pricing
- Axios for API communication

**Backend** (`/backend`):
- FastAPI server with MongoDB database integration
- Custom LLM integration via Groq API using Llama models for food analysis and recipe conversion
- Co-pilot chat functionality with professional ChatGPT-style interface
- RESTful API with `/api` prefix for all endpoints
- Pydantic models for data validation

## Development Commands

**Frontend Development:**
```bash
cd frontend
yarn install
yarn start    # Development server
yarn build    # Production build
yarn test     # Run tests
```

**Backend Development:**
```bash
cd backend
pip install -r requirements.txt
uvicorn server:app --reload --host 0.0.0.0 --port 8000
```

**Testing:**
```bash
# Run comprehensive backend API tests
python backend_test.py
```

**Code Quality:**
```bash
# Backend linting and formatting
cd backend
black .
isort .
flake8 .
mypy .

# Frontend linting
cd frontend
npm run lint  # Uses ESLint
```

## Key API Endpoints

**Core Functionality:**
- `POST /api/profile` - Create user profile with calculated BMR and calorie targets
- `POST /api/analyze-food` - Upload food image for AI nutritional analysis
- `POST /api/recipe` - Create recipe with AI-powered quick version conversion
- `GET /api/daily-stats/{user_id}/{date}` - Get daily nutrition and fitness statistics
- `GET /api/ingredient-substitutions/{ingredient}` - Get Western grocery substitutions

**Co-pilot Features:**
- `POST /api/copilot/chat` - Interactive chat assistant for cooking questions and guidance
- `POST /api/copilot/recipe-suggestions` - Get recipe suggestions based on available ingredients
- `POST /api/copilot/cooking-guidance` - Get step-by-step cooking guidance for recipes

## Database Collections

- `user_profiles` - User BMR calculations and dietary goals
- `food_entries` - AI-analyzed food intake with nutritional data
- `workout_entries` - Exercise logging with calorie burn estimates
- `recipes` - Traditional and AI-converted quick recipes

## Environment Configuration

The application requires `.env` files in both frontend and backend directories:
- Backend: MongoDB connection, LLM API keys, CORS settings
- Frontend: Backend API URL configuration

## Important Implementation Notes

**AI Integration:**
- Custom Groq API integration using Llama-3.1-8b-instant model
- Co-pilot chat interface with natural language processing
- Recipe suggestions and cooking guidance powered by AI
- All AI responses are structured for consistent parsing
- Fallback handling when AI parsing fails

**Nutrition Calculations:**
- BMR calculated using Mifflin-St Jeor equation 
- Activity level multipliers for daily calorie targets
- Goal-based calorie adjustments (±500 for weight goals)

**Frontend State Management:**
- `MealTrayContext` manages recipe selection and cost estimation
- User cookbook data stored in localStorage
- Recipe and ingredient data loaded from JSON files

## File Structure Highlights

- `/frontend/src/data/` - Static JSON files for recipes and ingredient pricing
- `/frontend/src/App.js` - Main React application with co-pilot chat interface
- `/frontend/src/App.css` - Custom styling including professional chat UI
- `/backend/server.py` - Main FastAPI application with all API endpoints
- `/backend/emergentintegrations/` - Custom LLM integration module
- `/backend_test.py` - Comprehensive API testing suite
- `/tests/` - Additional test utilities

## Co-pilot Features

**Chat Interface:**
- Professional ChatGPT-style design with message bubbles
- Real-time AI responses for cooking questions
- Markdown rendering for structured responses
- Two-mode interface: general chat and recipe suggestions

**Recipe Intelligence:**
- Ingredient-based recipe suggestions
- Step-by-step cooking guidance
- Traditional to quick recipe conversion
- Nutritional analysis and dietary adaptations

**UI/UX Improvements:**
- Clean, modern interface with proper contrast
- Responsive design for mobile and desktop
- Accessible color schemes and typography
- Branded as "Nutrichef AI"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Tamoghna12)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Tamoghna12)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
