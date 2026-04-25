## spotify-merge-playlist

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Nuxt.js application that allows users to merge multiple Spotify playlists and get music recommendations. It's a full-stack web application with a Vue.js frontend and Nuxt server API routes handling Spotify Web API integration.

## Development Commands

- **Start development server**: `npm run dev` (runs on http://localhost:3000)
- **Build for production**: `npm run build`
- **Preview production build**: `npm run preview`
- **Generate static site**: `npm run generate`
- **Install dependencies**: `npm install`

## Application Architecture

### Frontend Structure
- **Main App**: `app/app.vue` - Root component using Nuxt UI framework
- **Home Page**: `app/pages/index.vue` - Single-page interface for playlist merging
- **Styling**: Uses Nuxt UI components and Tailwind CSS via `app/assets/css/main.css`

### Backend API Routes
Located in `server/api/`:
- **`login.get.ts`** - Initiates Spotify OAuth flow with required scopes (playlist-read-private, playlist-modify-public, playlist-modify-private)
- **`callback.get.ts`** - Handles OAuth callback, exchanges code for tokens, sets HTTP-only cookies
- **`merge.post.ts`** - Core business logic that merges playlists and generates recommendations

### Key Dependencies
- **Nuxt 4**: Full-stack Vue.js framework
- **Nuxt UI**: Component library for UI elements
- **spotify-web-api-node**: Spotify Web API client
- **TypeScript**: Type safety throughout the application

## Authentication Flow
1. User clicks merge button without authentication → redirected to `/api/login`
2. Login endpoint redirects to Spotify OAuth with required scopes
3. Spotify redirects back to `/api/callback` with authorization code
4. Callback exchanges code for access/refresh tokens, stores in HTTP-only cookies
5. User returns to home page and can now merge playlists

## Playlist Merging Logic
The merge process in `server/api/merge.post.ts`:
1. Validates user has at least 2 playlist URLs
2. Extracts playlist IDs and fetches all tracks (filtering out podcasts)
3. Creates new playlist with combined name and description
4. Finds and adds duplicate tracks across all playlists
5. Uses non-duplicate tracks as seeds for Spotify recommendations (up to 5 seeds, fallback to 1)
6. Adds recommended tracks that aren't already in source playlists
7. Returns URL to the newly created merged playlist

## Environment Variables Required
- `SPOTIFY_CLIENT_ID` - Spotify app client ID
- `SPOTIFY_CLIENT_SECRET` - Spotify app client secret

## Frontend Features
- Real-time playlist URL validation and preview thumbnails via Spotify oEmbed API
- Loading states with elapsed time tracking during merge operations
- Toast notifications for user feedback
- Responsive design with Nuxt UI components
- Korean language interface

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/m1ns2o) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
