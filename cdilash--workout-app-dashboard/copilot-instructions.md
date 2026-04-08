## workout-app-dashboard

> A web-based dashboard for advanced visualization of workout data, designed to complement the local-first React Native mobile app. The dashboard ingests JSON exports and provides deep analytics on training volume, intensity, and balance.

# Workout App Dashboard - Project Guidelines

## Overview
A web-based dashboard for advanced visualization of workout data, designed to complement the local-first React Native mobile app. The dashboard ingests JSON exports and provides deep analytics on training volume, intensity, and balance.

## Tech Stack
- **Framework**: Next.js 16 (App Router)
- **Language**: TypeScript (strict)
- **Styling**: Tailwind CSS + Shadcn/ui principles
- **Charts**: Recharts (Responsive, Animated)
- **State Management**: Zustand + Persistence (LocalStorage + LZ-String)
- **Validation**: Zod (Strict Schema V2.0)

## Features (Implemented)

### 1. Data Ingestion & Architecture
- **Drag-and-Drop Import**: Supports Version 1.0 (Legacy) and 2.0 (Metadata-rich) exports.
- **Auto-Migration**: Automatically upgrades legacy data formats on ingest.
- **Sanitization**: Strips HTML/Scripts from user notes.
- **Persistence**: Saves data to `localStorage` with **LZ-String compression** to bypass 5MB limits.

### 2. Analytics Engine (Read-Time)
- **Unit Conversion**: Global toggle for **kg** / **lbs** across all metrics.
- **Volume & Load**: Absolute tonnage tracking.
- **Hard Sets**: Counts sets with RPE ≥ 7 or RIR ≤ 3.
- **Estimated 1RM**: Epley formula calculation for all exercises.

### 3. Visualizations
- **Fatigue Management**: Dual-axis chart (Volume vs. Avg RPE) to detect overreaching.
- **Program Balance**: Stacked bar chart showing **Push/Pull/Legs** split and **Compound/Isolation** ratio.
- **Strength Progression**: Line chart tracking e1RM over time.
- **Rep Ranges**: Histogram of training zones (Strength vs. Hypertrophy).
- **Muscle Radar**: Visualization of volume distribution by muscle group.
- **Heatmap**: 100-day activity frequency grid.

### 4. Intelligence & Reporting
- **Coach's Report**: Automated narrative summary of the last 7 days (Adherence, Focus, Trend).
- **PR Detection**: Highlights new All-Time Bests (Weight, Volume, e1RM).
- **Year in Lift**: "Spotify Wrapped" style summary card for social sharing.
- **Raw Data Export**: Buttons to generate flat CSVs (`sets.csv`, `workouts.csv`) for external analysis.

## Project Structure

```
src/
├── app/                  # Next.js App Router
├── components/           # React Components
│   ├── charts/           # Recharts visualizations (Fatigue, Balance, etc.)
│   ├── dashboard/        # Widgets (CoachReport, PRs, YearInLift)
│   └── ui/               # Base UI
├── lib/
│   ├── utils.ts          # Styles helpers
│   ├── types.ts          # Canonical Data Types (V2.0)
│   ├── stats.ts          # Analytics Engine (Heavy computation)
│   ├── parsers.ts        # Zod Validation & Migration
│   └── export.ts         # CSV Generation logic
├── stores/               # Zustand stores
└── styles/               # Global styles
```

## Development Guidelines
- **Strict Typing**: No `any`. Zod schemas act as the source of truth for external data.
- **Performance**: Heavy stats are memoized using `useMemo`.
- **Theme**: Dark mode supported via Tailwind classes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cDilash)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cDilash)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
