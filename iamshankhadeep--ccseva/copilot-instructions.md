## ccseva

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CCSeva is a macOS menu bar Electron application that monitors Claude Code usage in real-time. The app uses the `ccusage` npm package API to fetch token usage data and displays it through a modern React-based UI with tabbed navigation, analytics, notifications, and visualizations.

## Essential Commands

### Development
```bash
npm run electron-dev  # Start with hot reload (recommended for development)
npm run dev           # Build frontend only in watch mode
npm start            # Start built app
```

### Building
```bash
npm run build        # Production build (webpack + tsc compilation)
npm run pack         # Package app with electron-builder
npm run dist         # Build and create distribution package
npm run dist:mac     # Build for macOS specifically
```

### Code Quality
```bash
npm run lint         # Run Biome linter
npm run lint:fix     # Fix linting issues automatically
npm run format       # Format code with Biome
npm run format:check # Check code formatting
npm run check        # Run linting and formatting checks
npm run check:fix    # Fix linting and formatting issues
npm run type-check   # TypeScript type checking without emit
```

### Dependencies
```bash
npm install          # Install all dependencies
```

## Architecture Overview

### Dual-Process Electron Architecture
The app follows standard Electron patterns with clear separation:

- **Main Process** (`main.ts`): Manages system tray, IPC, and background services
- **Renderer Process** (`src/`): React app handling UI and user interactions
- **Preload Script** (`preload.ts`): Secure bridge exposing `electronAPI` to renderer

### Key Architectural Components

#### Service Layer (Singleton Pattern)
- **CCUsageService**: Uses the `ccusage` npm package data-loader API to fetch usage data, implementing a 30-second cache. Now supports plan configuration and actual session-based reset times.
- **SettingsService**: Manages user preferences persistence to `~/.ccseva/settings.json` including plan selection, custom token limits, timezone, and reset hour settings
- **NotificationService**: Manages macOS notifications with cooldown periods and threshold detection
- **ResetTimeService**: Handles Claude usage reset time calculations and timezone management
- **SessionTracker**: Tracks user sessions and activity patterns for analytics

#### Data Flow
1. Main process polls CCUsageService every 30 seconds
2. Service imports `loadSessionBlockData` and `loadDailyUsageData` from `ccusage/data-loader` to fetch usage data
3. The returned JavaScript objects are mapped to typed interfaces (`UsageStats`, `MenuBarData`)
4. Menu bar updates with percentage display, renderer receives data via IPC
5. React app renders tabbed interface with dashboard, analytics, and live monitoring views
6. NotificationService triggers alerts based on usage thresholds and patterns

#### Modern UI Component Architecture
```
App.tsx (main container with state management)
├── NavigationTabs (tabbed interface)
├── Dashboard (overview with stats cards)
├── LiveMonitoring (real-time usage tracking)
├── Analytics (charts and historical data)
├── TerminalView (command-line interface simulation)
├── SettingsPanel (user preferences)
├── LoadingScreen (app initialization)
├── ErrorBoundary (error handling)
├── NotificationSystem (toast notifications)
└── ui/ (Radix UI components)
    ├── Button, Card, Progress, Tabs
    ├── Alert, Badge, Tooltip, Switch
    └── Avatar, Popover, Select, Slider
```

### Build System Specifics

#### Dual Compilation Process
The build requires both Webpack (renderer) and TypeScript compiler (main/preload):
```bash
webpack --mode production && tsc main.ts preload.ts --outDir dist
```

#### Critical Path Dependencies
- **ccusage npm package**: Direct dependency providing data-loader API functions
- **Tailwind CSS v3**: PostCSS processing with custom gradient themes
- **React 19**: Uses new JSX transform (`react-jsx`)
- **Radix UI**: Component library for accessible UI primitives
- **Biome**: Fast linter and formatter replacing ESLint/Prettier

### IPC Communication Pattern

Main process exposes these handlers:
- `get-usage-stats`: Returns full UsageStats object
- `refresh-data`: Forces cache refresh and returns fresh data
- `usage-updated`: Event emitted to renderer every 30 seconds

Renderer accesses via `window.electronAPI` (type-safe interface in preload.ts).

## Data Processing Logic

### Usage Calculation
The app detects Claude plans automatically:
- **Pro**: ≤7,000 tokens
- **Max5**: ≤35,000 tokens  
- **Max20**: ≤140,000 tokens
- **Custom**: >140,000 tokens

