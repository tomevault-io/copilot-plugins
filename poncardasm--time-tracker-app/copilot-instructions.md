## time-tracker-app

> Generates a Svelte Playground link with the provided code.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a minimalist Progressive Web App (PWA) for time tracking built with Svelte 5. It's a client-side-only application with no backend - all data is stored in browser localStorage. The app supports both stopwatch and Pomodoro timer modes, with real-time tracking, manual time entry, project categorization, dark mode support, and CSV export functionality.

## Tech Stack

- **Build Tool**: Vite 6.x (latest stable)
- **Frontend Framework**: Svelte 5.0.0
- **Styling**: Tailwind CSS 4.1.0 (PostCSS processed)
- **PWA**: Service Worker with network-first caching strategy
- **Data Persistence**: Browser localStorage

## Architecture

### Core Application Flow

1. **State Management**: The app uses Svelte 5 runes ($state, $derived, $effect, $props) for reactive state management, organized into three store modules:

   - `stores/tasks.svelte.js` - Task data and CRUD operations (src/stores/tasks.svelte.js)
   - `stores/timer.svelte.js` - Active task timer, elapsed time, and Pomodoro state (src/stores/timer.svelte.js)
   - `stores/theme.svelte.js` - Dark mode state and persistence (src/stores/theme.svelte.js)

2. **Data Storage** (src/stores/tasks.svelte.js:1-33): All task records are stored in localStorage under the key 'timeTrackerTasks' as JSON. The tasks store exports functions for CRUD operations.

3. **Timer System** (src/stores/timer.svelte.js:24-62): Uses `setInterval` for live tracking with two modes:
   - **Stopwatch mode**: Calculates elapsed time by comparing current time with `startTime`
   - **Pomodoro mode**: Counts down from 25 minutes (work) and 5 minutes (break), with automatic phase switching and browser notifications

### Key Components

- **App.svelte** (src/App.svelte): Root component managing application-level state including modal visibility, modal modes, and event handlers
- **Header.svelte**: App header with dark mode toggle button
- **StartView.svelte**: Initial view with "Start Task" and "Add Manual Entry" buttons
- **ActiveView.svelte**: Active timer display showing elapsed time (stopwatch) or countdown (Pomodoro), with stop button
- **TaskModal.svelte** (src/components/TaskModal.svelte): Tri-modal design supporting 'start', 'manual', and 'edit' modes with project autocomplete dropdown
- **DeleteModal.svelte**: Confirmation modal for bulk task deletion
- **HistoryList.svelte**: Task history table with inline edit buttons, checkbox selection, and CSV export
- **Theme System** (src/stores/theme.svelte.js): Dark mode with localStorage persistence and system preference detection

### Component Structure

```
src/
├── App.svelte              # Root component, manages modals and event handlers
├── main.js                 # Entry point, mounts Svelte app
├── main.css                # Tailwind CSS imports and custom styles
├── utils.js                # Utility functions (formatTime, toLocalISOString)
├── stores/
│   ├── tasks.svelte.js     # Task data store with CRUD operations
│   ├── timer.svelte.js     # Timer state and Pomodoro logic
│   └── theme.svelte.js     # Dark mode state management
└── components/
    ├── Header.svelte       # Header with dark mode toggle
    ├── StartView.svelte    # Start/manual entry buttons
    ├── ActiveView.svelte   # Active timer display
    ├── TaskModal.svelte    # Task creation/editing modal
    ├── DeleteModal.svelte  # Delete confirmation modal
    └── HistoryList.svelte  # Task history table and export
```

### Data Structure

Task records follow this schema:

```javascript
{
    taskName: string,      // User-provided description
    project: string,       // Optional project category (nullable)
    startTime: number,     // Unix timestamp (ms)
    endTime: number,       // Unix timestamp (ms)
    durationMs: number     // Calculated duration in milliseconds
}
```

Active task state (during tracking):

```javascript
{
    description: string,   // Task description
    project: string,       // Optional project category
    startTime: number,     // Unix timestamp when started
    mode: string          // 'stopwatch' or 'pomodoro'
}
```

