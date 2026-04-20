## georgesticker

> \# TickTick Clone - Electron Application

\# TickTick Clone - Electron Application



Create a cross-platform Electron desktop application that functions as a comprehensive task management and productivity tool. Build iteratively, starting with core functionality and expanding features progressively.



\## Technology Stack



\- Electron (latest stable)

\- React with TypeScript for the renderer process

\- SQLite for local data persistence (use better-sqlite3)

\- Tailwind CSS for styling

\- Electron Store for application settings

\- node-schedule for reminders/notifications



\## Available Tools \& Workflow



You have access to MCP servers. Use them as follows:



\### Documentation (Use Proactively)

\- \*\*Context7\*\*: Query BEFORE implementing any Electron, React, or library-specific code. Always fetch current API documentation rather than relying on training knowledge. Electron APIs change frequently.



\### Database

\- \*\*SQLite MCP\*\*: Use for inspecting the database during development, verifying schema migrations, and debugging data issues. Query the database directly when troubleshooting.



\### Planning

\- \*\*Sequential Thinking\*\*: Use when planning architecture decisions, complex features (recurring tasks, calendar view), or when encountering non-trivial problems. Think through the approach before coding.



\### Persistence

\- \*\*Memory MCP\*\*: Store key architectural decisions, schema designs, and implementation notes. Retrieve at the start of each session.



\### Version Control

\- \*\*GitHub MCP\*\*: Commit after each meaningful milestone with descriptive messages.



\### Workflow Rules



1\. Before writing Electron main process code → query Context7 for current Electron docs

2\. Before implementing a complex feature → use Sequential Thinking to plan

3\. After completing a feature → commit with a descriptive message

4\. When debugging data issues → inspect with SQLite MCP

5\. At session end → store decisions in Memory MCP

6\. At session start → retrieve previous decisions from Memory MCP



\## File Structure



Organise with clear separation:

```

/src

&nbsp; /main         - Electron main process

&nbsp; /renderer     - React application

&nbsp; /shared       - Shared types and utilities

&nbsp; /database     - SQLite schema, migrations, and queries

/data           - SQLite database file location

```



\## Core Features (Phase 1)



\### Task Management

\- Create, read, update, and delete tasks

\- Task properties: title, description, due date, due time, priority (none, low, medium, high), completion status

\- Subtasks/checklist items within tasks

\- Task notes with basic markdown support



\### Organisation

\- Multiple lists/folders for grouping tasks

\- Smart lists: Inbox, Today, Tomorrow, Next 7 Days, All Tasks, Completed

\- Drag-and-drop reordering of tasks within lists

\- Drag-and-drop to move tasks between lists



\### User Interface

\- Clean, minimal sidebar navigation showing all lists and smart lists

\- Main content area displaying tasks for selected list

\- Quick-add task input bar (keyboard shortcut: Ctrl/Cmd+N)

\- Task detail panel (slide-out or modal) for editing

\- Light and dark theme support with system preference detection

\- Keyboard shortcuts for common actions



\## Enhanced Features (Phase 2)



\### Tags and Filtering

\- Create and assign multiple tags to tasks

\- Filter tasks by tag, priority, or date range

\- Search functionality across all tasks



\### Recurring Tasks

\- Daily, weekly, monthly, yearly recurrence patterns

\- Custom recurrence (e.g., every 3 days, specific weekdays)

\- Regenerate on completion vs fixed schedule option



\### Reminders and Notifications

\- System notifications for task reminders

\- Multiple reminders per task

\- Snooze functionality



\### Calendar View

\- Monthly calendar showing tasks by due date

\- Drag tasks to reschedule

\- Mini calendar in sidebar for date navigation



\## Productivity Features (Phase 3)



\### Pomodoro Timer

\- Integrated focus timer (25/5 minute default, configurable)

\- Link timer sessions to specific tasks

\- Session history and statistics



\### Habit Tracker

\- Create recurring habits separate from tasks

\- Visual streak tracking

\- Habit completion calendar/heatmap



\### Statistics Dashboard

\- Tasks completed over time (daily/weekly/monthly charts)

\- Productivity trends

\- Focus time summaries



\## Technical Requirements



\### Data Structure

\- Use SQLite with proper schema migrations

\- Implement a service layer separating database operations from UI

\- Support data export (JSON) and import



\### Performance

\- Virtual scrolling for large task lists

\- Lazy loading where appropriate

\- Efficient re-rendering (React.memo, useMemo where beneficial)



\### Application

\- System tray integration with quick-add capability

\- Global keyboard shortcut to show/hide window

\- Auto-launch on system startup (optional setting)

\- Proper window state persistence (size, position)



\## Development Approach



1\. Start with the Electron + React + TypeScript boilerplate

2\. Implement SQLite database layer with core schema

3\. Build the basic UI shell (sidebar, main area, task detail)

4\. Add CRUD operations for tasks and lists

5\. Implement smart lists logic

6\. Add each enhanced feature incrementally

7\. Polish with animations, keyboard shortcuts, and accessibility



\## Milestones \& Commits



Commit after completing each of these:



\- \[ ] Project initialisation and boilerplate

\- \[ ] Database schema and service layer

\- \[ ] Basic UI shell (sidebar, main area)

\- \[ ] Task CRUD operations

\- \[ ] List management

\- \[ ] Smart lists

\- \[ ] Drag-and-drop

\- \[ ] Theme support

\- \[ ] Phase 1 complete

\- \[ ] Tags and filtering

\- \[ ] Recurring tasks

\- \[ ] Reminders and notifications

\- \[ ] Calendar view

\- \[ ] Phase 2 complete

\- \[ ] Pomodoro timer

\- \[ ] Habit tracker

\- \[ ] Statistics dashboard

\- \[ ] Phase 3 complete



Begin with Phase 1. Ensure the architecture supports later phases without major refactoring.

```



---



\### Step 4: Create .gitignore



Create `~/projects/ticktick-clone/.gitignore`:

```

node\_modules/

dist/

out/

.webpack/

\*.db

\*.db-journal

.DS\_Store

\*.log

.env

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cyclopsgd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