### Burn Rate Algorithm
Calculates tokens/hour based on last 24 hours of usage data, used for depletion time predictions.

### Error Handling Strategy
- CCUsageService returns default stats on ccusage command failures
- React components display error states with retry buttons
- Main process continues functioning even if data fetch fails

## Development Considerations

### TypeScript Configuration
Uses strict mode with custom path aliases (`@/*` → `src/*`). Three separate tsconfig files:
- `tsconfig.json`: Main renderer process configuration
- `tsconfig.main.json`: Main Electron process configuration  
- `tsconfig.preload.json`: Preload script configuration

### Modern UI Architecture
- **Tailwind CSS v3**: Custom color palette for Claude branding with glass morphism effects
- **Radix UI Components**: Accessible, unstyled primitives for complex components
- **Sonner**: Toast notification system for user feedback
- **Lucide React**: Icon library for consistent iconography
- **Class Variance Authority**: Type-safe component variant management

### Menu Bar Integration
macOS-specific Tray API with text-only display (no icon). Features contextual menus and window positioning near menu bar with auto-hide behavior.

### Advanced Notification System
Implements intelligent notification logic:
- 5-minute cooldown between notifications
- Progressive alerts (70% warning → 90% critical) 
- Only notifies when status worsens, not repeated warnings
- Toast notifications within app for immediate feedback

## Required External Dependencies

- **`ccusage` npm package**: This is a direct dependency managed in `package.json`.
- **Claude Code**: Must be configured with valid credentials in `~/.claude` directory containing JSONL usage files, which the `ccusage` package uses as its data source.
- **macOS**: Tray and notification APIs are platform-specific

## Code Quality and Development Workflow

### Biome Configuration
The project uses Biome for linting and formatting with these key settings:
- **Import organization**: Automatically sorts and organizes imports
- **Strict linting**: Warns on `any` types, enforces import types, security rules
- **Consistent formatting**: 2-space indentation, single quotes for JS, double quotes for JSX
- **Line width**: 100 characters maximum

### ccusage Integration Best Practices

When using the `ccusage` package data-loader API:

1. **Use data-loader functions**: Import `loadSessionBlockData` and `loadDailyUsageData` from `ccusage/data-loader`
2. **Handle structured data**: The API returns typed JavaScript objects, no JSON parsing needed
3. **Separate data calls**: Make separate API calls for session and daily data to optimize performance
4. **Robust error handling**: Implement `try/catch` blocks around API calls to handle missing `~/.claude` configuration
5. **Caching strategy**: Implement 30-second caching to avoid excessive file system reads

## Recent Updates and Improvements

### Settings Management & Plan Selection (Latest)
- **Claude Plan Settings**: Added comprehensive plan selection in SettingsPanel with Auto-detect, Pro, Max5, Max20, and Custom options
- **Persistent Settings**: Extended SettingsService to save plan preferences to `~/.ccseva/settings.json` with backward compatibility
- **Custom Token Limits**: Custom plan option allows users to set non-standard token limits with validation
- **Real-time Plan Display**: TerminalView now shows selected plan settings instead of just auto-detected plans
- **Settings UI Enhancement**: Professional plan selection dropdown with token limit display and current plan detection

### Session-Based Reset Time Accuracy
- **Active Session Integration**: Reset time now uses actual `activeBlock.endTime` from session data instead of estimated monthly cycles
- **Real-time Countdown**: SettingsPanel displays live countdown showing "X hours Y minutes left" updating every minute
- **Simplified Logic**: Removed complex fallback calculations, shows "No active session" when appropriate
- **Dashboard Integration**: Updated Dashboard to use actual session-based reset times consistently

### Cost Calculation Improvements
- **Enhanced Average Cost**: Fixed Analytics average cost per 1000 tokens calculation with better edge case handling
- **Data Validation**: Added checks for both `totalTokens > 0 AND totalCost > 0` to prevent division by zero
- **Accurate Pricing**: Formula `(totalCost / totalTokens) * 1000` now properly validated for real-world cost accuracy

