## bgs-sensor-data-snapshots

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BGS Sensor Network Dashboard built with Next.js 15.4.5, TypeScript, and shadcn/ui components. Provides real-time monitoring of BGS's environmental sensor network using the BGS FROST API.

## Commands

- `npm run dev --turbopack` - Start development server with turbopack (fast refresh)
- `npm run dev -- -p 4000` - Start development server on port 4000  
- `npm run build` - Build for production
- `npm run start` - Start production server
- `npm run lint` - Run ESLint

## Architecture

### Tech Stack
- **Next.js 15.4.5** with App Router
- **React 19** with TypeScript 5
- **shadcn/ui** components with stone theme
- **Tailwind CSS v4**
- **BGS FROST API** for sensor data

### Key Components
```
src/
├── app/
│   └── sensor/[sensorId]/page.tsx     # Full sensor page with charts
├── components/
│   ├── ui/                            # shadcn/ui components
│   └── dashboard/
│       ├── BGSDashboard.tsx           # Main dashboard
│       ├── SensorTable.tsx            # Sensor list with filters
│       ├── SensorDetailSheet.tsx      # Sidebar with datastream details
│       ├── DatastreamChart.tsx        # Chart component (normalised by default)
│       └── DatastreamSummary.tsx      # Statistics cards
├── lib/
│   ├── api-config.ts                  # API base URL configuration
│   ├── bgs-api.ts                     # BGS FROST API functions
│   ├── chart-utils.ts                 # Simple chart utilities
│   └── date-utils.ts                  # Date formatting utilities
└── types/
    └── bgs-sensor.ts                  # TypeScript interfaces
```

## Key Features

### Dashboard
- **Sensor Table** - All sensors with search/filter by type, location, measurement
- **Summary Cards** - Total sensors, locations, sites, datastreams
- **Manual Refresh** - User-controlled data updates with visual feedback

### Chart System (Latest - Production Ready)
- **500 Latest Readings Default** - Shows most recent data on first load (no date constraints)
- **Normalised by Default** - 0-1 scaling for comparing different value ranges
- **Raw/Normalised Toggle** - Switch between actual values and normalised view
- **15 Unique Colours** - No colour conflicts between datastreams
- **Adaptive Y-Axis** - 5% padding for better min/max value readability
- **Smart Time Display** - X-axis shows time + date based on data range
- **Dynamic Date Inputs** - Update to reflect actual data range being displayed
- **Any Number of Datastreams** - No hardcoded limits

### Sensor Details
- **Full Sensor Pages** - `/sensor/[sensorId]` with URL routing
- **Interactive Date Pickers** - Auto-update to show actual data range
- **500 Latest Readings** - Default view shows most recent sensor data
- **Custom Date Ranges** - Up to 1000 readings for specific time periods
- **Location & Maps** - GPS coordinates with Leaflet integration
- **Depth Information** - Z-coordinate display showing sensor depth/elevation with coordinate reference system
- **Detail Sheets** - Sidebar with:
  - Chart visualization
  - Summary statistics
  - **Clickable datastream list** - Select to see latest 10 observations
  - Borehole reference extraction
  - Prominent depth badge in Sensor Overview section

## Data Integration

### API Configuration
- **Centralized Config** - `src/lib/api-config.ts` for easy API switching
- **Public API** - `https://sensors.bgs.ac.uk/FROST-Server/v1.1` (current)
- **Internal API** - Available for BGS internal use

### Sensor Data
- **4 Major Sites** - UKGEOS Glasgow, BGS Cardiff, UKGEOS Cheshire, Wallingford
- **200+ Sensors** - Groundwater, weather, soil gas monitoring
- **Gas Measurements** - CO₂, CH₄, O₂ from GasClam probes
- **Environmental Data** - Temperature, pressure, humidity, conductivity
- **Real-time Updates** - Direct FROST API integration

## Performance & Optimization

### Latest Optimizations (August 2025)
- **Bundle Size** - 30% reduction: 19.1kB main page, 26.3kB sensor page
- **Code Cleanup** - 300+ lines of dead code removed
- **Centralized Functions** - API calls consolidated in `bgs-api.ts`
- **Memoized Dependencies** - Prevents unnecessary re-renders
- **TypeScript Compliance** - Zero compilation errors

### Data Limits
- **Default View** - 500 most recent readings (no date constraints)
- **Custom Ranges** - Up to 1000 readings for selected date periods
- **Optimized Loading** - Fast initial load, detailed exploration available
- **No Artificial Limits** - Fetches all available sensors and datastreams

## Styling & UI

- **shadcn/ui Components** - Always use first before custom components
- **Stone Theme** - Consistent with BGS branding
- **Dark Mode** - System-aware with localStorage persistence
- **UK Localisation** - "Normalised" spelling, en-GB formatting
- **Responsive Design** - Works on mobile and desktop

## Development Guidelines

### Code Style
- **Minimal & Clean** - No over-engineering
- **Single Responsibility** - Components focus on one task
- **Centralized Logic** - Common functions in `/lib`
- **Type Safety** - Strict TypeScript configuration
- **Performance First** - Memoization and optimized dependencies

### API Usage
- **Error Handling** - Graceful fallbacks for API failures
- **Caching** - Intelligent caching with appropriate TTL
- **Rate Limiting** - Considerate API usage patterns
- **Scientific Validation** - Data integrity functions for research accuracy

## Recent Updates

### Data Quality Improvements (August 2025)
- **Event Datastream Filtering** - Non-measurement datastreams excluded from visualizations
- **Enhanced Property Names** - Preserves important qualifiers:
  - Water Level (linear) vs Water Level (polynomial) 
  - Vibrating Wire Pressure vs regular Pressure
- **Improved Map Zoom** - Sensor location maps show closer detail (zoom level 16+)
- **UI Polish** - Cursor pointer on interactive elements for better UX
- **Depth/Z-Coordinate Integration** - Sensor depth information extracted from location properties and prominently displayed

### Chart System Optimization
- **500 Latest Readings Default** - Fast loading with most relevant recent data
- **Dynamic Date Inputs** - Automatically reflect actual data range shown
- **Enhanced Time Display** - X-axis shows time + date based on data span
- **Optimized Chart Performance** - Single-pass data processing with validation
- **Simplified API Logic** - 500 for default view, 1000 for custom ranges
- **Error Handling** - Robust date validation prevents chart failures

### Data Processing Functions
- `filterNonEventDatastreams()` - Removes information-only datastreams from charts
- `getPropertyName()` - Intelligent datastream name parsing with qualifier preservation
- Scientific validation functions ensure data integrity for research accuracy

### Location & Depth Enhancement (August 2025)
- **Z-Coordinate Support** - Location interface includes optional `z` (depth) and `z_crs` (coordinate reference system) fields
- **API Integration** - Depth data automatically extracted from FROST API location properties during existing API calls
- **No Performance Impact** - Uses existing API responses, no additional HTTP requests required
- **Prominent Display** - Depth shown as badge in Sensor Overview section for high visibility
- **Smart Formatting** - Displays as "Depth: -6.888m maOD" with proper units and reference system
- **Responsive Design** - Consistent badge styling across both detail sheet and full sensor page views

### Production Deployment
- GitLab CI/CD pipeline with automated deployment
- Kubernetes deployment to BGS internal clusters
- Multi-stage Docker build with security scanning
- Environment-specific deployments (dev/staging/production)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pautva) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
