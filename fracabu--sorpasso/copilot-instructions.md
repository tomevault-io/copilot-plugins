## sorpasso

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Il Sorpasso is a Vue 3 landing page for a luxury car rental service with integrated Firebase backend. The app features multilingual support (Italian/English), contact forms that send emails via Firebase Functions, and an admin panel for managing inquiries.

## Development Commands

**Frontend Development:**
- `npm run dev` - Start development server with hot reload
- `npm run build` - Build for production
- `npm run preview` - Preview production build locally

**Firebase Functions:**
- `cd functions && npm run build` - Compile TypeScript functions
- `cd functions && npm run build:watch` - Compile with watch mode for development
- `cd functions && npm run serve` - Start local emulator
- `cd functions && npm run deploy` - Deploy functions to Firebase
- `cd functions && npm run lint` - Lint functions code
- `cd functions && npm run logs` - View function logs

**Linting & Type Checking:**
- `npx vue-tsc --noEmit` - Check TypeScript types for Vue files
- `npx vue-tsc --noEmit --project tsconfig.app.json` - Check app-specific TypeScript configuration
- `cd functions && npm run lint` - Lint functions code

## Architecture

### Frontend Structure
- **Vue 3 + TypeScript** with Composition API
- **Vite** as build tool with `@` alias pointing to `src/`
- **Vue Router** with nested routes for admin panel and service pages
- **Vue I18n** for internationalization (Italian default, English fallback)
- **TailwindCSS** with custom theme (accent color: #DC2626)
- **AOS** for scroll animations
- **VueUse/Head** for meta tag management

### Key Directories
- `src/components/` - Reusable Vue components
- `src/views/` - Route-level components (Home, admin pages, service pages)
- `src/layouts/` - Layout wrapper components
- `src/i18n/locales/` - Translation JSON files
- `src/router/` - Vue Router configuration
- `functions/src/` - Firebase Cloud Functions

### Firebase Integration
- **Firestore** for storing contact form submissions
- **Firebase Auth** for admin authentication (restricted to specific email)
- **Cloud Functions** for sending contact emails via Nodemailer (Node.js 18 runtime)
- **Firebase Hosting** for SPA deployment with automatic routing
- Config loaded from environment variables (`VITE_FIREBASE_*`)

### Routing
- `/` - Home page with all sections
- `/admin/login` - Admin authentication
- `/admin/dashboard` - Protected admin panel
- `/admin/database-check` - View contact submissions
- `/admin/email-debug` - Test email functionality
- `/servizi/*` - Individual service detail pages

### Authentication
Admin routes are protected with Firebase Auth guard that only allows access to `fracabu@gmail.com`.

### Internationalization
- Default locale: Italian (`it`)
- Fallback locale: Italian
- Locale persistence in localStorage as `preferred-locale`
- Document language attribute updated automatically

### Styling
- TailwindCSS with custom configuration
- Custom fonts: Inter (sans), Playfair Display (serif) 
- Typography plugin enabled
- Custom accent color: #DC2626 (red)
- Responsive design with mobile-first approach

## Environment Variables Required
```
VITE_FIREBASE_API_KEY
VITE_FIREBASE_AUTH_DOMAIN
VITE_FIREBASE_PROJECT_ID
VITE_FIREBASE_STORAGE_BUCKET
VITE_FIREBASE_MESSAGING_SENDER_ID
VITE_FIREBASE_APP_ID
VITE_FIREBASE_MEASUREMENT_ID
```

## Deployment
Project supports multiple deployment options:

**Firebase Hosting:**
- Primary deployment target with automatic SPA routing via `firebase.json`
- Functions deployed separately to Firebase Cloud Functions with Node.js 18 runtime
- Use `firebase deploy` after building the project

**Vercel:**
- Alternative deployment option configured via `vercel.json`  
- SPA routing handled by rewrites to `/index.html`
- Automatic deployment from git repository

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fracabu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