### ccusage Integration Refactor
- **Switched from CLI to API**: Refactored `CCUsageService` to use the `ccusage` npm package directly, replacing `child_process` calls.
- **Simplified data fetching**: API calls (`loadSessionBlockData`, `loadDailyUsageData`) now return structured JS objects, removing the need for manual JSON parsing and field name mapping.
- **Improved reliability**: Direct API integration is more robust and less prone to issues from shell environment differences.
- **Dependency management**: `ccusage` is now a formal npm dependency in `package.json`, ensuring version consistency.

### Current Project Structure
```
ccseva/
├── main.ts                     # Electron main process with tray management
├── preload.ts                  # Secure IPC bridge
├── src/
│   ├── App.tsx                 # Main React container with state management
│   ├── components/             # Modern UI components
│   │   ├── Dashboard.tsx       # Overview with stats cards
│   │   ├── Analytics.tsx       # Charts and historical data
│   │   ├── LiveMonitoring.tsx  # Real-time usage tracking
│   │   ├── TerminalView.tsx    # CLI simulation interface
│   │   ├── SettingsPanel.tsx   # User preferences
│   │   ├── NavigationTabs.tsx  # Tabbed interface
│   │   ├── NotificationSystem.tsx # Toast notifications
│   │   ├── LoadingScreen.tsx   # App initialization
│   │   ├── ErrorBoundary.tsx   # Error handling
│   │   └── ui/                 # Radix UI components
│   ├── services/               # Business logic services
│   │   ├── ccusageService.ts   # ccusage data-loader integration
│   │   ├── settingsService.ts  # User preferences persistence
│   │   ├── notificationService.ts # macOS notification management
│   │   ├── resetTimeService.ts # Reset time calculations
│   │   └── sessionTracker.ts   # Session tracking
│   ├── types/
│   │   ├── usage.ts            # TypeScript interfaces
│   │   └── electron.d.ts       # Electron API types
│   ├── lib/utils.ts            # Utility functions
│   └── styles/index.css        # Tailwind CSS with custom themes
├── biome.json                  # Biome linter/formatter config
├── components.json             # Radix UI component config
├── electron-builder.json       # App packaging configuration
├── webpack.config.js           # Renderer build configuration
├── tsconfig*.json              # TypeScript configurations (3 files)
├── tailwind.config.js          # Tailwind CSS configuration
└── postcss.config.js           # PostCSS configuration
```

### Git Repository Status
- **Initialized git repository** with comprehensive .gitignore
- **Two commits made**:
  1. Initial commit with full feature set
  2. Refactor commit improving ccusage integration
- **Clean working tree** ready for development

## Testing and Verification

Since there are no automated tests, manual verification checklist:

### Core Functionality
1. Menu bar text display appears with usage percentage
2. Click expands tabbed interface with multiple views
3. Right-click shows context menu with refresh/quit options
4. All tabs (Dashboard, Live, Analytics, Terminal, Settings) function correctly
5. Data updates every 30 seconds across all views
6. Error boundaries handle failures gracefully

### Data Integration
7. **ccusage data-loader integration**: Verify correct import and usage of data-loader functions
8. **Data consistency**: Ensure displayed data matches `ccusage` output
9. **Actual reset time accuracy**: Verify session-based reset times from active blocks
10. **Session tracking**: Confirm session data persistence and analytics
11. **Settings persistence**: Confirm plan and preference settings save to `~/.ccseva/settings.json`

### Plan Management & Settings
12. **Plan selection**: Test Auto-detect, Pro, Max5, Max20, and Custom plan options in SettingsPanel
13. **Custom token limits**: Verify custom plan allows setting and validation of non-standard limits
14. **Real-time updates**: Confirm plan changes immediately update Dashboard and TerminalView displays
15. **Settings persistence**: Verify settings survive app restarts and maintain backward compatibility

### UI/UX Features
16. **Toast notifications**: In-app notifications work properly
17. **macOS notifications**: System alerts appear at thresholds
18. **Real-time countdown**: SettingsPanel shows live "X hours Y minutes left" updating every minute
19. **Plan display consistency**: TerminalView shows selected plan settings (not just auto-detected)
20. **Cost calculation accuracy**: Analytics shows correct average cost per 1000 tokens
21. **Theme consistency**: Tailwind styling renders correctly
22. **Responsive design**: Interface adapts to different window sizes
23. **Component interactions**: All Radix UI components function properly

---
> Source: [Iamshankhadeep/ccseva](https://github.com/Iamshankhadeep/ccseva) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
