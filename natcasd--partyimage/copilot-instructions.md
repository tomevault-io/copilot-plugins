## partyimage

> PartyImage is an interactive web application designed to create engaging visual experiences for parties and events. The application enables real-time AI-generated image displays based on partygoers' prompts.

# PartyImage Project Overview

PartyImage is an interactive web application designed to create engaging visual experiences for parties and events. The application enables real-time AI-generated image displays based on partygoers' prompts.

## Core Features

### User Management
- Host users can create accounts and manage their API keys
- Secure authentication system for party hosts
- API key management for AI image generation services

### Session Management
- Party hosts can create unique sessions for their events
- Each session generates a unique QR code
- Session URLs are shareable with party attendees

### Image Generation System
- Real-time AI image generation from partygoer prompts
- Queue-based system for processing prompts in order
- Live display of generated images on the party screen

### Data Structure
The application is built around three main data models:
1. **Users**
   - Authentication credentials
   - API key management
   - Session ownership

2. **Sessions**
   - Unique session identifiers
   - QR code generation
   - Host association
   - Active/inactive status

3. **Image Batches**
   - Generated images storage
   - Session association
   - Queue position tracking
   - Processing status

## User Flow
1. Party host creates an account and logs in
2. Host creates a new session, generating a unique QR code
3. Partygoers scan the QR code to access the prompt submission interface
4. Submitted prompts enter a queue for processing
5. AI generates images based on the prompts
6. Generated images are displayed in real-time on the party screen

## Technical Architecture
- Frontend: Interactive web interface for both hosts and partygoers
- Backend: Session management and image processing system
- Queue System: Manages prompt processing order
- AI Integration: Connects to image generation APIs
- Real-time Updates: Live display of generated images

## Project Structure

### Frontend (Next.js)
```
apps/frontend/
├── src/
│   ├── app/                 # Next.js App Router
│   ├── components/          # Reusable React components
│   ├── lib/                 # Utility functions and Supabase client
│   ├── types/              # TypeScript type definitions
│   ├── hooks/              # Custom React hooks
│   └── styles/             # Additional styling files
├── public/                 # Static assets
└── [config files]          # Next.js, TypeScript, etc.
```

### Backend (Supabase)
```
supabase/
├── sql/                    # SQL migrations and queries
├── functions/              # Edge functions
├── migrations/             # Database migrations
├── seed/                   # Seed data
└── types/                  # Generated database types
```

This project aims to create an engaging and interactive experience for party hosts and attendees through AI-generated visual content.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/natcasd)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/natcasd)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
