## studious2025

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
- `npm install` - Install dependencies
- `npm start` - Start the production server on PORT (default 3000)
- `npm run dev` - Start the development server with .env file support
- `npm test` - Run tests (currently no tests implemented)

### Environment Setup
The application requires a `.env` file with:
- `OPENAI_API_KEY` - OpenAI API key for AI tutoring features
- `OPENAI_PROJECT_ID` - OpenAI project ID
- `PORT` - Server port (optional, defaults to 3000)

## Architecture

### Overview
Studious2025 is an educational platform with AI-powered tutoring. It uses a traditional client-server architecture:
- **Backend**: Node.js/Express server with SQLite database
- **Frontend**: Static HTML/CSS/JavaScript served by Express
- **AI Integration**: OpenAI API for intelligent tutoring responses

### Server Structure
- **Entry Point**: `server/server.js` - Express server setup, serves static files from client directory
- **Routes**: 
  - `/auth/*` - Authentication endpoints (register, login, logout)
  - `/home/*` - Home page endpoints with AI integration
  - `/topics/*` - Educational topics configuration
- **Database**: SQLite with users table for authentication
- **Session Management**: Express sessions for user authentication state

### Key Components
1. **Authentication System** (`server/api/authController.js`):
   - Password hashing with bcrypt
   - Session-based authentication
   - SQLite user storage

2. **AI Integration** (`server/api/simpleAiController.js`):
   - OpenAI API integration for tutoring responses
   - HTML-formatted responses for direct rendering

3. **Topics System** (`server/config/topics.json`):
   - Hierarchical subject structure (topics → subtopics)
   - Custom prompt prefixes for each subject
   - Suggested questions for student guidance

### Client Structure
- Static files in `client/` directory
- Separate HTML pages for different features (home, signup, about)
- Modular JavaScript files for different functionalities
- CSS organized by component/page

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ravick)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ravick)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
