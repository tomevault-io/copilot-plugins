## bahut

> **Bahut** is a Flutter mobile app that connects to the **École Directe** API (French education platform) to let students visualize their grades, schedule, homework, and school behavior.

# Bahut — Claude Code Context

## Project Overview

**Bahut** is a Flutter mobile app that connects to the **École Directe** API (French education platform) to let students visualize their grades, schedule, homework, and school behavior.

- **Package:** `com.frhamon.calcul_moyenne`
- **Version:** 1.4.0+9
- **Platforms:** iOS 14+, Android SDK 21+, Web, macOS, Linux, Windows
- **Language:** French throughout (UI strings, comments, commit messages)
- **Dart SDK:** ^3.10.7

---

## Architecture

Feature-based modular architecture with clean separation:

```
lib/
├── core/               # Shared infrastructure
│   ├── accessibility/  # HIG compliance, haptic feedback, min touch targets
│   ├── constants/      # API endpoints, app constants
│   ├── demo/           # Demo mode data (Play Store: demo@bahut.app / demo2024)
│   ├── errors/         # Freezed union Failure type
│   ├── network/        # Dio API client with cookie/token management
│   ├── services/       # background sync, notifications, calendar, widgets, etc.
│   └── theme/          # 7 color palettes + auto-switching by time
├── features/           # 16 feature modules (each has data/domain/presentation)
├── shared/             # Shared widgets, animations, navigation
├── router/             # GoRouter config
├── app.dart            # Root widget
└── main.dart           # Entry point + service initialization
```

Each feature follows: `data/models/` → `presentation/providers/` → `presentation/screens/` → `presentation/widgets/`.

---

## State Management

**Flutter Riverpod 2.x** with `StateNotifierProvider`. Key providers:

| Provider | File | Purpose |
|---|---|---|
| `authStateProvider` | `features/auth/presentation/providers/auth_provider.dart` | Auth, MFA, biometric, multi-account |
| `gradesStateProvider` | `features/grades/presentation/providers/grades_provider.dart` | Grades, filtering, weighted averages |
| `homeworkStateProvider` | `features/homework/presentation/providers/homework_provider.dart` | Homework list + status |
| `scheduleStateProvider` | `features/schedule/presentation/providers/schedule_provider.dart` | Schedule data |
| `statisticsProvider` | `features/statistics/presentation/providers/statistics_provider.dart` | Computed analytics |
| `badgesProvider` | `features/gamification/presentation/providers/badges_provider.dart` | Badge unlock logic |
| `themeModeProvider` | `core/theme/theme_provider.dart` | Light/dark/system |
| `appThemeTypeProvider` | `core/theme/theme_provider.dart` | Color palette selection |
| `autoThemeConfigProvider` | `core/theme/theme_provider.dart` | Time-based auto-switching |

---

## Navigation

**GoRouter v14.6.2** — `router/app_router.dart`

Auth redirect logic: loading → login → qcm (MFA) → biometric → children (multi-account) → dashboard

ShellRoute with bottom nav (5 tabs: Dashboard, Grades, Schedule, Homework, Profile):
- `/dashboard`, `/grades`, `/schedule`, `/homework`, `/vie-scolaire`
- `/averages`, `/profile`, `/settings`, `/statistics`, `/simulation`, `/badges`

---

## API — École Directe

**Base URL:** `https://api.ecoledirecte.com/v3/`

Auth flow:
1. GET GTK cookie from `/login.awp?gtk=1`
2. POST credentials → receive `X-Token` JWT header
3. All subsequent requests include `X-Token`

Key endpoints:
- `/v3/login.awp` — Authentication
- `/v3/eleves/{id}/notes.awp` — Grades
- `/v3/eleves/{id}/vieScolaire.awp` — School behavior
- `/v3/E/{id}/emploidutemps.awp` — Schedule
- `/v3/Eleves/{id}/cahierdetexte.awp` — Homework

API client: `core/network/api_client.dart` (Dio + cookie jar + token interceptor)

---

## Data Models

All models use **@freezed** annotation (immutable, copyWith, JSON serialization, pattern matching).

- `GradeModel` — with extensions: `valeurDouble`, `valeurSur20`, `isValidForCalculation`, `mainSubjectName`
- `UserModel` / `ChildModel`
- `HomeworkModel` / `CahierDeTexteJour`
- `ScheduleModel`
- `VieScolaireModel`
- `Failure` (union: server / network / auth / cache / biometric / unknown)

