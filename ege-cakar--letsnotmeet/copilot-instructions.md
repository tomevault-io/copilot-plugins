## letsnotmeet

> - Development: `npm start`

# CLAUDE.md - Guide for AI Assistance

## Build/Run Commands
- Development: `npm start`
- Build: `npm run build`
- Test: `npm test` (all tests)
- Single test: `npm test -- -t "test name pattern"`
- Lint: ESLint is built into Create React App (check using `npm run build`)

## Code Style Guidelines
- **Imports**: React first, then services, hooks, components, CSS last
- **Formatting**: Follow existing indentation (2 spaces) and line length
- **Components**: PascalCase for components (MeetingFinder, CalendarView)
- **Variables/Functions**: camelCase (fetchEvents, handleSubmit)
- **File Naming**: Match component names (CalendarView.js)
- **Error Handling**: Use try/catch with console.error, then throw errors up
- **Component Structure**: Props at top, hooks next, handlers, then return
- **Firebase Integration**: Use service files in src/services/
- **Google Calendar API**: All calendar operations through calendarService.js

Use existing patterns when extending functionality. This is a React app with Firebase authentication and Google Calendar integration for scheduling meetings.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ege-Cakar)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Ege-Cakar)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
