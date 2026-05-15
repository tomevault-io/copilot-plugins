## eventsntickets-scanner

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an event ticket scanner frontend application built with React and Vite. The application allows moderators to scan and verify event tickets using QR codes or OCR (Optical Character Recognition). It connects to a backend API hosted on Vercel.

## Development Commands

### Running the Application
- `npm run dev` - Start the development server
- `npm run build` - Build for production
- `npm run preview` - Preview the production build
- `npm run lint` - Run ESLint to check code quality

### Development Server
The dev server runs with a proxy configuration that redirects `/api/*` requests to `https://event-ticket-backend.vercel.app`.

## Architecture & Key Concepts

### Backend Integration
- **API Base URL**: Configured in `src/ServerConfig.js`
- **Production URL**: `https://event-ticket-backend.vercel.app/api/v1/`
- **Dev URL** (commented out): `https://dev-event-ticket-server.vercel.app/api/v1/`
- **Vite Proxy**: Development requests to `/api` are proxied to the backend (see vite.config.js:7-14)

### Authentication Flow
The application uses a token-based authentication system:
1. Moderators log in via the `ModeratorLogin` component (root path `/`)
2. Login endpoint: `POST /moderator/login` with email and password
3. On success, stores `moderator-token` and `moderator-data` in localStorage
4. Protected routes check for these credentials using `ProtectedModeratorRoute`
5. All API requests include `Authorization: Bearer ${token}` header

### Main Application Flow
1. **Login** (`/`) - ModeratorLogin component authenticates the user
2. **Scanner** (`/moderator/scanner`) - TicketScanner component (protected route)
   - Fetches today's stats from `GET /moderator/stats/{moderatorId}`
   - Fetches recent scans from `GET /moderator/recent/{moderatorId}`
   - Scans tickets via `POST /moderator/scan-ticket`

### Ticket Scanning System

The application supports multiple scanning modes:

**OCR Scanner Mode** (Primary):
- Uses OCR.space PRO API for text recognition from camera feed
- API Key: Hardcoded in TicketScanner.jsx:884
- Two endpoints with automatic fallback (primary and secondary)
- Continuous camera feed with 3-second scan intervals
- Confirmation modal before processing ticket
- Extracts ticket codes using multiple regex patterns

**Manual Entry Mode**:
- Manual text input for ticket codes
- Useful when camera scanning fails

**QR Scanner Mode** (Legacy, commented out):
- Uses `react-qr-barcode-scanner` library
- Code remains in the file but is not actively used

### State Management
No global state management library is used. The application relies on:
- React component state (useState)
- localStorage for authentication persistence
- React Router for navigation

### Key Components Structure

**TicketScanner.jsx** (src/pages/TicketScanner/TicketScanner.jsx:806-1951):
- Most complex component with OCR scanning logic
- Manages camera feed, OCR processing, and ticket verification
- Uses refs (videoRef, canvasRef, scanIntervalRef) for camera and scanning
- Shows confirmation modal before processing scanned codes
- Displays verification results including buyer info, event details, and seat information
- Downloads scan logs as JSON

**ModeratorLogin.jsx** (src/pages/Login/ModeratorLogin.jsx):
- Simple login form with email/password
- Stores auth token and moderator data in localStorage
- Redirects to scanner on successful login

**ProtectedModeratorRoute.jsx**:
- Wrapper component that checks for authentication
- Redirects to login if no token/data found

### Styling
- **TailwindCSS** v4.1 with DaisyUI components
- Uses `@tailwindcss/vite` plugin
- Custom gradient backgrounds and card-based UI
- Responsive design with mobile-first approach

### Important Implementation Details

**OCR Ticket Code Extraction** (TicketScanner.jsx:1179-1215):
The system uses a sophisticated pattern matching system to extract ticket codes from OCR text:
- Cleans text by removing whitespace and converting to uppercase
- Tries multiple regex patterns in order of priority (10+ chars, 8+ chars, etc.)
- Filters results to ensure validity (has numbers, not all same character, etc.)
- Returns the longest valid match found

**Ticket Verification Response Structure**:
The backend returns verification results containing:
- `status`: "valid", "used", or "invalid"
- `message`: Human-readable status message
- `verificationResult`: Object with buyer, event, and seat details
  - `buyer`: { name, email }
  - `event`: { title, date, time, location }
  - `totalSeats`, `usedScans`, `remainingSeats`: Seat tracking info
  - `purchaseDate`, `scanTime`, `scannedBy`: Metadata

**Sound Feedback**:
The app uses Web Audio API to generate success/error sounds based on scan results.

### Firebase
Firebase is configured (src/firebase/firebase.config.js) but the configuration file is empty/placeholder. The AuthProvider component is also a placeholder and not actively used.

### Routing Structure
```
/ → ModeratorLogin
/moderator/scanner → TicketScanner (protected)
```

## Common Pitfalls

1. **API Response Parsing**: The code explicitly handles text responses and parses them as JSON with error handling. Always use this pattern when making API calls.

2. **OCR API Key**: The OCR.space API key is hardcoded in TicketScanner.jsx. In production, this should be moved to environment variables.

3. **Camera Permissions**: OCR scanner requires camera permissions. Handle permission errors gracefully.

4. **LocalStorage Dependencies**: The app heavily relies on localStorage for auth. Clear localStorage when debugging auth issues.

5. **Backend URL Changes**: Remember to update both `ServerConfig.js` and `vite.config.js` proxy if backend URL changes.

## Environment-Specific Behavior

- **Development**: Uses Vite proxy to avoid CORS issues
- **Production**: Directly calls backend API URLs from ServerConfig.js

---
> Source: [akternowshin-hb/Eventsntickets-scanner](https://github.com/akternowshin-hb/Eventsntickets-scanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
