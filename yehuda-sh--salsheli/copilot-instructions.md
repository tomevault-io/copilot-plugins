## salsheli

> אפליקציית ניהול קניות משפחתית חכמה עם עיצוב **Notebook + Sticky Notes**.

# MemoZap

## Project Overview

אפליקציית ניהול קניות משפחתית חכמה עם עיצוב **Notebook + Sticky Notes**.

**פיצ'רים עיקריים:**
- רשימות קניות משותפות בזמן אמת (9 סוגים)
- מזווה דיגיטלי עם התראות מלאי נמוך
- שיתוף רשימות עם תפקידים (Owner/Admin/Editor/Viewer)
- קנייה משותפת עם סנכרון בזמן אמת
- היסטוריית קניות וסיכומים
- יומן פעילות משפחתי (Activity Log)

**Package:** `com.memozap.app`

---

## Tech Stack

- **Framework:** Flutter 3.38+ / Dart 3.8.1+
- **Backend:** Firebase (Auth, Firestore, Storage, Analytics, Crashlytics, Messaging)
- **State:** Provider + ChangeNotifier
- **UI:** Hebrew RTL first, Material 3, Dark Mode

---

## Design System

| Token | Values |
|-------|--------|
| Spacing | `kSpacing*` via `ui_constants.dart` (XTiny=4, Tiny=6, Small=8, SmallPlus=12, Medium=16, Large=24, XLarge=32) |
| Border Radius | `kBorderRadiusSmall(8)`, `Default(12)`, `Large(16)`, `XLarge(24)` |
| Typography | `kFontSizeTiny(10)` → `kFontSizeDisplay(34)` — 8 sizes |
| Icons | `kIconSizeSmall(16)`, `SmallPlus(20)`, `Medium(24)`, `MediumPlus(28)`, `Large(36)`, `XLarge(48)`, `XXLarge(64)` |
| Colors | **Theme only** — `Theme.of(context).colorScheme` |
| Background | `NotebookBackground()` on all 21 screens |
| Imports | **Relative** in lib/ (`../../core/...`) — `package:memozap/` only in `main.dart` |

**Rules:**
- ❌ No `Colors.xxx` (except `Colors.transparent`)
- ❌ No hardcoded `fontSize:` — use `kFontSize*` constants
- ❌ No hardcoded `BorderRadius` — use `kBorderRadius*` constants
- ❌ No hardcoded icon `size:` — use `kIconSize*` constants
- ❌ No hardcoded spacing/padding — use `kSpacing*` constants
- ❌ No hardcoded Hebrew strings — use `AppStrings`
- ✅ Wrap `HapticFeedback.*()` calls with `unawaited()`
- ✅ Add `const` to `SizedBox`, `EdgeInsets`, `Duration`, `Icon` where possible

---

## Project Guardrails (DO NOT CHANGE WITHOUT EXPLICIT APPROVAL)

### Review Workflow
- 3-5 שאלות הבהרה עם אפשרויות א/ב/ג
- **בלי קטעי קוד בשאלות**
- בכל שאלה להוסיף **"המלצת הסוכן"**

### Welcome Screen
- מופיע **רק** עד יצירת חשבון
- **Logout רגיל לא מחזיר** Welcome
- **מחיקת נתונים כן מחזירה** Welcome

### seenOnboarding
- נדלק אחרי login/register הצלחה (כולל Google/Apple)
- **נשמר אחרי Logout** (לא מתאפס)

### Auth Screens
- **נקיים, לא Sticky Notes** — עיצוב מינימליסטי
- `NotebookBackground.subtle()` **מותר** (רקע עדין בלבד, לא full notebook)

### Pending Invites
- בדיקה אחרי register/login (כולל Google/Apple)
- guard אם אין email/phone

### IDs/Keys (Config)
- כל ID/key חייב **resolve()** עם fallback:
  - **"other"** — למשתמש (UI-safe)
  - **"unknown"** — לדיבאג בלבד
- **ensureValid()** בקונפיגים (via `ConfigValidation` mixin)
- **Backward compatible** — aliases לערכים ישנים

### AnimatedButton
- **אפקט בלבד** — הפעולה ב-parent
- haptic רק ל-CTA (לא לניווט)
- scale: **0.97–0.98**
- אנימציה מתחילה ב-**tap-down**

---

## Key Commands

```bash
dart analyze lib/    # חובה לפני כל commit
flutter test         # חובה לפני כל PR
flutter run          # הרצה
```

---

## Key Files

| תחום | קבצים |
|------|-------|
| **Theme** | `lib/theme/app_theme.dart` (AppBrand extension, light/dark, dynamic colors) |
| **Constants** | `lib/core/ui_constants.dart`, `status_colors.dart` |
| **Config** | `lib/config/list_types_config.dart`, `filters_config.dart`, `storage_locations_config.dart` |
| **Models** | `lib/models/shopping_list.dart`, `unified_list_item.dart`, `user_entity.dart`, `inventory_item.dart`, `activity_event.dart` |
| **Providers** | `lib/providers/user_context.dart`, `shopping_lists_provider.dart`, `inventory_provider.dart`, `activity_log_provider.dart` |
| **Shared Widgets** | `lib/widgets/common/` — NotebookBackground, StickyNote, StickyButton, AppErrorState, AppLoadingSkeleton, AnimatedButton, TappableCard, OfflineBanner, SectionHeader, AppDialog, SkeletonLoader, BarcodeScannerSheet |
| **Strings** | `lib/l10n/app_strings.dart` |
| **Security** | `firestore.rules` (v4.4), `firestore.indexes.json` |

---

## Dependency-First Ordering

סדר עבודה:
1. Model / Schema
2. Repository / Service
3. Provider / State
4. UI / Widgets
5. Strings (AppStrings)
6. Tests & analyze

---

## Known Issues

- ~~**B3:** SavedContactsService בולע שגיאות~~ ✅ תוקן — כל 3 המתודות עושות rethrow
- **W1:** `use_build_context_synchronously` (2) ב-settings_screen (יש `mounted` guards — ממתין לאימות analyzer)
- **W2:** `directives_ordering` infos בכמה קבצים (pre-existing, cosmetic)
- ~~**W3:** `deprecated_member_use` — RadioListTile~~ ✅ תוקן (סשן 4)

See [CODE_REVIEW.md](CODE_REVIEW.md) for full status.

---

## Related Docs

- [CODE_REVIEW.md](CODE_REVIEW.md) — דוח Code Review מלא (עודכן 9/4/2026)
- [TEST_PLAN.md](TEST_PLAN.md) — תוכנית בדיקות (396 unit tests)
- [DESIGN_AUDIT.md](DESIGN_AUDIT.md) — סקירת עיצוב UI
- [docs/REFACTOR_PLAN.md](docs/REFACTOR_PLAN.md) — תוכנית ריפקטור 10 שלבים
- [docs/store-listing.md](docs/store-listing.md) — תוכן Store listing
- [docs/spec-home-screen.md](docs/spec-home-screen.md) — אפיון מסך הבית (כולל Activity Feed)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Yehuda-sh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