Generated files (`*.freezed.dart`, `*.g.dart`) — run `flutter pub run build_runner build` after model changes.

---

## Key Features

1. **Multi-account** — Switch between student accounts; biometric unlock
2. **Grades** — Weighted averages, period filtering, class comparisons, color-coded indicators
3. **Statistics** — fl_chart visualizations, 30-day trends, goal tracking
4. **Gamification** — 15+ badges with celebration animations
5. **Schedule** — Calendar sync with device (device_calendar)
6. **Homework** — Due dates, completion status
7. **Home screen widgets** (Android) — Average, homework, schedule widgets (Kotlin: `HomeworkWidgetProvider.kt`, `AverageWidgetProvider.kt`, `ScheduleWidgetProvider.kt`)
8. **Quick actions** — 3D Touch (iOS) / long press (Android)
9. **iOS Spotlight** — Grade search integration
10. **Background sync** — Workmanager, configurable 15 min–24 hours, new grade notifications
11. **7 themes** — Classique, Océan, Forêt, Coucher de soleil, Lavande, Rose, Minuit; auto-switch by time
12. **Demo mode** — `demo@bahut.app` / `demo2024` for Play Store review

---

## Theming

- `core/theme/app_themes.dart` — 7 theme definitions
- `core/theme/chanel_colors.dart` — Custom palette
- `core/theme/chanel_typography.dart` — Typography system
- Grade colors: success (≥14), info (≥10), warning (≥8), error (<8)

---

## Services (`core/services/`)

| Service | Purpose |
|---|---|
| `background_sync_service.dart` | Workmanager periodic sync |
| `notification_service.dart` | Flutter Local Notifications + Android channels |
| `calendar_service.dart` | Device calendar read/write |
| `home_widget_service.dart` | Home screen widget updates |
| `quick_actions_service.dart` | Quick action callbacks |
| `spotlight_service.dart` | iOS Spotlight indexing |
| `system_integration_service.dart` | Coordinator for all above |
| `connectivity_service.dart` | Network status |
| `haptic_service.dart` | HIG-compliant haptic feedback |

---

## Accessibility

- Minimum 44pt touch targets (`AccessibleButton`, `MinimumTouchTarget`)
- Semantic labels for screen readers
- Dynamic Type: text scaling 0.8×–1.5×
- Haptic feedback (iOS only): light/medium/heavy impacts, selection clicks
- HIG compliance utilities: `core/accessibility/accessibility_utils.dart`

---

## Naming Conventions

- Features: `lowercase_underscore` (e.g., `vie_scolaire`, `calendar_sync`)
- Providers: `StateNotifierProvider<NotifierClass, StateClass>`
- Models: suffix `Model`
- Screens: suffix `Screen`
- Services: suffix `Service`
- Extensions: descriptive (e.g., `GradeModelExtension`)

---

## Code Generation

After modifying any `@freezed` model or `@riverpod` provider:

```bash
flutter pub run build_runner build --delete-conflicting-outputs
```

---

## Testing

Minimal coverage. Only `test/widget_test.dart` exists. `mocktail` is available for mocking.

---

## Recent History (as of v1.4.0)

- v1.4.0 — Détail des devoirs, nouvelle icône, fix badge Progression, badges rétroactifs
- v1.3.0 — Carte moyenne swipeable, bannière nouveau trimestre, fix tri notes, fix badges streak
- v1.2.2 — Multi-accounts, badges, router fixes
- v1.2.1 — Notification fixes, notification test button
- v1.2.0 — Background notifications, biometric fixes
- v1.1.0 — Gamification, advanced statistics, unified UI
- v1.0.0 — Initial release

---

## Changements en cours (non versionnés, mars 2026)

