## sagex

> Android app that listens to WeChat notifications, classifies conversations by group/person, and produces AI summaries via an OpenAI-compatible API.

# SageX

Android app that listens to WeChat notifications, classifies conversations by group/person, and produces AI summaries via an OpenAI-compatible API.

## Build & run

- **JDK**: use JDK 21 (Android Studio's bundled JBR). The system JDK may be too new for AGP.
  ```bash
  export JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home"
  ./gradlew assembleDebug          # build debug APK
  ./gradlew testDebugUnitTest      # run JVM unit tests (incl. Robolectric DAO tests)
  ```
- Toolchain: Kotlin 2.0 + Jetpack Compose, AGP 8.7.3, Gradle 8.11.1. `minSdk 26`, `targetSdk/compileSdk 35`.
- `local.properties` (git-ignored) points at the Android SDK.

## Architecture

- **Language/UI**: Kotlin, Jetpack Compose (Material 3), Hilt DI, Room, Coroutines/Flow.
- **Packages** (`com.tiswu.sagex`):
  - `domain/` — pure model + `NotificationParser` interface + `ParseResult`; no Android deps, unit-testable.
  - `notification/` — `SageXNotificationListenerService` (NLS), `WeChatNotificationParser`, `NotificationAccessHelper`.
  - `data/` — `MessageRepository`; `data/local/` Room entities, DAO, `SageXDatabase`.
  - `di/` — Hilt modules (DB, dispatchers, parser multibinding).
  - `ui/home/` — Compose home screen + ViewModel.
- **Capture model**: notifications are the only sanctioned source (no chat-history API). The system keeps the NLS alive while "Notification access" is granted. Aggregated/muted notifications are an inherent limitation.
- **Conversation identity**: `ConversationKey = SHA-256(sourcePackage | type | title)` — keeps a group and a direct chat with the same display name distinct.
- **Multi-source extension point**: parsers are bound `@IntoSet`; WeChat is the only one today. Adding a source = new `NotificationParser` impl + `@IntoSet` binding.

## Spec-driven development (OpenSpec)

This project uses [OpenSpec](https://github.com/Fission-AI/OpenSpec). All features go through a change proposal before implementation.

```bash
openspec list                              # active changes
openspec show <change>                     # view a change
openspec validate <change> --strict        # validate artifacts
openspec archive <change>                  # archive after merge
```

Workflow per change: `proposal.md → design.md → specs/ → tasks.md` → implement (tick `tasks.md`) → tests green → archive. Slash commands: `/opsx:propose`, `/opsx:apply`, `/opsx:archive`.

### Roadmap (changes)

1. `add-notification-capture` — NLS + parse + Room storage + permission/status UI. ✅ done
2. `add-background-resilience` — foreground service (specialUse), boot receiver, battery/autostart whitelist. ✅ done
3. `add-conversation-list` — group/person conversation list + detail + navigation. ✅ done
4. `add-ai-summary` — OpenAI-compatible client + settings + manual summarize. ✅ done
5. `design-extensions` (design only) — system-share export, multi-app sources. ✅ designed (impl deferred)

Future (scoped in `design-extensions`): `add-summary-sharing`, `add-source-<app>`.

---
> Source: [whtis/sagex](https://github.com/whtis/sagex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-28 -->
