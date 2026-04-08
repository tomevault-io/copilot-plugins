## okta-realms-creator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ⚠️ Important Notice

**This is a proof of concept tool and is NOT officially supported by Okta.** The codebase should be reviewed by the customer's security and engineering teams before any production use.

## Project Overview

Okta Realm Management Admin UI - A full-stack application that enables Okta administrators to bulk-create authorization servers (realms) for partner organizations by uploading CSV files.

**Tech Stack:**
- **Frontend**: React 18 + TypeScript, Vite, Material UI, Okta React SDK, React Router, Axios
- **Backend**: Node.js + Express + TypeScript, Okta Node.js SDK, Multer, csv-parser

## Development Commands

### Backend (port 3001)
```bash
cd backend
npm install           # Install dependencies
npm run dev          # Start development server with hot reload (tsx watch)
npm run build        # Compile TypeScript to dist/
npm start            # Run compiled JavaScript from dist/
```

### Frontend (port 5173)
```bash
cd frontend
npm install          # Install dependencies
npm run dev         # Start Vite development server
npm run build       # Build for production
npm run preview     # Preview production build
npm run lint        # Run ESLint
```

## Architecture

### Monorepo Structure

The project is organized as a monorepo with separate frontend and backend directories:

```
SPA-admin-UI/
├── backend/           # Express API server
│   ├── src/
│   │   ├── controllers/   # Request handlers for routes
│   │   ├── routes/        # Express route definitions
│   │   ├── services/      # Business logic (OktaService for SDK calls)
│   │   ├── middleware/    # Custom middleware (CSV parser)
│   │   └── types/         # Shared TypeScript types
│   └── .env              # Backend environment config
└── frontend/          # React SPA
    ├── src/
    │   ├── components/    # Reusable UI components (FileUpload, ResultsDisplay)
    │   ├── pages/         # Page components (Dashboard, LoginCallback)
    │   ├── services/      # API client (apiService.ts)
    │   ├── config/        # Okta configuration
    │   └── types/         # Frontend TypeScript types
    └── .env              # Frontend environment config
```

### Key Architectural Patterns

#### Authentication Flow (Frontend)
- Uses Okta's OAuth 2.0 / OIDC flow with PKCE
- `@okta/okta-react` provides `<Security>` context and `<SecureRoute>` wrapper
- All routes protected by `<SecureRoute>` except `/login/callback`
- `useOktaAuth()` hook provides access to auth state and tokens throughout app
- Access tokens automatically included in API requests via `Authorization: Bearer` header

#### Backend API Design
- **OktaService** (`services/oktaService.ts`): Wraps Okta SDK client, handles all Okta API calls
- **RealmController** (`controllers/realmController.ts`): Business logic for realm operations
- **Route definitions** (`routes/realmRoutes.ts`): Express routes with Multer middleware for file uploads
- Controllers are instantiated with dependencies in `index.ts` (basic dependency injection)

#### CSV Processing Flow
1. Frontend: User uploads CSV via FileUpload component
2. Backend: Multer middleware stores file in memory buffer
3. Backend: `csvParser` middleware converts buffer to PartnerOrganization array
4. Backend: `OktaService.createMultipleRealms()` for each organization:
   - Creates realm with name, domains, and realmType='PARTNER'
   - Creates realm assignment rule matching users by email domain
   - Activates the realm assignment rule
5. Backend: Returns summary with success/failure for each organization
6. Frontend: ResultsDisplay component shows detailed results table

#### API Communication
- Frontend uses centralized `apiService` (Axios instance)
- Base URL configured via `VITE_API_URL` environment variable
- All requests include Okta access token in Authorization header
- Backend validates JWT tokens cryptographically on every request
- Backend checks user authorization (admin group membership)
- CORS configured on backend to accept requests from frontend URL only

### Important Conventions

#### TypeScript Types
- Shared types between frontend/backend defined separately in each `/types` directory
- Key types: `PartnerOrganization`, `RealmCreationResult`, `UploadResponse`
- Frontend types mirror backend response structures

#### File Upload Handling
- Only CSV files accepted (validated by Multer and frontend)
- 5MB file size limit enforced on backend
- CSV columns: `name` (required), `domain` (required), `description` (optional)
- Column names are case-insensitive (supports `name`, `Name`, etc.)

