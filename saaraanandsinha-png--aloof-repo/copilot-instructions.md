## aloof-repo

> This is an Android app (`saara app`) that reads WhatsApp notifications and uses an on-device SLM (Small Language Model via MediaPipe/Gemma) to extract and manage reminders. It features a Compose UI with Today, Week, Calendar, Chat, and Settings screens.

# Agent Instructions

## Project Overview
This is an Android app (`saara app`) that reads WhatsApp notifications and uses an on-device SLM (Small Language Model via MediaPipe/Gemma) to extract and manage reminders. It features a Compose UI with Today, Week, Calendar, Chat, and Settings screens.

## Tech Stack
- **Language:** Kotlin
- **UI:** Jetpack Compose + Material3
- **Architecture:** MVVM (ViewModel + StateFlow)
- **Database:** Room (with KSP)
- **Navigation:** Jetpack Navigation Compose
- **On-device AI:** MediaPipe Tasks GenAI (`tasks-genai:0.10.14`) + LlamaTik
- **Serialization:** kotlinx.serialization
- **Storage:** DataStore Preferences
- **Min SDK:** 26 | **Target SDK:** 36
- **Build:** Gradle with Kotlin DSL

## Project Structure
```
app/src/main/java/com/example/saaraapp/
├── ui/
│   ├── screens/        # Compose screens (Today, Week, Calendar, Chat, Settings)
│   ├── components/     # Shared Compose components
│   ├── navigation/     # NavGraph
│   └── theme/          # Colors, Typography, Theme
├── *ViewModel.kt       # MVVM ViewModels
├── *Repository.kt      # Data repositories
├── *Dao.kt / *Database.kt / *Entity.kt  # Room DB layer
├── FunctionGemmaHelper.kt  # On-device LLM inference
├── WhatsAppNotificationService.kt  # Notification listener
├── ModelDownloadManager.kt  # SLM model download
└── Utils.kt / DateParser.kt / KeywordExtractor.kt
```

## Git Commits
Always use **Conventional Commits** format for all commits:

```
<type>(<scope>): <short description>
```

### Types:
- `feat` – a new feature
- `fix` – a bug fix
- `chore` – dependency updates, config changes
- `refactor` – code change that is not a fix or feature
- `style` – formatting, missing semicolons, etc
- `docs` – documentation changes
- `test` – adding or updating tests

### Examples:
```
feat(notifications): add WhatsApp notification reader
fix(ui): correct AM/PM locale display issue
chore(deps): update Room to 2.8.4 and KSP to 2.2.10-2.0.2
refactor(viewmodel): migrate from StateFlow to Room database
```

## Git Strategy
Always use **rebase** instead of merge commits:
- Never use `git merge` — use `git rebase` instead
- Never create merge commits
- Keep commit history clean and linear

```bash
# Instead of this ❌
git merge main

# Always do this ✅
git rebase main
```

---
> Source: [saaraanandsinha-png/aloof-repo](https://github.com/saaraanandsinha-png/aloof-repo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
