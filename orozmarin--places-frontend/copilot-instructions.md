## places-frontend

> A Flutter app for saving, discovering, and reviewing restaurants/places. Users can manage personal place lists, rate locations, and discover nearby restaurants via Google Places.

# Places Frontend

A Flutter app for saving, discovering, and reviewing restaurants/places. Users can manage personal place lists, rate locations, and discover nearby restaurants via Google Places.

## Tech Stack

- **Flutter** 3.32.0 (managed via FVM — use `fvm flutter` or configure FVM globally)
- **Dart** >=3.1.0
- **State:** `async_redux` — Redux with async action support
- **Navigation:** `go_router` — Declarative routing with auth guards
- **HTTP:** `dio` with JWT interceptor
- **Serialization:** `json_serializable` + `copy_with_extension` (code generation)
- **Storage:** `shared_preferences` for token/user persistence
- **Config:** `flutter_dotenv` — `.env` file for API keys

## Key Directories

| Path | Purpose |
|------|---------|
| `lib/models/` | Immutable data models (JSON-serializable, copyWith) |
| `lib/store/` | Redux state — `AppState`, actions, reducers |
| `lib/screens/` | UI pages: `*_page.dart` = StoreConnector wrapper, companion file = UI |
| `lib/widgets/` | Reusable widgets (AppBar, navigation scaffold, cards) |
| `lib/service/` | Business logic managers (`AuthManager`, `PlaceManager`, `ApiService`) |
| `lib/http/` | Dio interceptors and token storage helpers |
| `lib/tools/` | Stateless utilities (`LocationHelper`, `ToastHelper`, `UserHelper`) |
| `lib/theme/` | `MyColors` constants and Material theme config |
| `lib/router.dart` | All route definitions and auth redirect logic |
| `lib/store/app_state.dart` | Root Redux state composition |

## Essential Commands

```bash
# Install dependencies
flutter pub get

# Run code generation (required after model changes)
flutter pub run build_runner build --delete-conflicting-outputs

# Watch mode during development
flutter pub run build_runner watch --delete-conflicting-outputs

# Run app (Android emulator — backend expects localhost at 10.0.2.2)
flutter run

# Run tests
flutter test

# Analyze
flutter analyze
```

> **Note:** The backend base URL is hardcoded as `http://10.0.2.2:8080/rest` (Android emulator localhost). Update `lib/service/api_service.dart` for other targets.

## Related Projects

| Project | Path |
|---------|------|
| Backend (Spring Boot 3 + MongoDB) | `/Users/marinoroz/Documents/OrozDigital/places-backend/` |

When a task involves both frontend and backend changes (e.g., new API endpoint + consuming it in Flutter), read and edit files in both projects freely.

## Sub-agents

Project sub-agents live in `.claude/agents/`. For multi-file feature work, delegate to the appropriate agent:

| Agent | When to use |
|-------|-------------|
| `auth-agent` | JWT, Dio interceptors, Firebase/Google sign-in, token storage, login screens |
| `places-agent` | Place CRUD, ratings, visits, Google Places API, geolocation |
| `social-agent` | Friendships, friend requests, visit invitations, co-visitors |
| `state-agent` | Redux actions, AppState, ViewModel factories (`*_page.dart`) |
| `ui-agent` | Screens UI, widgets, theme, GoRouter, extensions |

**When to delegate:** Use sub-agents for features that touch 3+ files within one domain. For single-file edits or quick questions, handle directly.

## Additional Documentation

Check these files when working on related tasks:

- `.claude/docs/architectural_patterns.md` — Redux action lifecycle, Page/Screen split, ViewModel factory, Manager pattern, router guards

## Git Permissions

Claude may perform the following git operations without asking for confirmation:

- `git status`, `git log`, `git diff`, `git show` — read-only inspection
- `git add`, `git commit` — staging and committing changes
- `git push` — pushing to remote (including `-u` for new branches)
- `git checkout`, `git switch`, `git branch` — branch creation and switching
- `git pull`, `git fetch` — syncing with remote
- `git rebase` (non-interactive) — rebasing branches
- `gh pr create` — opening pull requests

**Requires user confirmation:**
- Merging pull requests (`gh pr merge`) — user must review and merge PRs themselves
- Force push (`git push --force`) — always confirm first
- Deleting branches (`git branch -D`, `git push origin --delete`)
- `git reset --hard` or any destructive rewrite of committed history

## Memory System

Project memory lives in `~/.claude/projects/-Users-marinoroz-Documents-OrozDigital-places-frontend/memory/`.

**`MEMORY.md`** is always loaded — it's a short index of pointers to individual memory files.

Rules for saving memory:
- **Feedback** (`feedback_*.md`) — whenever something breaks in a non-obvious way or a workaround is needed. These are the most valuable — they prevent repeating mistakes.
- **Project** (`project_*.md`) — active features, infra state, business reasons behind architectural decisions. Update when state changes.
- **Do NOT save**: code patterns, file paths, architecture (derivable from codebase), git history (belongs in commits), in-progress task state (belongs in tasks).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/orozmarin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
