## parallel-spreadsheet

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Next.js 15 spreadsheet application that provides AI-powered data enrichment using Parallel AI's web research capabilities. Users can select cells in a spreadsheet interface and enrich them with live data from the web.

## Essential Commands

```bash
# Development
npm run dev          # Start development server with Turbopack (http://localhost:3000)

# Production  
npm run build        # Build production application
npm start            # Start production server

# Code Quality
npm run lint         # Run ESLint for code linting
```

## Environment Setup

Create `.env.local` with:
```
PARALLEL_API_KEY=your_api_key_here
```

## Architecture

### Tech Stack
- **Next.js 15.4.6** with App Router
- **React 19.1.0** 
- **TypeScript** with strict mode
- **Tailwind CSS v4** for styling
- **react-spreadsheet** for spreadsheet component

### Key Components

1. **Main Spreadsheet UI** (`app/page.tsx`): 
   - Client-side React component managing spreadsheet state
   - Handles cell selection, enrichment requests, and real-time updates via SSE
   - Keyboard shortcuts for productivity (⌘K, ⌘J, ⌘↵)

2. **Parallel API Route** (`app/api/parallel/route.ts`):
   - POST: Creates task groups and runs for selected cells
   - GET: Proxies Server-Sent Events from Parallel API
   - DELETE: Handles client-side cancellation

### Data Flow

1. User selects cells and clicks "Enrich"
2. Frontend POSTs to `/api/parallel` with row context and target columns
3. API creates Parallel task group with structured prompts and JSON schema
4. SSE connection established to stream results
5. Results populate cells in real-time with visual feedback

### Parallel AI Integration

The app uses Parallel AI's task group API for web research:
- **Processors**: lite (basic), base (simple), core (complex), pro (exploratory)
- **Structured Output**: JSON schema enforces exact column matching
- **Streaming**: SSE provides real-time progress updates

---
> Source: [zahidkhawaja/parallel-spreadsheet](https://github.com/zahidkhawaja/parallel-spreadsheet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