### Dashboard — `features/dashboard/presentation/screens/dashboard_screen.dart`
- **Bannière nouveau trimestre** : quand `filteredGrades.isEmpty && !isLoading && periods.isNotEmpty`, affiche `_NewPeriodBanner` avec le nom de la période et le message "Aucune note disponible pour le moment"
- **Carte moyenne swipeable** : `_AverageCard` détecte `onHorizontalDragEnd` (seuil 200 px/s) pour naviguer entre trimestres. Indicateurs visuels : points animés + flèches gauche/droite. Helpers dans `_DashboardScreenState` : `_currentPeriodIndex`, `_previousPeriodIndex`, `_nextPeriodIndex`
- **Tri des dernières notes** : utilise `filteredGrades` (période active) triées par `dateSaisieTime` desc + `id` desc en tiebreaker (plus fiable que `dateTime` qui est la date d'examen)

### Badges — `features/gamification/presentation/providers/badges_provider.dart`
- **Bug corrigé** : `break` remplacé par suppression du check sur `dateTime == null` — une note sans date ne coupait plus le comptage des notes consécutives
- **Tri corrigé** : tri par `dateSaisie` + `id` au lieu de `dateTime` seul, dans `badgeContextProvider` et `_updateConsecutiveGrades`
- Ces corrections débloquent les badges de régularité (`streak_3`, `streak_5`, `streak_10`) qui ne se validaient pas malgré 20+ notes ≥12 consécutives

---

## Slash Commands personnalisées

- **`/release`** — `.claude/commands/release.md` : bump de version, mise à jour `pubspec.yaml` + `CHANGELOG.md` + `CLAUDE.md`, commit, puis `flutter build appbundle --release`

---

## Publication Play Store — État (mars 2026)

### Fait
- [x] Build release AAB généré : `build/app/outputs/bundle/release/app-release.aab` (48.2 MB)
- [x] Permissions ajoutées au manifest : `POST_NOTIFICATIONS`, `RECEIVE_BOOT_COMPLETED`, `VIBRATE`
- [x] Version test interne créée dans la Play Console (statut "Non examinée" → en attente review Google)
- [x] Politique de confidentialité créée : `docs/privacy-policy.html` → https://menur4.github.io/bahut/privacy-policy
  - GitHub Pages à activer : repo → Settings → Pages → branch `main` / folder `/docs`
- [x] Icône Play Store 512×512 : `assets/icone-512.png`
- [x] Feature graphic 1024×500 : `assets/feature-graphic.png`
- [x] Textes fiche Play Store rédigés (titre, description courte, description longue)
- [x] Tags : Éducation, Outils scolaires, Productivité, Agenda et planning, Notes et mémos
- [x] Section "Accès à l'application" : compte démo `demo@bahut.app` / `demo2024`

### Reste à faire dans la Play Console
- [ ] Activer GitHub Pages pour rendre la politique de confidentialité accessible
- [ ] Uploader `assets/icone-512.png` dans la fiche
- [ ] Uploader `assets/feature-graphic.png` dans la fiche
- [ ] Remplir la fiche principale (titre, descriptions)
- [ ] Section "Sécurité des données" — credentials chiffrés, pas de partage tiers
- [ ] Section "Classification du contenu" — questionnaire
- [ ] Ajouter testeurs (liste Gmails) + partager le lien d'opt-in
- [ ] Screenshots téléphone (min 2) et tablette (min 1)

### applicationId
`com.famille.bahut` — défini dans `android/app/build.gradle.kts:33` (définitif une fois publié)

---

## MCP Canva

Serveur configuré dans `~/.claude.json` (scope user) :
```
npx -y @canva/cli@latest mcp
```

**Pour activer :** lancer `npx @canva/cli@latest login` dans un terminal séparé (ouvre le navigateur pour OAuth), puis relancer Claude Code. Une fois connecté, Canva MCP permet de créer des designs directement depuis la conversation (icônes, banners, screenshots).

---

## Android Native (Kotlin)

`android/app/src/main/kotlin/com/frhamon/calcul_moyenne/`
- `MainActivity.kt`
- `HomeworkWidgetProvider.kt`
- `AverageWidgetProvider.kt`
- `ScheduleWidgetProvider.kt`

---

## Key Dependencies

| Package | Version | Purpose |
|---|---|---|
| `flutter_riverpod` | ^2.6.1 | State management |
| `go_router` | ^14.6.2 | Navigation |
| `dio` | ^5.7.0 | HTTP client |
| `freezed` | ^2.5.7 | Immutable models |
| `flutter_secure_storage` | ^10.0.0 | Secure credential storage |
| `local_auth` | ^2.3.0 | Biometric auth |
| `workmanager` | ^0.9.0 | Background tasks |
| `flutter_local_notifications` | ^18.0.1 | Notifications |
| `fl_chart` | ^1.0.0 | Charts/statistics |
| `device_calendar` | ^4.3.3 | Calendar sync |
| `home_widget` | ^0.7.0 | Android home widgets |
| `flutter_platform_widgets` | ^7.0.1 | Adaptive Material/Cupertino |
| `intl` | ^0.20.0 | French date formatting |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/menur4) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
