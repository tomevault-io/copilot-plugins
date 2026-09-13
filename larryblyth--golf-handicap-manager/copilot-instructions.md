## golf-handicap-manager

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Golf Handicap Manager is a web application for tracking golf rounds and calculating handicaps. The application features:

- **Landing Page**: Marketing site with user authentication via Google OAuth
- **Dashboard**: Authenticated user interface for viewing golf data
- **Gmail Integration**: Automatic extraction of golf round data from emails
- **SQLite Database**: Storage for users and golf rounds

## Technology Stack

- **Backend**: Python 3 with built-in `http.server`
- **Frontend**: Vanilla HTML, CSS, and JavaScript
- **Database**: SQLite with manual schema management
- **Authentication**: Google OAuth 2.0 with JWT
- **Email Integration**: Gmail API (in development)

## Project Structure

```
golf-handicap-manager/
├── server.py              # Main Python HTTP server with routing
├── index.html             # Landing page with authentication
├── app.html              # Dashboard application interface
├── app.js                # Dashboard JavaScript functionality
├── auth.js               # Authentication management
├── gmail-integration.js  # Gmail API integration
├── styles.css            # Main website styles
├── app-styles.css        # Dashboard-specific styles
├── golf_handicap.db      # SQLite database (auto-created)
└── CLAUDE.md            # This documentation file
```

## Development Commands

```bash
# Start the development server
python3 server.py

# The application will be available at:
# - Main site: http://localhost:8000
# - Dashboard: http://localhost:8000/app
```

## Key Features

1. **User Authentication**: Google OAuth integration with JWT token handling
2. **Golf Round Tracking**: Store rounds with course, score, date, and metadata
3. **Dashboard Interface**: Modern SPA-style interface with navigation
4. **Gmail Integration**: Extracts golf data from emails (Golfshot, etc.)
5. **Database Management**: SQLite with automatic table creation and indexing

## Database Schema

- **users**: id, name, email, picture, provider, created_at
- **golf_rounds**: id, user_id, course, score, par, date, source, rating, slope, email_subject, created_at

## API Endpoints

- `POST /api/search-golf-emails`: Search and extract golf round data
- `POST /api/save-user`: Save user authentication data

## Current Status

The application is fully functional with:
- ✅ User authentication system
- ✅ Basic dashboard interface
- ✅ Golf round data management
- ✅ Demo data for testing
- 🚧 Gmail integration (using sample data)
- 🚧 Real-time handicap calculation

---
> Source: [larryblyth/golf-handicap-manager](https://github.com/larryblyth/golf-handicap-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
