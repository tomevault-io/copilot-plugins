## tomate-undesayuno

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Core Development

```bash
npm run dev                # Start both Firebase emulator and Astro dev server concurrently
npm run dev:astro          # Start Astro dev server only (waits for emulator on port 9000)
npm run dev:no-emulator    # Start Astro dev server with production Firebase
npm run emulator           # Start Firebase Realtime Database emulator only
npm run emulator:ui        # Start Firebase emulator with UI
npm run build              # Build production site to ./dist/
npm run preview            # Preview build locally
npm run generate-firebase-config  # Regenerate Firebase config from .env
```

### Accessing the Application

- **Main App**: `http://localhost:4321/` - Group breakfast ordering
- **Admin Panel**: `http://localhost:4321/admin` - Admin dashboard (requires Google authentication)
- **Emulator UI**: `http://127.0.0.1:4000` - Firebase emulator dashboard (when using `npm run emulator:ui`)

## Architecture Overview

### Tech Stack

- **Frontend**: Astro (v5.12.9) with Lit web components (v3.3.1)
- **Backend**: Firebase Realtime Database
- **Build System**: Astro static site generation
- **Development**: Firebase emulator for local development

### Application Type

Breakfast order management system for group ordering with menu selection and person management. Includes a secure admin panel for managing all groups with Google authentication.

### Project Structure

```
public/
├── js/
│   ├── components/       # Lit web components
│   │   ├── app-controller.js      # Main application controller
│   │   ├── admin-app-controller.js # Admin panel controller
│   │   ├── admin-login.js         # Admin login component
│   │   ├── admin-panel.js         # Admin dashboard component
│   │   ├── grupo-selector.js      # Group selector component
│   │   ├── grupo-resumen.js       # Group summary component
│   │   ├── menu-selector.js       # Menu selector component
│   │   ├── modal-system.js        # Modal system manager
│   │   ├── personas-manager.js    # Person management
│   │   └── personas-manager-grupo.js # Group person management
│   ├── services/         # Business logic services
│   │   ├── firebase-service.js    # Firebase initialization and operations
│   │   ├── auth-service.js        # Authentication service (Google OAuth)
│   │   ├── grupo-service.js       # Group management service
│   │   └── session-service.js     # Session management
│   ├── data/
│   │   └── menu-data.js           # Menu items data
│   └── utils/           # Utility functions
├── css/
│   └── global.css       # Global styles
└── images/
    └── tomatelogo.png   # Application logo

src/
├── pages/               # Astro pages
│   ├── index.astro     # Main entry page
│   └── admin.astro     # Admin panel page
├── layouts/            # Page layouts
├── components/         # Astro components
└── firebase/
    └── firebase-config.js  # Firebase configuration
```

### Key Components

#### Main Application
- **AppController** (`/public/js/components/app-controller.js`) - Main application orchestrator, manages views and navigation
- **GrupoSelector** (`/public/js/components/grupo-selector.js`) - Group creation and joining interface
- **PersonasManagerGrupo** (`/public/js/components/personas-manager-grupo.js`) - Group member and order management

#### Admin Panel
- **AdminAppController** (`/public/js/components/admin-app-controller.js`) - Admin panel controller with authentication flow
- **AdminLogin** (`/public/js/components/admin-login.js`) - Google OAuth login interface
- **AdminPanel** (`/public/js/components/admin-panel.js`) - Dashboard for managing all groups

#### Services
- **FirebaseService** (`/public/js/services/firebase-service.js`) - Centralized Firebase operations
- **AuthService** (`/public/js/services/auth-service.js`) - Google authentication and authorization
- **GrupoService** (`/public/js/services/grupo-service.js`) - Group management logic
- **SessionService** (`/public/js/services/session-service.js`) - User session management
- **ModalSystem** (`/public/js/components/modal-system.js`) - Modal dialog management

### Development Setup

