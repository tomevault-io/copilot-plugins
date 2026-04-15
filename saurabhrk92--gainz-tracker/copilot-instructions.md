## gainz-tracker

> **Gainz Tracker** is a comprehensive fitness tracking application built with Next.js 15.4.2 and TypeScript. It's a progressive web app for tracking workouts, exercises, templates, and progress with offline-first capabilities and Google Drive cloud sync.

# Gainz Tracker - Claude Memory Documentation

## Repository Overview
**Gainz Tracker** is a comprehensive fitness tracking application built with Next.js 15.4.2 and TypeScript. It's a progressive web app for tracking workouts, exercises, templates, and progress with offline-first capabilities and Google Drive cloud sync.

## Tech Stack & Architecture

### Core Technologies
- **Frontend**: Next.js 15.4.2 (App Router), TypeScript, Tailwind CSS
- **Database**: IndexedDB (client-side, offline-first)
- **Authentication**: NextAuth.js with Google OAuth
- **Cloud Sync**: Google Drive API for backup/restore
- **UI Components**: Custom component library with themed design system
- **Charts**: Custom SVG-based chart components
- **Icons**: Inline SVG system with anatomically accurate muscle group icons

### Project Structure
```
/app                          # Next.js App Router pages
  /components                 # Reusable UI components
    /exercises               # Exercise-specific components
    /progress               # Progress/analytics components
    /shared                 # Shared layout components (Navigation)
    /templates              # Workout template components
    /ui                     # Base UI components (Button, Card, Modal, etc.)
    /workout                # Workout execution components
  /exercises                # Exercise library page
  /history                  # Workout history page
  /progress                 # Progress analytics page
  /settings                 # Settings and sync page
  /templates                # Workout templates page
  /workout                  # Active workout page
  
/lib                         # Core business logic
  /auth                     # Authentication utilities
  /hooks                    # Custom React hooks
  /services                 # External service integrations
  /storage                  # Database layer (IndexedDB)
  /types.ts                 # TypeScript type definitions
  /constants.ts             # App constants and configurations
  /utils.ts                 # Utility functions
  /oneRepMaxCalculator.ts   # 1RM prediction algorithms
  /popular-exercises.ts     # Seed data for popular exercises
  /supersetUtils.ts         # Superset management and workout flow utilities
```

## Key Features & Implementation

### 1. Exercise Management
- **Exercise Library**: Create, edit, delete exercises
- **Equipment Types**: barbell, dumbbell, fixed_bar, machine, bodyweight, cable
- **Muscle Groups**: chest, back, shoulders, arms, legs, glutes, core, calves
- **Smart Defaults**: Rest times default from exercise settings
- **Usage Tracking**: Warns when deleting exercises used in workouts/templates

### 2. Workout Templates
- **Template Creation**: Exercise + Sets + Rest Time (simplified, no rep ranges)
- **Dynamic Rest Times**: Auto-updates when exercise changes
- **Day Assignment**: Assign templates to specific days or routine
- **Active Templates**: One active template per day
- **Usage Validation**: Prevents deletion of templates with workout history

### 3. Workout Execution
- **Live Workouts**: Real-time workout tracking with timers
- **Set Logging**: Track reps, weight, rest times
- **Progress Tracking**: Visual exercise overview with completion status
- **Rest Timer**: Automatic rest timer between sets
- **Plate Calculator**: Calculate barbell plate combinations
- **Early End**: Option to end workout early with preserved data

### 4. Progress Analytics & 1RM System
- **1RM Prediction Engine**: Multiple research-based formulas
  - Epley: `weight × (1 + 0.0333 × reps)` (best for 2-5 reps)
  - Brzycki: `weight × (36 / (37 - reps))` (best for 6-10 reps)
  - Lander: `(100 × weight) / (101.3 - 2.67123 × reps)`
  - Lombardi: `weight × reps^0.1` (for higher reps)
- **Smart Logic**: Skips actual 1RM sets, uses working sets (2+ reps)
- **Error Bars**: Shows min/max/average across all formulas
- **Confidence Levels**: Based on rep range accuracy
- **Dual Views**: Toggle between "1RM Estimates" and "Actual Maxes"
- **Volume Tracking**: Weekly/monthly volume progression charts
- **Swipeable Interface**: Navigate between exercises