## Development Workflow

### Installing Dependencies

First, install the required dependencies:

```bash
npm install
```

### Running the Application

#### Development Mode

Start the Vite development server with hot module replacement:

```bash
npm run dev
```

The app will automatically open at <http://localhost:8000>. Changes to source files will be reflected immediately thanks to Svelte's HMR.

#### Production Build

Build the optimized production bundle:

```bash
npm run build
```

Output will be in the `dist/` directory.

#### Preview Production Build

Preview the production build locally:

```bash
npm run preview
```

### Testing PWA Functionality

1. Serve over HTTPS (required for service workers) or use localhost
2. Open DevTools → Application → Service Workers to monitor SW lifecycle
3. Test offline: DevTools → Network → Set to "Offline" mode

### Service Worker Cache Updates

When modifying source files or other cached resources:

1. Increment the `CACHE_NAME` version in public/sw.js (currently 'time-tracker-v4')
2. The SW update mechanism will automatically reload the page when a new SW is installed

### Project Structure

```
time-tracker-app/
├── index.html              # Main HTML file
├── src/
│   ├── App.svelte         # Root Svelte component
│   ├── main.js            # Application entry point
│   ├── main.css           # Tailwind CSS imports and custom styles
│   ├── utils.js           # Utility functions
│   ├── stores/
│   │   ├── tasks.svelte.js    # Task data store
│   │   ├── timer.svelte.js    # Timer and Pomodoro logic
│   │   └── theme.svelte.js    # Dark mode state
│   └── components/
│       ├── Header.svelte
│       ├── StartView.svelte
│       ├── ActiveView.svelte
│       ├── TaskModal.svelte
│       ├── DeleteModal.svelte
│       └── HistoryList.svelte
├── public/
│   ├── sw.js              # Service Worker for PWA functionality
│   ├── manifest.json      # PWA manifest
│   ├── icon-192.svg       # App icon (192x192)
│   └── icon-512.svg       # App icon (512x512)
├── vite.config.js         # Vite configuration with Svelte plugin
├── tailwind.config.js     # Tailwind CSS configuration
├── postcss.config.js      # PostCSS configuration
└── package.json           # Dependencies and scripts
```

## Important Implementation Details

### Svelte 5 Runes

This project uses Svelte 5's runes syntax for state management:

- **$state**: Reactive state variables (e.g., `let tasks = $state([])`)
- **$derived**: Computed values that update automatically (e.g., `let filteredProjects = $derived.by(...)`)
- **$effect**: Side effects that run when dependencies change
- **$props**: Component props declaration

Store files use the `.svelte.js` extension to enable runes outside of `.svelte` components.

### Timer Persistence

The timer does NOT persist across page reloads. If a user refreshes while tracking, the active task is lost. However, the app now includes a `beforeunload` event handler (src/stores/timer.svelte.js:65-73) that warns users before accidentally closing the tab with an active timer.

### Modal Mode System

The TaskModal serves three purposes controlled by `taskModalMode` in App.svelte:

- **'start'**: Task name, project, and timer mode selection (stopwatch/Pomodoro) for immediate tracking
- **'manual'**: Task name, project, start/end datetime inputs for past entries
- **'edit'**: Same as manual but pre-populated with existing task data

The `editingTaskIndex` state variable tracks which task is being edited (null when not editing).

### Pomodoro Timer

The Pomodoro timer (src/stores/timer.svelte.js:87-112):

- Alternates between 25-minute work phases and 5-minute break phases
- Shows browser notifications when phases switch
- Requests notification permission on first Pomodoro start
- State includes: current phase, remaining time, and duration settings

### Project Autocomplete

The TaskModal includes a project autocomplete feature (src/components/TaskModal.svelte:13-32):

- Displays up to 10 most recent unique projects from task history
- Filters suggestions as user types
- Allows selecting from dropdown or entering a new project name

### Browser Notifications

Browser notification support for Pomodoro timer (src/stores/timer.svelte.js:12-22):

