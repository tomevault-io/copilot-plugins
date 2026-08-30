## saif

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

QUIK ERP is a single-tenant Flutter/Dart ERP system for AMAN Infra, covering sales, production, inventory, finance, HR, and administration. Backend is Firebase (Firestore, Auth, Storage, Cloud Functions).

## Commands

```bash
# Flutter
flutter pub get          # Install dependencies
flutter analyze          # Lint/analyze Dart code
flutter run              # Run the app

# Cloud Functions (from /functions directory)
npm run build            # Compile TypeScript to JS
npm run lint             # Lint TypeScript
npm run deploy           # Deploy functions to Firebase
```

## Architecture

### Entry Point & Auth Flow
1. `lib/main.dart` — initializes Firebase, wraps app in `TenantContext` and `ModuleAccessController` providers
2. `lib/auth/auth_wrapper.dart` — handles Firebase Auth state, loads user profile from Firestore `users/{uid}`, extracts `companyId`, initializes tenant context
3. Users are auto-logged-out if `isActive == false` in Firestore

### Shell / Navigation
- `lib/shell/zoho_shell.dart` — main app container; routes between 100+ pages using a `ShellPage` enum (no GoRouter/named routes)
- `lib/shell/shell_page.dart` — the `ShellPage` enum; add entries here when adding new screens
- `lib/shell/shell_sidebar.dart` — collapsible left sidebar using `SidebarGroup` models
- Navigation is manual: `setState(() { activePage = ShellPage.xxx; })` inside the shell

### Module System
- `lib/core/modules/module_registry.dart` — defines all 21 modules with IDs, routes, icons
- `lib/core/modules/module_access_controller.dart` — loads enabled modules from Firestore `companies/{tenantId}/modules/{moduleId}`; new modules are initialized by `TenantModuleService.ensureTenantModulesInitialized()`
- Check module access in UI: `ModuleAccessProvider.of(context).isModuleEnabled(moduleId)`

### Tenancy & Firestore Scoping
- All Firestore data is scoped under `companies/{companyId}/{collection}`
- Use `TenantFirestore` (in `lib/core/tenancy/`) for all queries — it enforces tenant scoping automatically
- Never query Firestore root collections directly for business data

### Data Layer Pattern
Each module follows: **Model → Repository → Service → Screen**
- `*_model.dart` — Dart class with `fromFirestore()` / `toFirestore()` / `toMap()`
- `*_repository.dart` — direct Firestore access; `watchX()` returns Streams, `fetchX()` returns Futures
- `*_service.dart` — business logic, orchestrates repositories
- `screens_*.dart` or `*_screen.dart` — UI, reads from providers or calls services directly

### State Management
- Pure Provider/ChangeNotifier (no Riverpod, no BLoC)
- `ChangeNotifierProvider` wrapping at app or module level
- Providers expose `maybeOf()` / `of()` static context accessors

### PDF Generation
- `printing` + `pdf` packages used in sales and production modules for quotations and BOQ documents

### Cloud Functions
- TypeScript source in `functions/src/`; handles email (Nodemailer), XLSX exports, and admin operations

## Key Files

| File | Purpose |
|------|---------|
| `lib/main.dart` | App init, root providers |
| `lib/auth/auth_wrapper.dart` | Auth state + tenant setup |
| `lib/shell/zoho_shell.dart` | Main navigation shell (large) |
| `lib/shell/shell_page.dart` | Navigation enum |
| `lib/core/modules/module_registry.dart` | Module definitions |
| `lib/core/tenancy/tenant_context.dart` | Tenant management |
| `firestore.rules` | Security rules (`signedIn()`, `belongsToCompany()`, `isActive()`) |

## Known Constraints

- `use_build_context_synchronously: false` in `analysis_options.yaml` — async gaps after `await` must not use `context`; use `mounted` guards or pass data through instead
- The shell (`zoho_shell.dart`) is large; prefer adding new page rendering logic in separate widget files and referencing them from the shell switch statement
- There is no automated test suite; `flutter_test` is available but untested

---
> Source: [sarfarazbaig2004/saif](https://github.com/sarfarazbaig2004/saif) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
