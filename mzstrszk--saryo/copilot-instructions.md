## saryo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

"茶寮 (Saryo)" is a Japanese green tea information portal website. This is a full-stack Next.js application for managing and displaying green tea farm and tea information across Japan. 

The project includes:
- **Admin features**: Farm management, tea information management, article management with authentication
- **Public features**: Farm search, map exploration, article reading, advanced search functionality
- **Map integration**: Leaflet-based mapping showing farm locations across Japan
- **Responsive design**: Green and beige color scheme optimized for mobile

## Architecture

This is a **Next.js 15 App Router** application with:
- **Frontend**: React Server Components, TypeScript, Tailwind CSS, Leaflet maps
- **Backend**: Server Actions, API Routes, Prisma ORM, NextAuth.js authentication
- **Database**: PostgreSQL with PostGIS extension for geospatial data
- **Deployment**: Vercel with Vercel Postgres and Vercel Blob for image storage

## Project Structure

Since this is a new project, the expected structure will be:

```
src/
├── app/                    # Next.js App Router
│   ├── (admin)/           # Admin panel (farms, teas, articles management)
│   ├── farms/             # Farm search and details
│   ├── map/               # National map view
│   ├── articles/          # Article browsing
│   └── search/            # Advanced search
├── components/            # Shared components
│   ├── ui/               # UI components
│   ├── forms/            # Form components  
│   ├── maps/             # Map-related components
│   └── layout/           # Layout components
├── lib/                  # Utilities and database
└── types/                # TypeScript definitions
```

## Key Data Models

- **Farms**: Japanese tea farms with location data, images, and descriptions
- **Teas**: Tea varieties linked to farms with tags (cultivation period, grade, etc.)
- **Articles**: Blog posts about green tea, linked to specific teas/farms
- **Tags**: Both tea tags and article tags for categorization
- **Images**: Multiple images per farm/tea with ordering and optimization

## Development Commands

This is a new project with no existing build configuration. When implementing, typical Next.js commands would be:

```bash
# Development
npm run dev          # Start development server

# Building
npm run build        # Build for production
npm run start        # Start production server

# Code quality
npm run lint         # ESLint
npm run type-check   # TypeScript checking

# Testing (when implemented)
npm test            # Unit tests
npm run test:e2e    # E2E tests with Playwright
```

## Key Implementation Notes

1. **Authentication**: Admin-only access for content management, NextAuth.js implementation
2. **Geolocation**: PostgreSQL with PostGIS for location queries and map functionality
3. **Image Management**: Vercel Blob integration with image optimization and multiple image types per entity
4. **Search**: Full-text search across farms, teas, and articles with tag-based filtering
5. **Maps**: Leaflet integration showing farm pins with clustering for performance
6. **Responsive Design**: Green/beige theme with mobile-first approach

## Database Schema Highlights

- **One-to-many**: Farms → Teas, Farms → Images, Teas → Images  
- **Many-to-many**: Teas ↔ Tags, Articles ↔ Tags, Articles ↔ Teas
- **Geospatial**: Latitude/longitude fields for farm locations
- **Image ordering**: Display order field for image galleries

## Performance Considerations

- Server Components for data fetching
- Image optimization with Next.js Image component
- Map clustering for 1000+ farm pins
- Pagination for search results and article lists
- Lazy loading for image galleries

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mzstrszk)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mzstrszk)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