### 5. Data Persistence & Sync
- **Local-First**: IndexedDB for offline functionality
- **Smart Sync**: Background cloud sync without UI blocking
- **Backup System**: Manual and automatic Google Drive backups
- **Conflict Resolution**: Handles sync conflicts gracefully
- **Auto-Sync**: Every 30 minutes + workout events
- **Throttling**: Intelligent sync frequency to avoid API limits
- **Auto-Restore**: Detects fresh login and automatically restores latest backup from Google Drive

## Database Schema (IndexedDB)

### Core Entities
```typescript
Exercise {
  id: string
  name: string
  muscleGroup: MuscleGroup
  type: EquipmentType
  barWeight?: number
  defaultRestTime: number
  created: Date
  lastUsed: Date
}

WorkoutTemplate {
  id: string
  name: string
  muscleGroup: MuscleGroup | 'full_body'
  day: WeekDay
  exercises: TemplateExercise[]
  isActive: boolean
  created: Date
  lastUsed: Date
}

TemplateExercise {
  exerciseId: string
  targetSets: number
  restTime?: number
  order?: number
  notes?: string
  // Superset functionality
  supersetGroup?: string
  supersetOrder?: number
  restBetweenExercises?: number
}

WorkoutSession {
  id: string
  templateId: string
  date: Date
  status: 'in_progress' | 'completed' | 'paused' | 'ended_early'
  exercises: SessionExercise[]
  duration: number
  totalVolume: number
}

SessionExercise {
  exerciseId: string
  targetSets: number
  completedSets: number
  sets: Array<{ reps: number; weight: number }>
}
```

## UI/UX Design System

