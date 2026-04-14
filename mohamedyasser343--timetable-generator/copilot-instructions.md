## timetable-generator

> Handles the real-world case where courses may have type "LECTURE AND LAB" but rooms are strictly "LECTURE" or "LAB". The `roomTypeCompatible()` method allows flexible matching using string inclusion checks.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **timetable generator** built with NestJS that uses **Constraint Satisfaction Problem (CSP) solving** to schedule courses, instructors, rooms, and time slots. The system imports data from CSV files, applies hard and soft constraints, and generates a feasible timetable.

## Essential Commands

### Development
```bash
npm run start:dev          # Start server in watch mode (uses ts-node-dev)
npm run start              # Start server in production mode
```

### Data Import
```bash
npm run import:csv         # Import CSV data (courses, instructors, rooms, timeslots) into SQLite
```

The import command runs [src/import-csv.ts](src/import-csv.ts) to load data from the root CSV files:
- `courses.csv` - Course information (CourseID, CourseName, Credits, Type, SessionsPerWeek)
- `instructors.csv` - Instructor data (InstructorID, Name, Role, PreferredSlots, QualifiedCourses)
- `rooms.csv` - Room data (RoomID, Type, Capacity, Building, Floor)
- `timeSlots.csv` - Time slot data (Day, StartTime, EndTime, Priority)
- `sections.csv` - Section data (SectionID, CourseCode, SectionName, Capacity, PreferredInstructor)

### API Endpoints
- `POST /timetable/generate` - Generate timetable using CSP solver and save to database
- `GET /timetable` - Retrieve all timetable entries

Server runs on `http://localhost:3000` with CORS enabled.

### Frontend
Static HTML/CSS/JS calendar UI in [frontend/](frontend/):
- `index.html` - Calendar grid layout with filtering controls
- `app.js` - Fetches timetable from API, renders entries on calendar
- Filter by instructor, room, or course

## Architecture

### Database Layer
- **ORM**: TypeORM with SQLite (`timetable.sqlite`)
- **Configuration**: [ormconfig.ts](ormconfig.ts) - single shared DataSource instance
- **Entities** (in [src/entities/](src/entities/)):
  - `Course` - Primary key: `code`, includes `sessionsPerWeek` field
  - `Instructor` - Primary key: `externalId`
  - `Room` - Primary key: `name`, includes optional `building` and `floor` fields
  - `TimeSlot` - Auto-generated ID, includes `priority` field (0=normal, 1=early morning, 2=late evening)
  - `Section` - Primary key: `id` (e.g., "CSC111-A"), references Course via `courseCode`, includes `capacity` and optional `preferredInstructor`
  - `TimetableEntry` - Generated schedule assignments with `sectionId`, `courseCode`, `instructorId`, `roomName`, `timeslotId`

### Module Structure
- **ImportModule** ([src/modules/import/](src/modules/import/)) - CSV import service
- **TimetableModule** ([src/modules/timetable/](src/modules/timetable/)) - Timetable generation and retrieval
- **CspModule** ([src/modules/csp/](src/modules/csp/)) - Core CSP solver implementation

### CSP Solver Details ([src/modules/csp/csp.service.ts](src/modules/csp/csp.service.ts))

**Hard Constraints** (must be satisfied):
1. **No professor teaches multiple classes at same time** - Each instructor can only be assigned to one timeslot at a time
2. **No room hosts multiple classes at same time** - Each room can only be used for one section per timeslot
3. **Each section has all required sessions per week** - Sections inherit `sessionsPerWeek` from their course and get multiple timetable entries
4. **Room type must match course type** - Uses `roomTypeCompatible()` method that handles:
   - Exact matches (LAB→LAB, LECTURE→LECTURE)
   - "LECTURE AND LAB" courses can use either LECTURE or LAB rooms
5. **Room capacity must accommodate section** - Room capacity must be >= section capacity
6. **Instructor day preferences** - "Not on <Day>" restrictions are enforced (with fallback relaxation if needed)

**Soft Constraints** (preferences, implemented in scoring function):
1. **Prefer qualified instructors** (score -50) - Instructors listed in course's `qualifiedCourses` field
2. **Prefer section's preferred instructor** (score -30) - If section has `preferredInstructor` specified
3. **Avoid early morning/late evening slots** (+10 per priority level) - Uses `TimeSlot.priority` field
4. **Avoid consecutive distant rooms for same instructor** (+5 per floor/building distance) - Calculates distance using `Room.building` and `Room.floor`
5. **Distribute section sessions evenly across week** (+15 penalty per existing section assignment on same day) - Prevents clustering
6. **Minimize gaps for students** (+3 per hour of gap) - Prefers consecutive or near-consecutive time slots for same section

**Algorithm**:
- **Variable model**: Each section session (section × course.sessionsPerWeek) is a separate variable
- **Domain construction**: For each session, generate all valid (timeslot, room, instructor) combinations respecting hard constraints
- **Fallback mechanism**: If instructor preferences make a section impossible to schedule, preferences are relaxed
- **Variable ordering**: MRV heuristic - sessions with smaller domains assigned first
- **Value ordering**: Soft constraint scoring - tries best-scored assignments first
- **Backtracking search**: Standard chronological backtracking with constraint checking

## Key Implementation Details

### Sections
Sections represent individual offerings of a course (e.g., CSC111-A, CSC111-B). Each section:
- References a course via `courseCode`
- Has its own `capacity` for room matching
- Can have an optional `preferredInstructor`
- Inherits `sessionsPerWeek` from its course

The solver schedules sections (not courses directly). This allows multiple sections of the same course to be scheduled independently.

### Multiple Sessions Per Week
The `Course.sessionsPerWeek` field determines how many times each section meets. The solver creates separate variables for each section session (e.g., a section of a course with sessionsPerWeek=2 creates 2 scheduling variables). This satisfies the hard constraint "each section must have all required lectures per week".

### Room Type Compatibility
Handles the real-world case where courses may have type "LECTURE AND LAB" but rooms are strictly "LECTURE" or "LAB". The `roomTypeCompatible()` method allows flexible matching using string inclusion checks.

### Instructor Availability
The `preferredSlots` field supports day restrictions like "Not on Sunday". The CSP solver parses this and filters out forbidden timeslots. If this makes a course impossible to schedule, the constraint is relaxed for that course only.

### Soft Constraint Scoring
All soft constraints are combined into a single numeric score (lower = better). The solver sorts domain values by score before trying them, implementing a least-constraining-value heuristic that guides the search toward better solutions.

### TypeScript Configuration
- Target: ES2021
- Module: CommonJS
- Decorators enabled (required for NestJS)
- Base URL: `./` for path resolution

### Data Flow
1. CSV files → `ImportService.importCsvData()` → SQLite database (with entity fields validated)
2. User calls `POST /timetable/generate` → `CspService.solve()` → backtracking with constraint checking
3. Solution (array of section session assignments) saved to `TimetableEntry` table
4. User retrieves via `GET /timetable`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MohamedYasser343)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MohamedYasser343)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
