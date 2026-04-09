## alex-landing

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Structure

This is a full-stack personal trainer landing page with two main parts:
- **Backend**: Directus CMS (Headless CMS) in `/backend` directory with Docker
- **Frontend**: React SPA with Vite, TypeScript, and shadcn/ui in `/frontend` directory

The project creates a landing page for Alexander Paskhalis, a personal fitness trainer, with contact information and service descriptions.

## Development Commands

### Backend (Directus CMS)
```bash
cd backend
docker-compose up -d    # Start Directus with Docker
docker-compose down     # Stop Directus
docker-compose logs     # View logs
```

### Frontend (React/Vite)
```bash
cd frontend
npm run dev        # Development server
npm run build      # Production build
npm run lint       # ESLint
npm run preview    # Preview production build
```

## Architecture Overview

### Backend Architecture
- **Directus v10.10.0** headless CMS with Docker
- **Collections**:
  - `contacts`: Stores contact information (telegram, instagram, whatsapp, phone, email, main photo)
  - `services_blocks`: Service descriptions with title, rich text, and multiple photos
- **Database**: SQLite
- **API Structure**: RESTful endpoints at `/items/{collection}`

### Frontend Architecture
- **React 18** with TypeScript
- **Vite** for build tooling
- **React Router** for routing (currently just Index and NotFound pages)
- **shadcn/ui** component library with Radix UI primitives
- **Tailwind CSS** for styling
- **TanStack Query** for data fetching
- **Component Structure**:
  - `pages/Index.tsx`: Main landing page with hero, about, services, contacts sections
  - `components/ServiceSection.tsx`: Reusable service display component
  - `components/ServiceCarousel.tsx`: Image carousel for services
  - `components/SocialIcons.tsx`: Social media icons component
  - `components/ui/`: shadcn/ui components

### Key Features
- Responsive design with mobile-first approach
- Hero section with contact info overlay
- Services sections with alternating image layouts
- Social media integration (Instagram, Telegram, WhatsApp)
- Smooth scrolling navigation

## Content Management
- Content is managed through Directus admin panel (accessible at http://localhost:1337 when backend is running)
- Dynamic content from Directus CMS with fallback to hardcoded defaults
- Images are stored in `/backend/uploads/` with automatic optimization

## API Integration
- Frontend fetches dynamic content from Directus backend
- **API Configuration**: Located in `/frontend/src/lib/api.ts`
- **API Services**: Located in `/frontend/src/lib/directusApi.ts`
- **Contact Data**: Includes email, phone, social media logins, main photo, greeting text, and about info
- **Services Data**: Dynamic service blocks with title, rich text content, and multiple photos with video support

## Development Notes  
- Frontend uses Russian language content
- Content is now dynamic from Directus CMS (falls back to hardcoded defaults if API fails)
- Backend runs in Docker container on localhost:1337 for full functionality
- Uses React Query for API state management with loading states
- Supports both images and videos in service sections
- No authentication system implemented for public content
- No form submissions - contact via social media/phone only

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TeaProphet)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TeaProphet)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
