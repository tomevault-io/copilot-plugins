## nec-timetable-management

> This folder contains the Faculty Mobile Application for the Nandha Engineering College Academic Timetable Management System.

This folder contains the Faculty Mobile Application for the Nandha Engineering College Academic Timetable Management System.

PROJECT:
Nandha Engineering College
B.E. Computer Science and Engineering
R22 Curriculum
Academic Year 2024–2025 onwards

CURRENT PRODUCT SCOPE:
This repository is ONLY for the Faculty Mobile Application.

PRIMARY USER:
Faculty

PLATFORM:
Mobile Application

TECHNOLOGY:
- React Native / Expo
- TypeScript
- Zustand
- Axios
- React Navigation
- Expo Secure Store
- REST API
- Git/GitHub

IMPORTANT ROLE BOUNDARY:
This repository belongs to the Faculty Mobile Application.

DO NOT implement:
- Department Admin Web Application
- Global timetable generation engine
- CSP/constraint solver
- Faculty conflict resolution algorithm
- Class conflict resolution algorithm
- PostgreSQL/database logic inside the mobile app
- Independent timetable generation logic
- Random faculty allocation
- Random subject allocation

The mobile app is a CLIENT of the shared backend.

==================================================
BUSINESS CONTEXT
==================================================

The college timetable problem is a global scheduling problem.

A single faculty member may teach:
- multiple years
- multiple semesters
- multiple classes/sections
- multiple courses

Example:
Faculty X may handle:

3rd Year / Semester V / CSE-C
→ UI/UX Design
→ 3 periods/week

2nd Year / Semester III / CSE-D
→ Computer Networks
→ 3 theory periods/week

2nd Year / Semester III / CSE-D
→ Computer Networks Laboratory
→ 4 continuous periods/week

The backend/global timetable engine resolves all such scheduling conflicts.

The mobile app must only display the final valid assignment returned by the backend.

==================================================
SOURCE OF TRUTH
==================================================

The official CSE R22 curriculum is the source of truth for:
- Department
- Year
- Semester
- Course Code
- Course Name
- Course Category
- L/T/P/C
- Theory/Lab classification
- Elective structure
- Official curriculum applicability

NEVER invent or guess:
- course codes
- course names
- semesters
- faculty allocations
- timetable slots
- curriculum rules

When curriculum information is required, use backend/API data derived from the official curriculum.

==================================================
FACULTY RESPONSIBILITY
==================================================

Faculty members log in using their credentials.

After authentication:
- Faculty identity comes from the authenticated account.
- Do not ask the faculty to repeatedly enter their name.
- Faculty can view/manage the courses they handle, only through backend-supported APIs.
- Faculty can view their assigned classes and timetable.

Faculty does NOT manually create timetable slots.

Faculty does NOT resolve timetable conflicts.

==================================================
MOBILE APP RESPONSIBILITIES
==================================================

The mobile app should support:

1. Login
2. Faculty Profile
3. Faculty Dashboard
4. Courses Handled
5. Assigned Classes
6. Today's Schedule
7. Weekly Timetable
8. Theory Schedule
9. Laboratory Schedule
10. Faculty Workload
11. Notifications
12. Settings
13. Logout

The app must clearly display:
- Course Code
- Course Name
- Department
- Year
- Semester
- Class/Section
- Session Type
- Day
- Period
- Lab block where applicable

==================================================
CURRENT ARCHITECTURE
==================================================

Preferred flow:

Faculty Mobile App
        ↓
API Client
        ↓
Shared Backend / REST API
        ↓
Database / Timetable Engine

The mobile application must NEVER directly access the database.

The mobile application must NEVER contain the global scheduling engine.

==================================================
API RULES
==================================================

Use the existing centralized API client.

Do not place raw API calls throughout UI components.

Prefer:
Screen
→ Hook / Service
→ API client
→ Backend

Reuse existing:
- Axios configuration
- Authentication interceptor
- Token refresh
- Error normalization
- Type definitions
- Zustand auth store
- Navigation
- UI components

Before creating a new endpoint:
1. Inspect the existing backend/API contract.
2. Reuse an existing endpoint when possible.
3. If functionality is genuinely missing, report the missing backend capability.
4. Do not create fake production endpoints or mock business logic.

