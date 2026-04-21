## shadytimes

> **Goal**: Build a new tide information application for Shady Island access planning in Steveston, Richmond, BC

# Shady Island Tides - Application Specification

## Project Overview

**Goal**: Build a new tide information application for Shady Island access planning in Steveston, Richmond, BC

**Context**: Shady Island is accessed via a weir exposed only at low tide (~1.1m or less). This app helps users plan island access by displaying tide predictions, observations, and access windows.

**Target Users**: Small group (you and friends) using primarily mobile devices, often while boating in bright outdoor conditions

**Prototype Reference**: [ShadyTides2.html](ShadyTides2.html) - Located in `/prototypes/` directory, kept for reference

---

## Phase 1 - Core Application ✅ COMPLETED

**Status**: Implemented and released as v1.0.0

### Purpose & Key Functionality

**Primary Use Case**: Determine if and when the island is accessible

**Key Questions to Answer**:
- Can I access the island right now?
- When are the next low tide access windows?
- How long will each access window last?

**Success Criteria**: Users can quickly see upcoming low tide windows with current-day observations

### Technical Stack

- **Frontend**: HTML, CSS, JavaScript (can span multiple files)
- **No frameworks**: Keep it simple like the prototype
- **Repository**: Use current directory (`/home/robbie/ShadyTimes`)
- **Project Structure**:
  - Move `ShadyTides2.html` to `/prototypes/` directory
  - New app files in root or `/src/` (best practice structure)
- **Deployment**: GitHub Pages, single environment

### Data Source

**DFO (Department of Fisheries and Oceans) API**
- **Station ID**: `5cebf1e13d0f4a073c4bbf8c` (Steveston BC)
- **Predictions Endpoint**: `https://api-iwls.dfo-mpo.gc.ca/api/v1/stations/5cebf1e13d0f4a073c4bbf8c/data?time-series-code=wlp`
- **Observations Endpoint**: `https://api-iwls.dfo-mpo.gc.ca/api/v1/stations/5cebf1e13d0f4a073c4bbf8c/data?time-series-code=wlo`

### Core Features (From Prototype)

- Date navigation (previous/next day buttons + date picker)
- Display data in 15-minute intervals
- Show both predictions and observations
- Color-coded tide levels
- Show observations only for current/past dates

### New Features (Phase 1)

**Status & Indicators**:
- **Current Status Banner**: "Island is accessible NOW" or "Island is NOT accessible" indicator
- **Countdown Timer**: Time until next access window opens OR time until current window closes
- **Tide Trend**: Rising or falling indicator with arrow/icon

**Tide Information**:
- **High/Low Tide Times**: Display prominently at top of day view (daily high and low tide times)
- **Access Windows**: Show time ranges when tide is ≤1.1m (accessible)
- **Real-Time Header**: Header always displays current status for TODAY, regardless of date being viewed

### Design & User Experience

**Device Priority**:
- Mobile-first design
- Desktop should look acceptable (not primary focus)
- Optimize for viewing in bright outdoor conditions (high contrast)

**Layout**:
- Single-page application
- One-page display: status info at top, tide table below
- Full 24-hour tide schedule display with smart auto-scrolling (current time for today, 10am for other dates)

**Visual Style**:
- Clean and minimalist
- Beautiful and modern aesthetic
- Professional appearance

**Color Scheme**:
- **Main theme**: Any appropriate color, let designer choose
- **Tide Level Colors**: Auto-gradient based on the window (number of rows) and tide levels.
- **Alternating Rows**: Use different shades only if needed for readability

### Data Display Specifications

**Tide Levels**:
- **Critical Threshold**:
≤1.1m is accessible - display in green gradient (darker green for lower/safer tides)
1.1-1.5m is marginally accessible - display in orange zone
>1.5m is not accessible - gradient from orange to white-blue for highest tides
- **Color Coding**: Auto-gradient-based, colors only (no icons)
- **Legend**: Display at bottom of table/page, indicating green gradient = accessible

**Time & Date**:
- **Time Format**: 12-hour (e.g., 3:00 PM)
- **Time Intervals**: 15-minute intervals
- **Timezone**: Pacific Time (PT/PDT) - automatically handle daylight saving
- **Current Time Highlight**: Bold outline or distinct background color
- **Date Format**: "Day, DDth of MM YYYY" (e.g., "Monday, 3rd of February 2026")
- **Default View**: Today's date

