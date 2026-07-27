## flutter-hatcha-app

> Hatcha is a GenUI demo app: an AI-driven event planner where hosts create events via conversational interviews, configure guest-facing invite questions, and view response analytics on a dashboard. All AI intelligence lives in the Python `hatcha_agent` A2A server — the Flutter app contains no direct Gemini API calls.


# Architecture & Patterns

Hatcha is a GenUI demo app: an AI-driven event planner where hosts create events via conversational interviews, configure guest-facing invite questions, and view response analytics on a dashboard. All AI intelligence lives in the Python `hatcha_agent` A2A server — the Flutter app contains no direct Gemini API calls.

---

## 1. Two-Layer AI Rule

Every AI request routes through the Python `hatcha_agent` A2A server. No Gemini API keys in Flutter.

- **GenUI layer** (`ControllerConfig`): Widget-rendering. Python `RoutingExecutor` selects agent by message prefix + sticky session.
- **GeminiService layer** (`GeminiConversationConfig`): Text-only tasks. Sends `"System Instruction: {prompt}\n\n{message}"` → Python `TextProxyAgent` extracts prompt, proxies to Gemini.

---

## 2. Feature Sub-Directory Convention

```
lib/features/<feature>/
├── genui/          → ControllerConfig subclasses (A2A widget rendering)
├── generative/     → GeminiConversationConfig subclasses + services (text-only AI)
├── interviews/     → InterviewConfig factory functions (host_dashboard, configure_event)
└── widgets/        → Feature-specific Flutter widgets
```

Note: `interviews/` subdirectories belong to features that _use_ interviews (`host_dashboard`, `configure_event`), not the `interview` feature itself.

---

## 3. Service Locator Globals

Access via `import 'package:hatcha/service_locator.dart';`

| Global                  | Type                             | Purpose                                                             |
| ----------------------- | -------------------------------- | ------------------------------------------------------------------- |
| `context.styles`        | `AppStyle`                       | Design tokens (colors, text, spacing) — **use this, not `$styles`** |
| `context.colors`        | `AppColors`                      | Shorthand for `context.styles.colors`                               |
| `context.textStyles`    | `AppText`                        | Shorthand for `context.styles.text`                                 |
| `$theme`                | `ThemeController`                | Dark/light mode                                                     |
| `$palette`              | `PaletteController`              | Swappable color palette (brand, event themes)                       |
| `$app`                  | `AppLogic`                       | App initialization (`bootstrap`) and session reset (`clearSession`) |
| `$party`                | `PartyLogic`                     | Events, participants, modules, host dashboard state                 |
| `$contacts`             | `ContactsLogic`                  | Contact CRUD and lookup                                             |
| `$currentAccount`       | `CurrentAccountLogic`            | Current logged-in user, login/switch                                |
| `$notification`         | `NotificationLogic`              | Per-user notifications (invites, RSVP, unviewed badge)              |
| `$tokenUsage`           | `TokenUsageLogic`                | AI token usage tracking and persistence                             |
| `$sessionSettings`      | `SessionSettingsLogic`           | Persisted session-level user preferences                            |
| `$genUi`                | `GenUiLogic`                     | GenUI controller registry                                           |
| `$gemini`               | `GeminiServiceLogic`             | GeminiService registry                                              |
| `$digestService`        | `EventDigestService`             | Batched digest generation                                           |
| `$dashboardControllers` | `HostDashboardControllerManager` | Dashboard GenUI controller pool                                     |
| `$toast`                | `ToastLogic`                     | Toast notifications                                                 |
| `$networkPolicy`        | `NetworkPolicy`                  | Network configuration                                               |
| `$connectivity`         | `ConnectivityService`            | Connectivity monitoring                                             |

> **`$styles` is deprecated.** Use `context.styles` (via `AppStyleContext` extension on `BuildContext`) instead.

---

## 4. Interactive Widget Pattern

Form widgets exist in two interactive variants for different usage contexts:

| Tier | Directory | Example | Purpose |
|------|-----------|---------|---------||
| Interactive | `guest_invite/`, `interview/` | `TextInput`, `MultiSelect` | User fills out form (context-dependent, no prefix) |

The "Interactive" tier has no naming prefix — the same widget name (e.g. `TextInput`) exists in both `guest_invite/` and `interview/` directories with context-specific behavior.

---

## 5. Controller Lifecycle

`HostDashboardControllerManager` (`$dashboardControllers`) owns all module + response controllers:

- `getControllersForEvent(eventId)` — returns `DashboardControllers` for an event (creates if needed)
- `switchToEvent(newEventId)` — switch active event context
- `disposeControllersForEvent(eventId)` — safely disposes all controllers for an event
- `disposeAllControllers()` — disposes everything
- `GenUiController.dispose()` is **deferred** if an A2A request is in-flight

---

## 6. Data Binding

Widgets bind to the global `DataModel` via path references (`A2uiSchemas.*Reference()`). Subscribe in the `widgetBuilder`:

```dart
final notifier = context.dataContext.subscribe<T>(DataPath(path));
context.dataContext.update(DataPath(path), newValue);
```

**Guest invite rule**: All data paths MUST start with `guest.` — enforced by `GuestAnswerAgent` Python prompt.

---

## 7. InterviewConfig Pattern

Interview is a reusable module, not a screen. Features create configs with `title`, `action`, `initialContext`, `leadingWidgets`, `onConfirm`, and `newQuestionsCount`. The orchestrator (`OrchestratorConversationConfig`) drives the conversation; per-question `InterviewGenuiControllerConfig` instances render individual widgets.

---

## 8. EventDigestService

Fire-and-forget digest manager. `$digestService.generateDigest(eventId, summary)` for initial creation; `$digestService.updateDigest(eventId, content)` for incremental updates (rapid calls batched automatically, max 3 retries).

---

## 9. InviteModule.description Contract

Descriptions are **pure natural language** — no component-type prefixes. Downstream AI infers widget choice from semantic verbs:

- `"select any ... from: A, B, C"` → MultiSelect
- `"choose one from: A, B, C"` → RadioSelect
- `"rate ... from X to Y"` → SteppedSlider
- `"share / describe ..."` → TextInput

---

## 10. Module childId vs childIds

| Property   | Schema type                      | Card chrome                     |
| ---------- | -------------------------------- | ------------------------------- |
| `childId`  | `componentReference` (String)    | None — child renders raw        |
| `childIds` | `componentArrayReference` (list) | Self-wrapping `AppSquircleCard` |

---

## 11. Python Agent Architecture

- All agents registered in `agents/hatcha_agent/agent.py`
- Routing: `agent_executor.py` → `RoutingExecutor` (session-sticky by `contextId`)
- System prompts: `agents/hatcha_agent/prompts/*.py` (one file per agent)
- GenUI agents (`use_ui=True`) produce `<a2ui-json>...</a2ui-json>` blocks; text agents return plain text
- Local dev: `cd agents && make run` (ADK server on `:8080`)

---
> Source: [gskinnerTeam/flutter-hatcha-app](https://github.com/gskinnerTeam/flutter-hatcha-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
