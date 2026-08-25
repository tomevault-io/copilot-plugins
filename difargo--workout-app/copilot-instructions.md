## workout-app

> Mobile-first fitness application built with React + Firebase.

# AGENTS.md

## Project Overview

Mobile-first fitness application built with React + Firebase.

The application contains:

- Workouts
- Nutrition
- Measurements
- Trainer Dashboard
- Admin Panel
- Telegram Integration
- Workout History
- Nutrition History

The project is actively developed and must be treated as a production application.

## Critical Development Rules

### Work only from the latest stable base

Always use the latest provided files as the source of truth.

Never restore old code versions.

Never revert features without explicit approval.

### Make minimal changes

Do not rewrite large parts of App.jsx.

Do not refactor unrelated code.

Apply the smallest possible diff.

Preserve existing architecture.

### Preserve existing functionality

Never break:

- Firebase
- Authentication
- Trainer Dashboard
- Admin Panel
- Telegram Integration
- Workout History
- Nutrition History
- Measurements
- Navigation
- Bottom Navigation Bars
- User Roles

### Required verification before completion

Before finishing any task:

1. Check JSX syntax.
2. Check all closing tags.
3. Check CSS syntax.
4. Verify build passes.
5. Verify no existing functionality was removed.
6. Verify requested changes are actually implemented.

## UI Design System

### General Style

The light, lavender iOS interface is the primary and current product design.

- Mobile first, fullscreen iOS-like UX.
- Main palette: background `#F7F6F8`, cards `#FFFFFF`, secondary surface `#F5F2F7`, borders `#EFEBF3`, text `#28262E`, secondary text `#7B7682`, accent `#8F7AC8`, active accent `#806BB6`.
- Do not restore the former grey-blue, bright-purple, or Premium Dark colours and components.
- Every new page must visually match the current light iOS interface.

### Design Principles

- Compact layouts
- Minimal visual noise
- Consistent spacing
- Consistent border radius
- Consistent component heights
- One unified product appearance
- Use fixed, opaque iOS-style headers that account for the safe area.
- Keep bottom bars consistent in height, spacing, and element placement.
- Treat modals as separate, polished sheets with padding, rounded corners, a shadow, and a close button.

### Style Architecture

- Preserve the existing CSS Modules structure.
- Keep component styles colocated; do not move them into one global CSS file.
- Reuse established interface patterns instead of introducing legacy components or a parallel visual system.
- Do not change business logic, Firebase, authentication, roles, or navigation while making visual changes unless explicitly requested.

## Workout Module Rules

Must preserve:

- Fixed card sizes
- No card flickering
- Fullscreen workout flow
- Workout timer
- Firebase sync
- Exercise history
- Previous workout data

Requirements:

- Open next unfinished workout
- Bottom navigation buttons
- Exercise video as primary element

## Nutrition Module Rules

Style:

FatSecret + MyFitnessPal

Must preserve:

- Food search
- AI food search
- AI photo search
- Shared "My Database"
- Multi-ingredient dishes
- Automatic nutrition calculation
- Compact cards
- Small "+" buttons
- Fast search UX

## Measurements Module

Current implementation:

12 measurements:

- Weight
- Neck
- Shoulder Girth
- Chest
- Biceps
- Forearm
- Wrist
- Belly
- Pelvis
- Thigh
- Calf
- Ankle

Images stored in:

public/measurements/

Requirements:

- Do not change image paths unless requested.
- Measurements must always load from latest saved measurement.
- Preserve fullscreen measurement wizard.

## Trainer Dashboard Rules

Roles:

- Client
- Trainer
- Admin

Trainer can access:

- Trainer Dashboard
- Clients
- Workouts
- Nutrition
- History

Trainer must only see:

- Assigned clients
- Assigned client data
- Assigned client statistics

Never expose unrelated users.

## Admin Panel Rules

Admin functionality must never be broken.

## Images

Never generate images automatically.

Only generate images if explicitly requested with:

- "сгенерируй"
- "создай картинку"

## Versioning

Increment the application version for completed runtime releases. Documentation-only changes must not change the runtime version.

---
> Source: [DiFargo/workout-app](https://github.com/DiFargo/workout-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
