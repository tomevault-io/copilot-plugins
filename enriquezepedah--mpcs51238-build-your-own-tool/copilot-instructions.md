## mpcs51238-build-your-own-tool

> A personal workout tracking app built with Next.js 16 and Tailwind CSS v4. Log workouts with exercises, sets, reps, and weight. Create reusable routines. Browse an exercise library organized by muscle group. View personal records and weekly progress.

# Workout Tracker

A personal workout tracking app built with Next.js 16 and Tailwind CSS v4. Log workouts with exercises, sets, reps, and weight. Create reusable routines. Browse an exercise library organized by muscle group. View personal records and weekly progress.

## Tech Stack

- Next.js 16 (App Router)
- TypeScript
- Tailwind CSS v4
- React Context + useReducer for state management
- Client-side state only (data resets on refresh)

## Pages

| Route | Page | Description |
|---|---|---|
| `/` | Dashboard | Quick stats, recent workouts, CTA to log |
| `/workouts` | Workout List | All workouts sorted by date |
| `/workouts/new` | Log Workout | Form to add a new workout (supports ?routine= param) |
| `/workouts/[id]` | Workout Detail | Full breakdown of a single workout |
| `/exercises` | Exercise Library | Browse ~45 exercises filtered by muscle group |
| `/routines` | Routines | List of reusable workout templates |
| `/routines/new` | Create Routine | Form to define a routine with target sets/reps |
| `/routines/[id]` | Routine Detail | View routine, start workout from it |
| `/records` | Personal Records | Best lifts per exercise, weekly volume |

## Data Model

```typescript
MuscleGroup = "Chest" | "Back" | "Shoulders" | "Biceps" | "Triceps" | "Legs" | "Glutes" | "Core" | "Full Body"
LibraryExercise { id, name, muscleGroup }
WorkoutSet { id, reps, weight }
Exercise { id, name, notes, sets: WorkoutSet[], muscleGroup?, libraryExerciseId? }
Workout { id, date, title, exercises: Exercise[], createdAt }
RoutineExercise { id, name, muscleGroup?, libraryExerciseId?, targetSets, targetReps }
Routine { id, name, exercises: RoutineExercise[], createdAt }
```

## State Management

React Context wraps the app via `WorkoutProvider` in the root layout.
- Workout actions: ADD_WORKOUT, DELETE_WORKOUT, UPDATE_WORKOUT
- Routine actions: ADD_ROUTINE, DELETE_ROUTINE, UPDATE_ROUTINE
- Exercise library is a static constant (not in context)
- Seed data provides sample workouts and 3 routines (Push/Pull/Legs)

## Style

Clean & minimal (Notion/Linear-inspired). Inter font, gray-50 background, white cards with borders, black accent buttons. Sidebar navigation with responsive mobile hamburger menu.

## Development

```bash
npm run dev      # Start dev server on localhost:3000
npm run build    # Production build
npm run lint     # ESLint
npx playwright test  # Run Playwright tests
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EnriqueZepedaH)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/EnriqueZepedaH)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