==================================================
AUTHENTICATION & SECURITY
==================================================

Use the existing authentication architecture.

Requirements:
- Secure token storage
- Protected screens
- Session persistence
- Token refresh
- Logout
- Expired-session handling
- No plaintext credentials
- No hardcoded secrets

Faculty must only access their own:
- profile
- courses
- assignments
- timetable
- workload
- notifications

Backend remains the authoritative access-control layer.

==================================================
TIMETABLE RULES
==================================================

The mobile app displays the timetable generated by the backend.

Do NOT schedule or modify it locally.

The backend guarantees:
- one class cannot have two courses in the same period
- one faculty cannot teach two classes in the same period
- course belongs to correct curriculum
- faculty is eligible for course
- required periods are satisfied
- applicable laboratory continuity rules are satisfied

If a lab is represented as a 4-period block, display it as one continuous session such as:
P1–P4

Do not split it visually into unrelated sessions.

Do not invent bell timings.

If actual period clock times are not supplied by the backend/configuration, display P1–P7 only.

==================================================
UI/UX PRINCIPLES
==================================================

This is a faculty-facing mobile application.

Prioritize:
- Today's schedule
- Next class
- Weekly timetable
- Courses handled
- Assignments
- Workload
- Notifications

The UI must be:
- simple
- readable
- mobile-first
- fast
- accessible
- consistent
- touch-friendly

Use reusable components.

Avoid unnecessary complexity.

Always provide:
- Loading state
- Empty state
- Error state
- Retry/refresh where appropriate

==================================================
CURRENT PROJECT IMPLEMENTATION RULE
==================================================

Before modifying the project:

1. Inspect the existing repository.
2. Understand the existing architecture.
3. Reuse existing code where possible.
4. Do not rewrite working functionality without justification.
5. Do not create duplicate stores, API clients, navigation systems, or UI token systems.
6. Preserve existing conventions.

When making changes:
- Prefer small, focused changes.
- Keep TypeScript types accurate.
- Avoid 'any' unless absolutely unavoidable.
- Keep components reusable.
- Keep business logic outside presentation components.
- Keep API logic outside screens.

==================================================
TESTING RULES
==================================================

After meaningful implementation changes:
- Run TypeScript checks.
- Run lint checks when configured.
- Run relevant tests.
- Test the feature on Android/Expo where possible.
- Check loading/error/empty states.
- Check navigation.
- Check authenticated and unauthenticated flows.

Do not claim a feature is working unless it has actually been tested.

==================================================
CURRENT DEVELOPMENT PRIORITY
==================================================

Build in this order unless there is a strong technical reason to change it:

1. Authentication
2. Dashboard
3. Today's Schedule
4. Weekly Timetable
5. Courses
6. Assignments
7. Workload
8. Notifications
9. Profile
10. Settings
11. Testing / polish / production readiness

Current completed foundation may already contain:
- project scaffolding
- API client
- auth
- protected navigation
- dashboard
- today's schedule
- weekly timetable
- reusable loading/error/empty states

Before implementing something, inspect the repository and confirm its current status instead of assuming it is missing.

==================================================
DESIGN PRINCIPLE
==================================================

The hierarchy of the system is:

Department
→ Year
→ Semester
→ Class/Section
→ Official Curriculum
→ Course
→ Eligible Faculty
→ Timetable generated by backend

The mobile app mainly consumes the result.

The mobile app must not reverse this hierarchy or invent its own scheduling rules.

==================================================
WORKING STYLE
==================================================

When asked to implement a feature:

1. Inspect existing code first.
2. Identify dependencies.
3. State the implementation approach briefly.
4. Implement incrementally.
5. Test the result.
6. Report:
   - files changed
   - files created
   - APIs used
   - tests performed
   - known issues
   - remaining dependencies

Do not blindly implement based on assumptions.

When backend behavior is missing or ambiguous, clearly identify it instead of inventing business logic.

Always preserve the existing project architecture unless there is a concrete reason to improve it.

---
> Source: [rohith-kanna-jr-2006/nec-timetable-management](https://github.com/rohith-kanna-jr-2006/nec-timetable-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