- Requests permission when first Pomodoro starts
- Notifies when switching from work to break ("Time for a break!")
- Notifies when switching from break to work ("Back to work!")

### CSV Export

The export function uses the modern File System Access API (`showSaveFilePicker`) when available, with automatic fallback to blob download for unsupported browsers. Export includes all task fields: Task Name, Project, Start Time, End Time, and Duration.

### Accessibility

Modal components include proper ARIA attributes:

- `role="dialog"` and `aria-modal="true"` on modal containers
- `aria-labelledby` linking to modal titles
- Proper focus management
- Accessibility warnings resolved per recent commit (1da73df)

### Checkbox Selection

- Individual checkboxes use `data-index` attributes to map to task array indices
- "Select All" checkbox is disabled when history is empty
- Delete button dynamically shows count of selected items
- Selection state doesn't persist across history reloads

## Common Modification Scenarios

### Adding New Fields to Tasks

1. Update the task record structure in the timer store's `stopTask` function (src/stores/timer.svelte.js:114-133)
2. Modify TaskModal component to include input for the new field
3. Update HistoryList component to display the new field in the table
4. Update the CSV export function to include the new field
5. Consider migration strategy for existing localStorage data

### Adding New Components

1. Create the component file in `src/components/`
2. Use Svelte 5 runes syntax ($state, $props, $derived, $effect)
3. Import and use the component in App.svelte or other parent components
4. Follow existing patterns for dark mode support (use Tailwind's `dark:` prefix)

### Modifying Timer Behavior

Edit `src/stores/timer.svelte.js`:

- Timer interval logic is in the `startTimer` function (lines 24-55)
- Pomodoro durations are in `pomodoroState` initialization (lines 3-8)
- Notification logic is in the `notifyUser` function (lines 12-22)

### Working with Stores

Store files use `.svelte.js` extension and export functions to read/mutate state:

- **tasks store**: getTasks(), addTask(), updateTask(), deleteTasks()
- **timer store**: getActiveTask(), getElapsedMs(), getPomodoroState(), startTask(), stopTask()
- **theme store**: getIsDark(), toggleTheme()

Always use exported functions rather than directly mutating store state.

### Changing the Caching Strategy

Edit public/sw.js:24-60. Current strategy is network-first with cache fallback. HTML files always bypass cache to ensure updates are immediately visible.

### Working with Tailwind CSS

Tailwind 4.x classes are processed at build time. To add custom styles:

1. Add utility classes directly in Svelte component markup
2. For custom CSS, add to src/main.css (will be processed by Tailwind)
3. Extend Tailwind theme in tailwind.config.js

Tailwind's dark mode uses the 'class' strategy, toggling the `dark` class on the `<html>` element (managed by theme store).

### Adding New UI Modes

Follow the pattern of `taskModalMode` and `editingTaskIndex` state variables in App.svelte. Create handler functions in App.svelte and pass them as props to child components.

## Svelte MCP Tools

You are able to use the Svelte MCP server, where you have access to comprehensive Svelte 5 and SvelteKit documentation. Here's how to use the available tools effectively:

### 1. list-sections

Use this FIRST to discover all available documentation sections. Returns a structured list with titles, use_cases, and paths.
When asked about Svelte or SvelteKit topics, ALWAYS use this tool at the start of the chat to find relevant sections.

### 2. get-documentation

Retrieves full documentation content for specific sections. Accepts single or multiple sections.
After calling the list-sections tool, you MUST analyze the returned documentation sections (especially the use_cases field) and then use the get-documentation tool to fetch ALL documentation sections that are relevant for the user's task.

### 3. svelte-autofixer

Analyzes Svelte code and returns issues and suggestions.
You MUST use this tool whenever writing Svelte code before sending it to the user. Keep calling it until no issues or suggestions are returned.

### 4. playground-link

Generates a Svelte Playground link with the provided code.
After completing the code, ask the user if they want a playground link. Only call this tool after user confirmation and NEVER if code was written to files in their project.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/poncardasm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