1. **Environment Configuration**:
   - Copy `.env` file with Firebase credentials (never commit this file)
   - Firebase config is auto-generated at `/public/js/firebase-config.js` during dev/build
   - Generated config file is excluded from git but exposed in production

2. **Firebase Emulator**:
   - Realtime Database runs on port 9000
   - Emulator UI available on port 4000
   - Database rules defined in `database.rules.json`

3. **Astro Development**:
   - Development server runs on `http://localhost:4321`
   - Static build outputs to `dist/` directory
   - Firebase hosting configured to serve from `dist/`

### Firebase Configuration

- **Hosting**: Serves from `dist/` directory (NOT `public/`)
- **Database Rules**: Defined in `database.rules.json`
  - `desayunos`: Public read/write for group data
  - `authorizedUsers`: Authenticated read, no write (managed manually)
  - `adminAccessLogs`: Authenticated read/write for access logging
- **Emulator Ports**:
  - Database: 9000
  - UI: 4000
- **Authentication**: Google OAuth provider for admin panel access

### Lit Components Architecture

All components use Lit Element v3 with:
- ES modules loaded from CDN
- Reactive properties with decorators
- CSS-in-JS styling using `css` template literals
- Event-driven communication between components

### Development Workflow

1. **Initial Setup**:
   ```bash
   # 1. Install dependencies
   npm install
   
   # 2. Create .env file with Firebase credentials
   cp .env.example .env  # Edit with your Firebase config
   ```

2. **Start Development**:
   ```bash
   npm run dev  # Generates Firebase config + starts emulator and Astro dev server
   ```

3. **Build for Production**:
   ```bash
   npm run build  # Generates Firebase config + creates production build in dist/
   ```

4. **Preview Production Build**:
   ```bash
   npm run preview  # Preview the production build locally
   ```

### Build Process

The build process automatically:
1. Reads environment variables from `.env`
2. Generates `/public/js/firebase-config.js` with configuration and helper functions
3. Builds the Astro application
4. The generated config file is served publicly but excluded from git

### Important Notes

- Always use `dist` directory for Firebase hosting deployment
- Firebase emulator must be running for local development
- The app uses concurrent execution to run both emulator and dev server
- Uses `wait-on` to ensure emulator is ready before starting Astro
- Admin panel requires Google authentication and email must be in `authorizedUsers` list
- Firebase configuration is loaded from environment variables (never commit these)
- Database rules restrict admin-only paths to authenticated users

### Security Considerations

- **Authentication**: Admin panel uses Google OAuth for authentication
- **Authorization**: Only emails in `/authorizedUsers` database path can access admin panel
- **Access Logging**: All admin access attempts are logged to `/adminAccessLogs`
- **Configuration Security**: 
  - Firebase credentials stored in `.env` file (never committed)
  - Build script generates `/public/js/firebase-config.js` from environment variables
  - Generated config file excluded from git but served in production
  - Environment variables required:
    - `PUBLIC_FIREBASE_API_KEY`
    - `PUBLIC_FIREBASE_AUTH_DOMAIN`
    - `PUBLIC_FIREBASE_DATABASE_URL`
    - `PUBLIC_FIREBASE_PROJECT_ID`
    - `PUBLIC_FIREBASE_STORAGE_BUCKET`
    - `PUBLIC_FIREBASE_MESSAGING_SENDER_ID`
    - `PUBLIC_FIREBASE_APP_ID`

### Admin Panel Setup

1. **Add authorized users to database**:
   - In Firebase Console or Emulator UI
   - Navigate to `/authorizedUsers`
   - Add email addresses as array values

2. **Access admin panel**:
   - Navigate to `/admin`
   - Click "Sign in with Google"
   - Use an authorized email account

3. **Admin capabilities**:
   - View all active groups
   - See group statistics (members, totals, activity)
   - Mark groups as paid
   - Delete groups
   - View detailed orders per person

---
> Source: [manufosela/tomate-undesayuno](https://github.com/manufosela/tomate-undesayuno) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