#### Security Features
- **JWT Validation**: All API requests verified using `@okta/jwt-verifier`
- **Authorization**: User group membership checked for admin privileges
- **Input Sanitization**: CSV data validated and sanitized to prevent:
  - CSV injection attacks
  - Regex injection in expressions
  - XSS attempts
  - Malformed domain names
- **Domain Escaping**: Special regex characters properly escaped in realm assignments

#### Error Handling
- Backend catches errors per-organization, returns partial success results
- Frontend displays per-organization status in results table
- API errors shown in Material UI Alert components
- Security errors (401/403) clearly communicated to user

#### Okta SDK Usage
- Backend uses `@okta/okta-sdk-nodejs` Client class
- Realms created via `realmApi.createRealm()` with profile containing name, domains, and realmType='PARTNER'
- Realm assignments created via `realmAssignmentApi.createRealmAssignment()` with conditions using Okta Expression Language
- Assignments activated via `realmAssignmentApi.activateRealmAssignment()`
- Expression matches users: `String.stringContains(user.login, "@domain.com")`
- 500ms delay between creation requests to avoid rate limiting

## Configuration

### Backend Environment Variables (.env)
```env
OKTA_DOMAIN=your-domain.okta.com        # Okta tenant domain (without https://)
OKTA_API_TOKEN=your-api-token            # Okta API token with admin privileges
PORT=3001                                # Backend server port
FRONTEND_URL=http://localhost:5173       # Frontend URL (for CORS)
```

**Note**: The backend uses an API token to authenticate with Okta's Management API, not OAuth credentials. The SPA authentication (client ID) is only used in the frontend.

### Frontend Environment Variables (.env)
```env
VITE_OKTA_DOMAIN=your-domain.okta.com              # Okta tenant domain
VITE_OKTA_CLIENT_ID=your-client-id                 # Okta app client ID
VITE_OKTA_ISSUER=https://your-domain.okta.com/oauth2/default  # Token issuer
VITE_API_URL=http://localhost:3001                 # Backend API base URL
```

### Okta Application Setup Requirements
- Application type: Single-Page Application (SPA)
- Sign-in redirect URI: `http://localhost:5173/login/callback`
- Sign-out redirect URI: `http://localhost:5173`
- Grant types: Authorization Code with PKCE
- Trusted Origins: Configure `http://localhost:5173` for CORS

## Okta Realms and Assignments

This application creates **Realms** and **Realm Assignments** using Okta's APIs:

### Realms
Logical groupings that segment users by partner organization. Each realm:
- Has a unique realm ID
- Contains one or more domains associated with the partner organization
- Is typed as 'PARTNER' to indicate it's for a partner organization
- Provides user isolation and management capabilities

### Realm Assignments
Automatic rules that assign users to realms based on conditions. Each assignment:
- Has a name describing what it does (e.g., "Assign @acme.com usernames to Acme realm")
- Uses Okta Expression Language to match users (e.g., by email domain)
- Specifies which realm to assign matched users to
- Has a priority number (lower = higher priority)
- Must be activated to take effect

Example: For organization "Acme Corp" with domain "acme.com", the app creates:
1. A realm for Acme Corp
2. An assignment rule: "If user.login contains '@acme.com', assign to Acme realm"
3. Activates the rule so it automatically applies to new and existing users

## Development Workflow

1. Start backend server first: `cd backend && npm run dev`
2. Start frontend server: `cd frontend && npm run dev`
3. Access application at http://localhost:5173
4. Sign in with Okta admin credentials
5. Upload CSV file with partner organizations
6. View real-time creation results

## Common Tasks

### Adding a New API Endpoint
1. Define route in `backend/src/routes/realmRoutes.ts`
2. Add handler method to `backend/src/controllers/realmController.ts`
3. If needed, add business logic to `backend/src/services/oktaService.ts`
4. Create corresponding frontend service method in `frontend/src/services/apiService.ts`

### Adding a New Frontend Component
1. Create component in `frontend/src/components/`
2. Import Material UI components as needed
3. Use `useOktaAuth()` hook to access authentication state
4. Call API via `apiService` methods

### Debugging Okta API Calls
- Backend logs all realm creation attempts to console
- Check Okta Admin Dashboard → Reports → System Log for API call history
- Verify API token has necessary admin scopes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jamesdaly-dev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jamesdaly-dev)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
