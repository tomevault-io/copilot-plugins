## medirat

> **Document Purpose:** This document provides a comprehensive technical overview of the "MEDiRAT®" application. It is intended to be used as a context file for Large Language Models like Gemini 2.5 Pro to assist in future development, maintenance, deployment, and backend integration tasks.


# MEDiRAT®: Technical Brief for Gemini AI

**Document Purpose:** This document provides a comprehensive technical overview of the "MEDiRAT®" application. It is intended to be used as a context file for Large Language Models like Gemini 2.5 Pro to assist in future development, maintenance, deployment, and backend integration tasks.

---

## 1. Project Overview

### 1.1. Application Name
MEDiRAT®

### 1.2. Core Purpose
A web-based Risk Assessment and Maintenance Management system designed for medical equipment within a hospital or clinical setting.

### 1.3. Key Features
- **Centralized Dashboard:** Provides an at-a-glance overview of critical metrics, including overdue services, high-risk equipment, and upcoming maintenance tasks.
- **Equipment Inventory Management:** A searchable, filterable list of all medical equipment with detailed views for each item.
- **Risk Assessment Module:** Implements the **Risk Priority Number (RPN)** methodology. Technicians can log assessments by providing scores for Likelihood, Severity, and Detectability.
- **Service Logging:** A complete history of maintenance activities for each piece of equipment.
- **AI-Powered Note Generation:** Integrates with the **Google Gemini API** to automatically generate professional service log notes from simple keywords.
- **Role-Based Access Control (RBAC):** A three-tier user system (`Service Manager`, `Technician`, `Hospital Staff`) with distinct permissions for viewing and modifying data.
- **Notification System:** A frontend simulation of an in-app alert system for services due within 24 hours.
- **Personalization:** Features user-specific profile pictures, a collapsible sidebar, and a persistent light/dark theme.

---

## 2. Technology Stack

### 2.1. Frontend
- **Framework:** React 18+ with TypeScript
- **Styling:** Tailwind CSS (using `class` strategy for dark mode)
- **State Management:** React Context API for global state (`AuthContext`, `ThemeContext`). Component-level state is managed with `useState` and `useMemo`.
- **Charting Library:** Recharts
- **Build Tool (for production):** Vite

### 2.2. AI Integration
- **Service:** Google Gemini API
- **SDK:** `@google/genai`
- **Model:** `gemini-2.5-flash` for fast and efficient text generation.

### 2.3. Proposed Backend & Deployment
- **Backend Service:** Supabase
- **Database:** PostgreSQL
- **Authentication:** Supabase Auth (JWT-based)
- **Deployment Platform:** Google Cloud Run
- **Containerization:** Docker
- **Web Server:** Nginx (serving static files in production)

---

## 3. Application Architecture & Structure

### 3.1. Directory Structure
```
/
|-- public/
|-- src/
|   |-- components/      # Reusable React components (UI elements)
|   |   |-- icons/       # SVG icon components
|   |-- contexts/        # React Context providers (AuthContext, ThemeContext)
|   |-- hooks/           # Custom React hooks (useAuth, useTheme)
|   |-- services/        # Logic for external interactions (API calls, mock data)
|   |-- types/           # TypeScript type definitions
|   |-- App.tsx          # Main application component, layout, and routing
|   |-- index.css        # Main CSS file with Tailwind directives
|   |-- index.tsx        # Application entry point
|-- .dockerignore
|-- Dockerfile
|-- GEMINI.md            # This file
|-- index.html           # HTML root file
|-- nginx.conf           # Nginx configuration for production
|-- package.json
|-- postcss.config.js
|-- tailwind.config.js
|-- tsconfig.json
|-- vite.config.ts
```

### 3.2. Core Logic & State Management
- **`App.tsx`:** Acts as the main controller. It manages the current view (e.g., dashboard, list), selected equipment, notifications, and sidebar state. It conditionally renders the `LoginScreen` or the main `AppContent` based on authentication status.
- **`AuthContext.tsx`:** Manages the current user's session. It handles login/logout, persists user data to `localStorage`, and provides a `hasPermission` function used for RBAC throughout the app. It's designed to be replaced by Supabase Auth.
- **`ThemeContext.tsx`:** Manages the `light` or `dark` theme, persisting the user's choice to `localStorage` and applying the appropriate class to the `<html>` element.