### Color Palette
- **Primary**: Purple gradient (`bg-gradient-primary`)
- **Muscle Groups**: Each has unique color (chest: #ff7c7c, back: #8dd1e1, etc.)
- **Status Colors**: Green (success), Red (danger), Orange (warning), Blue (info)

### Component Library
- **Button**: Primary, secondary, danger, glass variants
- **Card**: Consistent container with rounded corners and shadows
- **Modal**: Overlay system with size variants (sm, md, lg, xl)
- **ConfirmModal**: Custom themed confirmation dialogs (replaces browser prompts)
- **Icon System**: Inline SVG with anatomically accurate muscle group icons

### Navigation
- **Hamburger Menu**: Slide-out navigation (no close button per user preference)
- **Pages**: Home, Templates, Exercises, History, Progress, Settings
- **Active States**: Visual indicators for current page

## Recent Major Changes & User Preferences

### Superset Functionality (Latest Addition)
- **Superset Creation**: Group 2-4 exercises with multi-select UI and configurable rest times
- **Visual Grouping**: Green-framed layout showing exercises grouped together
- **Smart Navigation**: Automatic progression through superset exercises during workouts
- **Rest Timer Logic**: Short rest between exercises (10-30s), longer rest after completing superset (60-180s)
- **Single Unit Management**: Delete entire superset as one unit, treat as single exercise conceptually
- **Workout Integration**: Visual superset indicators and smart rest timing during execution

### Template System Improvements
- **Removed**: Rep range field (user feedback: not useful)
- **Enhanced**: Dynamic rest time updates
- **Simplified**: Only Exercise + Sets + Rest Time

### 1RM Prediction System
- **Implementation**: Complete prediction engine with multiple formulas
- **User Request**: Skip actual 1RM sets, use working sets for prediction
- **Visualization**: Error bars showing prediction uncertainty
- **Charts**: Enhanced with proper axes, grid lines, and date formatting

### UI/UX Improvements
- **Custom Modals**: Replaced all browser confirm/alert dialogs
- **Async Operations**: All backup operations are fully non-blocking
- **Responsive Design**: Fixed overflow issues in backup cards
- **Icon Accuracy**: Detailed, anatomically correct muscle group icons
- **Navigation**: Removed close button from sidebar per user preference

### Data Management
- **Seed Data**: Removed all example data, added 40 popular exercises
- **Auto-Population**: Exercises auto-populate on first load
- **Delete Protection**: Warns about usage before deletion
- **Auto-Restore on Login**: Automatically restores latest backup when user logs into Google Drive for the first time

## Performance Optimizations

### Async Operations
- **Backup Creation**: Fully asynchronous with immediate UI feedback
- **Workout Sync**: Background sync without blocking UI
- **Chart Rendering**: Optimized SVG rendering with proper scaling
- **Database Operations**: IndexedDB with proper error handling

### UI Responsiveness
- **Lazy Loading**: Components load as needed
- **Optimistic Updates**: UI updates immediately, sync happens background
- **Error Boundaries**: Graceful error handling throughout app
- **Offline Support**: Full functionality without internet

## Development Guidelines

### Code Patterns
- **TypeScript**: Strict typing throughout
- **Async/Await**: Proper error handling with try/catch
- **Component Structure**: Functional components with hooks
- **State Management**: useState/useEffect, no external state library
- **Database Layer**: Abstracted through custom DB service

### User Experience Principles
- **Offline First**: App works without internet
- **No Blocking Operations**: All heavy operations are async
- **Immediate Feedback**: UI responds instantly to user actions
- **Data Safety**: Multiple backup layers and conflict resolution
- **Intuitive Design**: Minimal learning curve for fitness enthusiasts

## API Integrations

### Google Drive Sync
- **Authentication**: OAuth 2.0 through NextAuth.js
- **Backup Format**: JSON exports of IndexedDB data
- **Sync Strategy**: Smart throttling based on activity
- **Error Handling**: Silent failures with retry logic
- **File Management**: Automatic cleanup of old backups

### Future Extensibility
- **Plugin System**: Ready for additional equipment types
- **Exercise Database**: Can integrate with external exercise APIs
- **Social Features**: Architecture supports sharing/collaboration
- **Analytics**: Event tracking for usage analytics

## Critical Implementation Notes

### Superset Implementation
```typescript
// Data structure for superset grouping
TemplateExercise {
  supersetGroup?: string;     // UUID for grouping exercises
  supersetOrder?: number;     // Order within superset (1, 2, 3...)
  restBetweenExercises?: number; // Short rest between exercises (10-30s)
}

// Smart rest time calculation
const restTime = getRestTimeForTransition(currentExercise, nextExercise);
// Returns short rest if moving within superset, longer rest if moving between supersets

// Workout structure organization
const structure = organizeWorkoutStructure(templateExercises);
// Groups exercises into individual and superset categories for proper flow
```

### 1RM Calculation Logic
```typescript
// User-specified logic: Skip 1RM sets, use working sets
const workingSets = sets.filter(set => set.reps > 1);
// Prioritize lowest rep working sets for accuracy
const sortedSets = workingSets.sort((a, b) => {
  if (a.reps !== b.reps) return a.reps - b.reps;
  return b.weight - a.weight;
});
```

### Async Backup Pattern
```typescript
// Non-blocking backup with immediate UI feedback
setSyncStatus('Creating backup...');
setTimeout(async () => {
  try {
    await syncService.uploadBackup('manual');
    setSyncStatus('Backup created successfully!');
    setTimeout(() => setSyncStatus(''), 3000);
  } catch (error) {
    setSyncStatus('Backup failed');
    setTimeout(() => setSyncStatus(''), 5000);
  }
}, 100);
```

### Auto-Restore on Fresh Login
```typescript
// Detect transition from unauthenticated to authenticated
const wasAuthenticated = prevAuthenticatedRef.current;
const isNowAuthenticated = isAuthenticated;

if (wasAuthenticated === false && isNowAuthenticated === true) {
  handleFreshLogin(); // Only restore on actual login, not auth state checks
}

// Session-based deduplication prevents multiple restoration attempts
const sessionKey = 'gainz_restore_attempted';
const hasAttemptedRestore = sessionStorage.getItem(sessionKey);
```

### Chart Implementation
- **Custom SVG**: No external chart libraries
- **Responsive**: Scales properly on all devices
- **Interactive**: Hover states and touch-friendly
- **Error Bars**: Visual uncertainty indicators for 1RM predictions

## Common User Workflows

### Creating a Workout
1. Templates → Create New Template
2. Add exercises (auto-populated rest times)
3. Set target sets per exercise
4. Assign to day/routine
5. Activate template

### Executing a Workout
1. Home → Start Workout (or Resume if active)
2. Navigate through exercises
3. Log sets (reps + weight)
4. Use rest timer between sets
5. Complete or end early

### Tracking Progress
1. Progress → Toggle 1RM/Actual view
2. Swipe between exercises
3. View strength progression with error bars
4. Analyze volume trends over time

### Managing Data
1. Settings → Create Backup (async)
2. Restore from backup (with confirmation)
3. Delete old backups
4. All operations with custom themed modals

## Testing & Deployment
- **Build**: `npm run build` (all builds are successful)
- **TypeScript**: Strict type checking enabled
- **Production**: Static export compatible
- **Performance**: Optimized for mobile devices

---

*This documentation serves as comprehensive memory for Claude to understand the entire Gainz Tracker codebase, user preferences, implementation decisions, and architectural patterns.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/saurabhrk92)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/saurabhrk92)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
