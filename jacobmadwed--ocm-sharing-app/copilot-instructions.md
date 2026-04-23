## ocm-sharing-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**Frontend Development:**
- `pnpm dev` - Start Vite development server on port 1420
- `pnpm build` - Build the frontend for production
- `pnpm serve` - Preview the production build

**Tauri Desktop App:**
- `pnpm tauri dev` - Run the desktop app in development mode (builds frontend first)
- `pnpm tauri build` - Build the desktop app for production

**Convex Backend:**
- `npx convex dev` - Start Convex development server for real-time backend
- `npx convex deploy` - Deploy backend functions to production

## Architecture Overview

This is a **Tauri + SolidJS + Convex** desktop application for image sharing with email/SMS capabilities.

**Tech Stack:**
- **Frontend**: SolidJS with TypeScript, Tailwind CSS
- **Desktop**: Tauri (Rust-based) for native desktop app
- **Backend**: Convex for real-time database and file storage
- **Styling**: Tailwind CSS v4 with Radix UI components
- **Communication**: SendGrid (email), Twilio (SMS), UploadThing (file uploads)

**Key Architecture Patterns:**
- **SolidJS Signals**: State management using `createSignal()` for reactive updates
- **Convex Mutations/Queries**: Backend operations through `convex/` directory
- **Component Structure**: UI components in `src/components/` with shadcn/ui pattern
- **Tauri Integration**: Desktop file system access through Tauri plugins

**Core Data Flow:**
1. **File Selection**: AdminGear component handles folder selection via Tauri dialog
2. **Image Processing**: ImageGrid watches selected folder and displays images
3. **Upload Pipeline**: Images uploaded to Convex storage, metadata stored in database
4. **Sharing**: Email/SMS modals use respective services to share image links

**Database Schema (Convex):**
- `images` table: storageId, filename, contentType, size, uploadedAt
- Indexed by upload time for chronological display

**File System Access:**
- Tauri configured with broad filesystem permissions: HOME, DOWNLOADS, DOCUMENTS, PICTURES
- Asset protocol enabled for local file access

**Environment Setup:**
- Requires `VITE_CONVEX_URL` environment variable
- SendGrid and Twilio API keys needed for email/SMS functionality

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jacobmadwed) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
