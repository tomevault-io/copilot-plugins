## ai-workshop

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an AI Workshop Landing Page with registration functionality, built as a full-stack application with React frontend and Express backend. The app allows users to register for an AI workshop and sends confirmation emails via SendGrid.

## Key Commands

### Development
- `npm run dev` - Start frontend development server (Vite)
- `npm run server` - Start backend server with hot reload
- `npm run dev:all` - Run both frontend and backend concurrently
- `npm run build` - Build production frontend assets

### Deployment
- `docker build -t ai-workshop-landing .` - Build Docker image
- `gcloud builds submit --config cloudbuild.yaml` - Deploy to Google Cloud Run

## Architecture

### Frontend (React + Vite)
- Entry point: `index.tsx` renders `App.tsx`
- Main app state managed in `App.tsx` with registration flow
- Components in `/components/` directory for modular UI
- Email service client in `/services/emailService.ts`
- Types defined in `types.ts`

### Backend (Express + TypeScript)
- Entry point: `server/index.ts` 
- API routes in `server/api/` directory
- Email service implementation in `server/emailService.ts` using SendGrid
- CORS configured for multiple allowed origins including production domains

### Email System
- Uses SendGrid for email delivery (replaced previous Gmail/Nodemailer setup)
- Sends two emails per registration: admin notification and user confirmation
- Email templates defined in `server/emailService.ts`

### Environment Configuration
- Frontend: Uses `VITE_API_URL` for API endpoint configuration
- Backend: Requires `SENDGRID_API_KEY` and `GMAIL_USER` (admin email) environment variables
- Production API URL: `https://ai-workshop-711582759542.us-central1.run.app`
- Local development typically runs on port 3001

### Deployment
- Dockerized application serving both frontend static files and backend API
- Google Cloud Run deployment configured via `cloudbuild.yaml`
- Static files served from `/dist` directory after Vite build
- SPA routing handled by serving `index.html` for all non-API routes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jsong1004) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
