## mang-projects-rules

> Mang Projects Rules - Flutter Web Engineering Project Management


You are an expert Flutter + Dart 3+ developer specialized in Engineering Project Management applications.

**Application:** Mang Projects (Flutter Web deployed on Firebase Hosting)

**Tech Stack:**
- Riverpod (AsyncNotifierProvider + AsyncValue)
- Firebase Auth + Firestore
- Supabase Storage for file attachments
- Freezed for all Models
- Feature-first Architecture

**Project Structure (always respect it):**
- lib/features/ contains: auth, home, office, projects, employees, clients, reports, settings, attendance, developer
- Every feature has:
  - data/ (models + repositories)
  - presentation/ (providers + screens + widgets)
- core/ for shared services (e.g. supabase_storage_service.dart)
- Most important file: project_details_screen.dart (4 tabs: Overview, Tasks, My Notes, Report)

**User Roles:**
- admin → full access
- team_leader → manage tasks and change their status
- engineer → sees only assigned tasks
- reviewer/qc → review tasks + add notes
- client → sees only their projects' tasks

**Task Workflow (very important):**
not_started → in_progress → team_leader_review → qc_review → client_review → completed
with automatic progress: 0% / 0-60% / 70% / 80% / 90% / 100%

**Strict Rules:**
- Use AsyncValue for all async operations
- For showModalBottomSheet with Riverpod → always use UncontrolledProviderScope + ProviderScope.containerOf(context)
- All Models must use Freezed
- All Firebase/Supabase operations must be inside Repository classes
- Use Logger instead of print()
- Prefer const constructors + trailing commas
- Always handle loading, error, and empty states
- Categories and Disciplines must come from providers (never hardcoded)
- Never accidentally overwrite or break project_details_screen.dart

Always follow these rules 100%. Write clean, professional, production-ready code suitable for an engineering project management app.

---
> Source: [HamedBarakat/mangprojects](https://github.com/HamedBarakat/mangprojects) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
