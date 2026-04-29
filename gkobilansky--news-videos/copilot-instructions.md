## news-videos

> This is a **Vertical Newsbite Generator** - a self-hosted Next.js application that converts headlines and source links into 10-15 second portrait videos with AI voice-over, captions, and b-roll footage. The app is designed to be local-first, outputting MP4 files to a local `/output` directory.

# Project Overview: Vertical Newsbite Generator

## Project Purpose
This is a **Vertical Newsbite Generator** - a self-hosted Next.js application that converts headlines and source links into 10-15 second portrait videos with AI voice-over, captions, and b-roll footage. The app is designed to be local-first, outputting MP4 files to a local `/output` directory.

## Tech Stack
- **Frontend/API**: Next.js 15 with App Router, React Server Components + Server Actions
- **Database**: Supabase Postgres (Docker) with migrations in [supabase/migrations/](mdc:supabase/migrations)
- **AI Services**: OpenAI (chat + TTS), Runway Gen-3 (video generation)
- **Video Processing**: Local ffmpeg for video assembly
- **Testing**: Jest + React Testing Library (161 tests, 13 test suites)
- **Storage**: Local filesystem with DB storing relative paths

## Core Architecture
- **Service Layer**: [src/services/](mdc:src/services) - Business logic and API integrations
- **Server Actions**: [src/app/actions/](mdc:src/app/actions) - Client-server communication
- **Components**: [src/components/](mdc:src/components) - Reusable React components
- **Types**: [src/types/index.ts](mdc:src/types/index.ts) - TypeScript definitions
- **Validation**: [src/lib/story-validation.ts](mdc:src/lib/story-validation.ts) - Input validation

## Development Status
- ✅ **User Story U1**: Story creation form ([/stories/new](mdc:src/app/stories/new/page.tsx))
- ✅ **User Story U2**: Script generation and editing ([/stories/[id]/script](mdc:src/app/stories/[id]/script/page.tsx))
- 🚧 **User Story U3**: Video generation pipeline (infrastructure complete, final integration in progress)

## Key Documentation
- [CLAUDE.md](mdc:CLAUDE.md) - Comprehensive development guidelines
- [README.md](mdc:README.md) - Project setup and architecture
- [package.json](mdc:package.json) - Dependencies and scripts

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gkobilansky) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