**Units**:
- **Height Units**: Meters (default) with toggle for feet
- **Precision**: Display in cm when meters selected, inches when feet selected
- **Unit Toggle**: Positioned at bottom with legend

**Date Navigation**:
- Previous/next day buttons
- Date picker for jumping to specific date
- Default to today on app open

### Performance & Data Management

**Loading States**:
- Spinner during data fetch
- Simple loading animation

**Error Handling**:
- Error popup/modal when API fails
- Clear error message with retry option

**Caching Strategy**:
- **Predictions**: Cache all day (they don't change)
- **Observations**: Cache for 15 minutes (subject to updates)
- **Per-Day Caching**: Each day can be cached independently

**Refresh**:
- No auto-refresh in Phase 1
- Manual refresh button/pull-to-refresh acceptable

**Browser Support**:
- Chrome minimum (primary target)
- Modern browsers (last 2 versions)

### File Organization

```
/home/robbie/ShadyTimes/
├── prototypes/
│   └── ShadyTides2.html          # Original prototype (reference only)
├── index.html                     # Main app entry point
├── styles/
│   └── main.css                   # Styles
├── scripts/
│   ├── api.js                     # API calls and caching
│   ├── tides.js                   # Tide calculations and logic
│   └── ui.js                      # UI updates and interactions
└── README.md                      # Project documentation
```

---

## Phase 2 - Future Enhancements

**Note**: Keep these in mind while building Phase 1 to avoid architectural decisions that would block future features

### Multi-Location Support
- Add Sand Heads Lighthouse station
- Location switching UI
- Default location: Shady Island
- Store user preference

### Advanced Visualizations
- Multi-day view (show multiple days at once)
- Tide chart/graph showing 24-hour curve
- Monthly view with high-level summary
- Weekly view with condensed information
- Different time intervals (hourly, etc.)

### Integrations
- Sunrise/sunset times (via external API or calculation)
- Weather data (temperature, conditions, wind)
- Moon phase display
- Historical comparison ("same time yesterday")

### User Features
- Notifications/alerts for specific tide conditions
- Share feature (share tide info with friends)
- Print-friendly view
- Bookmarks for favorite times/dates

### Technical Improvements
- Offline support (PWA with cookies)
- Pre-fetching adjacent days
- Switch to web framework (React/Vue/Svelte)
- Build custom backend service
- SQL database for data management
- Custom API layer

### Performance
- Auto-refresh capability
- Background data fetching

---

## Implementation Checklist

### Setup
- [ ] Create project structure (folders, files)
- [ ] Move ShadyTides2.html to prototypes directory
- [ ] Set up index.html with basic structure

### Core Data & API
- [ ] Implement DFO API client with error handling
- [ ] Add caching logic (predictions: all day, observations: 15min)
- [ ] Parse and process tide data
- [ ] Calculate high/low tide times from data
- [ ] Determine access windows (tide ≤1.1m)
- [ ] Implement tide trend calculation (rising/falling)

### UI Components
- [ ] Date navigation (prev/next + date picker)
- [ ] Current status banner ("accessible NOW" indicator)
- [ ] Countdown timer component
- [ ] Tide trend indicator (rising/falling arrow)
- [ ] Daily high/low tide times display
- [ ] Access windows display
- [ ] Tide table (15-min intervals)
- [ ] Color gradient for tide levels
- [ ] Current time highlighting in table
- [ ] Legend at bottom
- [ ] Unit toggle (meters/feet)
- [ ] Loading spinner
- [ ] Error modal/popup

### Styling
- [ ] Mobile-first responsive design
- [ ] High contrast for bright conditions
- [ ] Clean, minimalist, modern aesthetic
- [ ] Choose appropriate main theme color (designer's choice)
- [ ] Implement auto-gradient color system per specifications
- [ ] Optimize display for 10am-10pm time range

### Testing
- [ ] Test on mobile devices
- [ ] Test in bright outdoor conditions
- [ ] Test date navigation
- [ ] Test unit toggle
- [ ] Test error states
- [ ] Test caching behavior
- [ ] Test with different dates (past, present, future)
- [ ] Verify DST handling

### Deployment
- [ ] Set up GitHub Pages
- [ ] Configure deployment
- [ ] Test production build

---

## Notes

- Keep architecture flexible for Phase 2 multi-location support
- Prioritize mobile performance and readability
- Test in actual usage conditions (on boat, bright sunlight)
- Iterate based on real-world usage feedback
- DFO API is public and stable (no authentication required)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Robbie-Perry) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
