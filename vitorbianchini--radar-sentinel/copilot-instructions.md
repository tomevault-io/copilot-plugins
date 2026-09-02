## radar-sentinel

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Radar Sentinel is a real-time GPS-based radar monitoring system built with React and TypeScript. It alerts users when approaching speed cameras and traffic enforcement devices based on their location, heading, and speed. Originally created via AI Studio, it uses Vite for development.

## Development Commands

```bash
# Install dependencies
npm install

# Run development server (port 3000)
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview
```

## Core Architecture

### Alert Detection Logic (Dual-Vector System)

The app uses a sophisticated two-step detection system in `App.tsx`:

1. **Vector Check (lines 140-142)**: Determines if the user is moving *towards* the radar coordinate by comparing user heading against bearing to radar using `isHeadingMatching(userHeading, bearingToRadar, DIRECTION_TOLERANCE_DEGREES)`

2. **Orientation Check (lines 146-159)**: Validates if the user's heading matches the radar's monitoring angle based on `dirType`:
   - `dirType: 0` (Omnidirectional): Always matches if moving towards it
   - `dirType: 1` (Unidirectional): User heading must match radar's `direction` within tolerance
   - `dirType: 2` (Bidirectional): User heading must match radar's `direction` OR opposite direction (direction + 180°)

This prevents false alerts when passing radar locations without being in their monitored direction.

### Data Flow

1. **File Upload** (`handleFileUpload`, App.tsx:30): Parses Maparadar TXT format with columns: `X, Y, TYPE, SPEED, DirType, Direction`
2. **GPS Tracking** (`startTracking`, App.tsx:80): Uses `navigator.geolocation.watchPosition` with high accuracy
3. **Alert Detection** (App.tsx:119-174): useEffect that runs on location/radar changes, applying dual-vector logic
4. **Audio Alerts** (App.tsx:177-189): Triggered based on speed comparison with rate limiting (3.5s intervals)

### Key Constants (App.tsx:8-11)

- `ALERT_DISTANCE_METERS = 800`: Distance threshold to trigger alerts
- `DIRECTION_TOLERANCE_DEGREES = 35`: Angular tolerance for heading matching
- `SOUND_REPEAT_INTERVAL_MS = 3500`: Minimum time between audio alerts
- `SPEED_THRESHOLD_KMH = 5`: Minimum speed to activate monitoring

### Utilities

- **geoUtils.ts**: Haversine distance calculation, bearing computation, heading matching with circular angle normalization
- **audioUtils.ts**: Web Audio API beeps (square wave for overspeed, sine wave for approach)
- **RadarIcon.tsx**: SVG icons based on Maparadar type codes (1=Fixed, 2=Mobile, 3=Traffic Light, 5=Barrier, 6=Police)

### Type System (types.ts)

- `Radar`: Core entity with lat/lng, type, speed limit, direction, and dirType
- `UserLocation`: GPS data with speed in km/h and optional heading
- `RadarAlert`: Active alert state with distance and radar reference
- `AppState`: State machine (INITIAL → LOADING_FILE → MONITORING → ERROR)

## Maparadar File Format

Tab/comma/semicolon-separated text file:
```
longitude, latitude, type, speed_limit, dirType, direction
-46.6333, -23.5505, 1, 60, 1, 180
```

- Types: 1=Fixed, 2=Mobile, 3=Traffic Light, 5=Barrier, 6=Police
- DirType: 0=Omni, 1=Uni, 2=Bi
- Direction: 0-359 degrees (compass heading)

## Testing Considerations

- Requires GPS/geolocation access (mobile device or browser geolocation spoofing)
- Test with various `dirType` values to verify directional filtering
- Verify heading calculations work correctly at 0°/360° boundary
- Test speed threshold and distance calculations with mock locations

---
> Source: [vitorbianchini/radar-sentinel](https://github.com/vitorbianchini/radar-sentinel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
