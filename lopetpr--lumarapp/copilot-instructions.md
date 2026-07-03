## lumarapp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
# Debug build
./gradlew assembleDebug

# Release build
./gradlew assembleRelease

# Unit tests
./gradlew test

# Instrumented tests (requires device/emulator)
./gradlew connectedAndroidTest

# Single unit test class
./gradlew testDebugUnitTest --tests "com.example.lumarapp.ExampleUnitTest"

# Lint
./gradlew lintDebug
```

## Architecture

Single-module Android app. Kotlin + Jetpack Compose. Navigation via `androidx.navigation:navigation-compose`.

**Package layout** — feature-first + Clean Architecture layers:

```
com.example.lumarapp
├── MainActivity.kt
├── LumarApplication.kt
├── core/
│   ├── navigation/             # Graph.kt (route constants), RootNavGraph.kt
│   └── ui/
│       ├── components/         # DefaultTextField, TopBar, AdminBottomBar, AdminDrawer
│       └── theme/              # Color.kt, Theme.kt, Type.kt
├── di/                         # AppModule.kt, RepositoryModule.kt (Hilt)
├── auth/
│   ├── data/
│   │   ├── local/              # TokenManager (DataStore)
│   │   ├── remote/             # AuthApiService, AuthInterceptor, LoginRequest, LoginResponse
│   │   └── repository/         # AuthRepositoryImpl (maps DTO → domain model)
│   ├── domain/
│   │   ├── model/              # User (domain model — no DTO leakage)
│   │   ├── repository/         # AuthRepository interface (returns User)
│   │   └── usecase/            # LoginUseCase, CheckSessionUseCase, LogoutUseCase
│   └── presentation/
│       ├── navigation/         # AuthScreen (sealed routes), AuthNavGraph
│       └── login/
│           ├── LoginScreen.kt, LoginViewModel.kt, LoginUiState.kt
│           └── components/LoginContent.kt
├── home/admin/
│   ├── presentation/navigation/ # AdminScreen (sealed routes), AdminNavGraph
│   ├── presentacion/           # AdminHomeScreen (Scaffold + Drawer + inner NavHost)
│   └── categorias/             # Full CRUD module (see below)
└── splash/
```

**Categorías module** (reference for new CRUD modules):
```
home/admin/categorias/
├── data/
│   ├── remote/         # CategoriaService, CategoriaRequest, CategoriaResponse
│   └── repository/     # CategoriaRepositoryImpl (CRUD + Spanish error mapping)
├── domain/
│   ├── model/          # Categoria
│   ├── repository/     # CategoriaRepository interface
│   └── usecase/        # GetCategoriasUseCase, CreateCategoriaUseCase, UpdateCategoriaUseCase, DeleteCategoriaUseCase
└── presentation/
    ├── CategoriaScreen.kt, CategoriaViewModel.kt, CategoriaUiState.kt
    └── components/     # CategoriaContent, CategoriaSheetContent, DeleteCategoriaDialog
```

**Navigation pattern**: `RootNavGraph` owns the outer `NavHost`. `AdminHomeScreen` owns an inner `NavHost` for admin sub-screens (Dashboard, Categorías, Productos, etc.). Routes live in sealed classes per feature (`AuthScreen`, `AdminScreen`); graph-level IDs in `Graph`.

**Admin shell**: `AdminHomeScreen` provides `Scaffold` + `TopBar` + `ModalNavigationDrawer` + `AdminBottomBar`. Sub-screens render inside its inner `NavHost` and receive `paddingValues`. TopBar subtitle is route-dependent via `topBarInfo()`. For dynamic subtitles (e.g., category counts), use `onSubtitleUpdate` callback pattern from child screen → AdminHomeScreen.

**Composable split**: Screen owns ViewModel + side-effects. Content is stateless — receives `state` + lambdas. No `hiltViewModel()` in Content.

**State management**: ViewModel + `StateFlow<UiState>` + `collectAsStateWithLifecycle()`. Computed properties in UiState via `get()` (e.g., `val totalCategorias get() = categorias.size`).

**Clean Architecture flow**: `presentation` → `domain` (use cases + interfaces) → `data` (implementations). `domain` layer has zero Android dependencies. One UseCase per action with `operator fun invoke()`.

**CRUD pattern** (established in categorías):
- BottomSheet for create/edit (`ModalBottomSheet` with `containerColor = Color.White`)
- AlertDialog for delete confirmation
- Repository maps HTTP errors to Spanish messages
- ViewModel manages sheet/dialog visibility via UiState flags

**Shared components**: `DefaultTextField` for form fields, `TopBar`, `AdminBottomBar`, `AdminDrawer`.

**Theme colors** (use these, don't hardcode hex):
- `Blues` — primary blue `#1F51DC`
- `Gray800 / Gray500 / Gray300 / Gray100` — text and border grays
- `RedSoft / RedText` — delete actions
- `GreenSoft / GreenBg` — success states
- `BlueSoft / BlueSoftBorder` — info states

**UI rules**: Background `Color.White` on screens. Strings in Spanish. Error messages in Spanish.

## Adding a New Feature (Top-Level Graph)

1. Create `<feature>/presentation/navigation/screen/<Feature>Screen.kt` — sealed class with routes.
2. Create `<feature>/presentation/navigation/graph/<Feature>NavGraph.kt` — `NavGraphBuilder` extension.
3. Add graph route constant to `Graph.kt`.
4. Nest the new graph inside `RootNavGraph`.

## Adding a New Admin Sub-Module (CRUD)

1. Add route to `AdminScreen` sealed class.
2. Create module folder: `home/admin/<module>/` with `data/`, `domain/`, `presentation/` layers.
3. Create `data/remote/`: Service (Retrofit), Request/Response DTOs.
4. Create `domain/`: Model, Repository interface, UseCases (Get, Create, Update, Delete).
5. Create `data/repository/`: RepositoryImpl with CRUD + Spanish error mapping.
6. Create `presentation/`: UiState, ViewModel, Screen, `components/` (Content, SheetContent, DeleteDialog).
7. Register DI: `AppModule.kt` (provide Service), `RepositoryModule.kt` (bind Repository).
8. Wire in `AdminHomeScreen`: add `composable()` in inner NavHost → call Screen with `paddingValues`.
9. See `.claude/rules-modulo.md` for detailed conventions and checklist.

---
> Source: [lopetpr/LumarApp](https://github.com/lopetpr/LumarApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
