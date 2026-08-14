## game-cam-webapp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Game Cam WebApp - A full-stack application for managing and viewing wildlife camera footage with interactive map-based filtering. Uses Python FastAPI backend and React frontend.

## Development Commands

### Frontend
```bash
cd frontend
npm install          # Install dependencies
npm run dev          # Start development server on http://localhost:5173
npm run lint         # Run ESLint

# IMPORTANT: Never run build during development - it interferes with hot reloading
# npm run build      # Production build (ONLY for deployment, not development)
# npm run preview    # Preview production build (ONLY for testing production builds)
```

### Backend
```bash
cd backend
pip install -r requirements.txt  # Install Python dependencies
python main_server.py            # Start FastAPI server on port 10000
```

## Architecture

### Tech Stack
- **Backend**: FastAPI (Python) with uvicorn server
- **Frontend**: React 19 with Vite build tool
- **Maps**: Leaflet.js (via CDN in index.html)
- **Video**: Native HTML5 video with HTTP range request streaming

### Key Components

#### Frontend Structure
- `frontend/src/App.jsx` - Main application with map and filter controls
- `frontend/src/components/MapPane.jsx` - Interactive Leaflet map with camera sites
- `frontend/src/components/BottomPanel.jsx` - Filter controls and video list
- `frontend/src/components/Player.jsx` - Video player with queue management
- `frontend/src/api.js` - API communication utilities

#### Backend Structure
- `backend/main_server.py` - FastAPI server with endpoints:
  - `GET /api/labels` - Filter and retrieve video metadata
  - `GET /api/video` - Stream video files with range request support
- `backend/sorting.py` - Data processing and filtering logic

### Data Flow
1. Frontend sends filter parameters to `/api/labels`
2. Backend processes JSON metadata files and returns filtered results
3. Video streaming via `/api/video` with direct file system access
4. Session storage maintains video queue across browser tabs

## Important Configuration

### Backend Settings (main_server.py)
- Server runs on port 10000
- CORS allows all origins (development setting)
- JSON_DIR path hardcoded for metadata location
- Direct file system access for video streaming

### Frontend Build (vite.config.js)
- Vite configuration with React plugin
- Development server on port 5173
- Production build outputs to frontend/dist

## Development Notes

### Video Streaming
- Backend supports HTTP range requests for efficient video streaming (1MB chunks)
- Videos served directly from file system paths stored in JSON metadata
- MIME type detection via Python's `mimetypes.guess_type()`
- Error handling: HTTPException 404 for missing files, 416 for invalid ranges

### Map Integration
- Leaflet loaded via CDN in public/index.html (v1.9.3)
- Three tile layers available: Topo, OSM, Satellite
- GPS-based camera site markers with interactive filtering

### State Management
- React hooks (useState, useEffect) for component state
- Session storage for video queue persistence across browser tabs
- No global state management library
- Cross-tab communication via `sessionStorage` for video queue

### API Communication
- Frontend api.js handles all backend communication
- Query parameters for filtering: sites, animals, actions, add_labels, start, end, restricted
- No authentication/authorization implemented
- Minimal error handling on frontend for API failures

### Data Processing
- **External Dependency**: JSON metadata from `C:\Users\jrsch\Documents\Visual Studio Code\Deer Score Prediction\yolov5\GameCamFiles\GameCamClassifierJSONs`
- **Dynamic Loading**: `load_latest_json()` loads most recent JSON by creation time
- **Fallback**: Defaults to `GameCamClassifiers.json` if no files found
- **Filtering Logic**: Complex multi-criteria with special 'none' handling for empty categories

## Filtering Specification
### Overview
The filtering system should allow users to filter wildlife camera videos based on multiple criteria: camera sites, animals, actions, and additional labels. The system should support both inclusive (OR) and exclusive (AND) logic depending on the filter type.

### Detailed Filtering Rules

Filtering has a forward mode and a backward mode. 

### Forward mode 
This starts with the cam sites category considering all videos that are described by one of the cam sites that are checked. Since every video must have a cam site label, there is no "none" label for cam sites. Then animals, actions, and additional labels are considered. Each of these categories has a "none" label in addition to all other labels from the json file. Each of these categories along with the none labels is complete, meaning that if no boxes are checked from that category, there can be no populated videos. There is also a date range category that filters videos based on date range. The videos list therefore represents the intesection of videos across categories of the union of videos within each category independently.

### Backward mode
This mode first considers the video list and compiles a list of all the unique entries for each category (except date range) associated with the videos. It then greys out entries that are not on the list for that category. Entries that are greyed out can still be checkmarked, but checking/unchecking them has no effect on the video list because of constraints in other categories.

### Development Utilities
- `copy_code.py` - Copies entire codebase to clipboard (excludes node_modules, .git, media files)
- No testing framework or test files present
- No Docker, CI/CD, or deployment automation
- No environment variables or .env configuration

## Important Development Rules
- **NEVER run `npm run build` or `npm run preview` during development** - these interfere with hot reloading
- Always use `npm run dev` for development work
- Only build for production deployment, never during development sessions

---
> Source: [thejsx/Game-Cam-WebApp](https://github.com/thejsx/Game-Cam-WebApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
