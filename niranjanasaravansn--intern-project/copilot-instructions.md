## intern-project

> This document outlines the architecture, setup, and deployment procedures for the Research Portal application (`research-mac`).

# Research Portal (GEMINI Project)

This document outlines the architecture, setup, and deployment procedures for the Research Portal application (`research-mac`).

## 1. Project Overview

The Research Portal is a full-stack web application designed to manage academic research activities, including:
*   **Journal Publications:** Tracking and management of research papers.
*   **Projects:** Monitoring funded projects, proposals, and grants.
*   **Events:** Management of academic events, finances, and pre-event planning.
*   **User Dashboard:** Personalized views for faculty and students.
*   **Admin Dashboard:** Centralized control for administrators.

## 2. Technology Stack

### Client (Frontend)
*   **Framework:** React 18
*   **Build Tool:** Vite
*   **Language:** TypeScript
*   **UI Library:** Bootstrap 5, React Bootstrap
*   **Icons:** Lucide React
*   **HTTP Client:** Axios
*   **Authentication:** Azure MSAL (Microsoft), Google OAuth (@react-oauth/google)
*   **Calendar:** FullCalendar

### Server (Backend)
*   **Runtime:** Node.js
*   **Framework:** Express.js
*   **Database:** MySQL (via `mysql2`)
*   **API Documentation:** Swagger UI
*   **File Handling:** Multer
*   **Module System:** ES Modules (`import`/`export`)

## 3. Directory Structure

```
research-mac/
├── client/                 # React (Vite + TypeScript) Application
│   ├── src/
│   │   ├── admin/          # Admin Dashboard modules
│   │   ├── components/     # Reusable UI components
│   │   ├── context/        # React Context (Auth, etc.)
│   │   ├── shared/         # Shared utilities (authenticityUtility.ts)
│   │   ├── styles/         # Global styles
│   │   ├── App.tsx         # Main Component
│   │   └── main.tsx        # Entry Point
│   ├── .env.development    # Local development config
│   ├── .env.production     # Production build config
│   ├── package.json
│   └── vite.config.ts
├── server/                 # Node.js Express Application
│   ├── config/             # DB connections & Swagger setup
│   ├── middleware/         # Auth & Error handling middleware
│   ├── modules/            # Feature-based modules (Controller/Routes/Models)
│   ├── schema/             # SQL Schema definitions
│   ├── uploads/            # File upload storage
│   ├── .env.development    # Local environment config
│   ├── .env.production     # Production environment config
│   ├── iisnode-start.cjs   # Entry point for IISNode
│   ├── server.js           # Main Express App entry
│   └── package.json
├── web.config              # IIS Configuration for the root
└── GEMINI.md               # This documentation
```

## 4. Configuration & Environment

The project uses **Split Environment Configuration** to support seamless transitions between local development and production.

### Client Configuration (`client/`)

*   **`.env.development`** (Loaded via `npm run dev`)
    ```ini
    VITE_API_URL=http://localhost:5000/api
    VITE_GOOGLE_CLIENT_ID=...
    VITE_MICROSOFT_CLIENT_ID=...
    VITE_MICROSOFT_TENANT_ID=...
    ```

*   **`.env.production`** (Loaded via `npm run build`)
    ```ini
    VITE_API_URL=https://research.ksrct.ac.in/api
    VITE_GOOGLE_CLIENT_ID=...
    VITE_MICROSOFT_CLIENT_ID=...
    VITE_MICROSOFT_TENANT_ID=...
    ```

### Server Configuration (`server/`)

The server determines the environment via the `NODE_ENV` variable.

*   **`.env.development`** (Loaded when `NODE_ENV` is missing or 'development')
    ```ini
    PORT=5000
    HOST=localhost
    USER=root
    PASSWORD=root
    DB=research
    FRONTEND_URL=http://localhost:5173
    ```

## 5. Development Setup (Local)

1.  **Prerequisites:** Node.js (v18+), MySQL.
2.  **Database:** Import the schema from `server/schema` into your local MySQL instance.

### Running the Server
```bash
cd server
npm install
npm run dev
```
*   Server runs on: `http://localhost:5000`
*   Swagger Docs: `http://localhost:5000/api-docs`

### Running the Client
```bash
cd client
npm install
npm run dev
```
*   Client runs on: `http://localhost:5173` (Vite default)

## 6. Key Utilities

### Authenticity Utility (`authenticityUtility.ts`)
Located in `client/src/shared/utils/`. This utility dynamically resolves the Base URL for API requests.
*   **Development:** Returns `http://localhost:5000`
*   **Production:** Returns `https://research.ksrct.ac.in` (derived from `VITE_API_URL`).

## 8. Quality Assurance (SonarQube)

Code quality is tracked using SonarQube.

*   **Server URL:** `http://localhost:9000`
*   **Project Key:** `Research`
*   **Configuration:** `sonar-project.properties` (at root)

To run analysis manually:
```bash
# Ensure you are in the root directory
sonar
```

## 9. Database Schema
Major tables include:
*   `users`: Authentication and profile info.
*   `journals`: Research publication details.
*   `projects`: Funded project details.
*   `events`: Event management data.
*   `finance`: Budget and expense tracking for events.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Niranjanasaravansn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