---

## 4. Data Model (`types.ts`)

### 4.1. Main Interfaces
- **`Equipment`**: Represents a single piece of medical equipment. Contains all its properties, including nested arrays for `maintenanceHistory` and `riskAssessments`.
- **`MaintenanceLog`**: A record of a single service event.
- **`RiskAssessment`**: A record of a risk assessment, including the L, S, D scores and the calculated RPN.
- **`User`**: Represents a user, including their role, permissions, and avatar URL.

### 4.2. Enums
- **`Department`**: Defines all possible hospital departments.
- **`EquipmentStatus`**: Defines the operational status of equipment.
- **`RiskLevel`**: Defines the severity categories based on RPN score.
- **`UserRole`**: Defines the three user roles in the system.

### 4.3. Risk Priority Number (RPN)
The RPN is a core concept calculated as:
`RPN = Likelihood * Severity * Detectability`
The result is then mapped to a `RiskLevel` using the `getRiskLevelFromRPN` function in `constants.ts`.

---

## 5. Key Features Implementation

### 5.1. Authentication & RBAC
- **Roles:** `Service Manager` (full access), `Technician` (can perform services), `Hospital Staff` (view-only).
- **Permissions:** A granular list of permissions (e.g., `add:equipment`, `acknowledge:notification`) is stored in the `User` object.
- **`hasPermission(permission: string)`:** This function from `useAuth` is the primary mechanism for enforcing access control. UI elements like buttons and navigation links are conditionally rendered based on its return value.

### 5.2. AI Note Generation (`geminiService.ts`)
- The service initializes the `GoogleGenAI` client using an API key from `process.env.API_KEY`.
- The `generateServiceNotes` function takes user-provided keywords.
- It constructs a specific prompt instructing the `gemini-2.5-flash` model to generate a professional service log note.
- It includes a `try...catch` block for robust error handling.

### 5.3. Deployment & Production Build (`Dockerfile`, `vite.config.ts`)
- **Build Process:** The application uses Vite to transpile TypeScript/React and bundle it into optimized static HTML, CSS, and JavaScript files in a `/dist` directory.
- **Containerization (`Dockerfile`):**
    - **Stage 1 (Builder):** Uses a `node` image to install dependencies (`npm ci`) and run the Vite build (`npm run build`). The `API_KEY` is passed as a build argument and embedded into the static files via Vite's `define` config.
    - **Stage 2 (Production):** Uses a lightweight `nginx:alpine` image. It copies the built static files from the builder stage and the custom `nginx.conf`.
- **Nginx Configuration (`nginx.conf`):**
    - Serves the static files from `/usr/share/nginx/html`.
    - Includes a `try_files` directive to redirect all client-side routes to `index.html`, which is standard for Single-Page Applications (SPAs).
- **Deployment to Google Cloud Run:** The final Docker image is pushed to Google Artifact Registry and then deployed as a new Cloud Run service. The `API_KEY` must be set as a runtime environment variable in the Cloud Run service configuration for the Gemini integration to work.

### 5.4. Supabase Integration Plan
- The `supabase_integration_guide.txt` file provides the full plan.
- **Database Schema:** SQL scripts are provided to create custom PostgreSQL types (enums), tables (`profiles`, `equipment`, etc.), and set up relationships.
- **Security:** Row Level Security (RLS) policies are defined to restrict data access based on the authenticated user's role and ID.
- **Frontend Refactoring:**
    1.  Replace `mockData.ts` with API calls to Supabase using `useEffect` hooks.
    2.  Refactor `AuthContext` to use `supabase.auth` for user management instead of `localStorage`.
    3.  Create a Supabase client in `services/supabaseClient.ts`.
    4.  Generate TypeScript types from the database schema for full type safety.

This document should provide sufficient context for any future AI-assisted development on the MEDiRAT®.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cofiemark)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cofiemark)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
