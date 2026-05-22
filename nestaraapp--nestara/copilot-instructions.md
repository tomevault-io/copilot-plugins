## nestara

> > **Read this entire file at the start of every session.** It defines what we're building, how we build it, and the rules you must follow. When in doubt, re-read it. When the user's prompt seems to contradict this file, ask before deviating.

# CLAUDE.md — Nestara

> **Read this entire file at the start of every session.** It defines what we're building, how we build it, and the rules you must follow. When in doubt, re-read it. When the user's prompt seems to contradict this file, ask before deviating.

---

## 1. Project Overview

**Nestara** is a premium iOS family organization app. Tagline: *Where home life flows.*

It brings calm, warmth, and structure to daily home life — chores, calendars, routines, shared shopping, and a recognition system in one elegant space. The interface should feel like a well-designed home: warm, ordered, quiet enough to actually use every day.

The audience is families and busy households who want a beautiful, easy-to-use way to stay on top of chores, schedules, and routines without feeling overwhelmed.

The app is built with SwiftUI on iOS 17+, backed by Firebase (Auth, Firestore, Storage, Functions, Cloud Messaging). Cloud Functions are TypeScript. Email is sent via Resend.

The five tabs are:

1. **Home** — personalized dashboard
2. **Days** — calendar (events) · branded as "Nestara Days"
3. **Tasks** — recurring household responsibilities, point-earning, parent-assigned · branded as "Nestara Tasks"
4. **Lists** — multiple named to-do lists with items · branded as "Nestara Lists"
5. **Shop** — shared shopping list

**Circle** (family members, settings, invites) and **Recognition** (points, badges, leaderboard) live in the profile menu accessible from the avatar on Home.

Tasks and Lists are deliberately separate. They look different, behave differently, and the user should never wonder which one to use:

| Tasks | Lists |
|---|---|
| Recurring (daily/weekly/etc.) | One-off items |
| Earn points | No points |
| Only parents can create/assign | Anyone can create lists and items |
| Tied to a single member | Two kinds: "Yours" (private) or "Assigned" (shared with members) |
| May require a photo on completion | No photo proof |
| Have a due **time** (e.g., 8:00 PM) | Items have a due **date** (or none) |
| Templates: pick from a shared task library | Multiple named lists per family |
| Live in the Tasks tab | Live in the Lists tab, segmented Yours / Assigned |

---

## 2. Tech Stack

| Layer | Choice |
|---|---|
| Language | Swift 5.9+ |
| UI | SwiftUI (no UIKit unless absolutely necessary) |
| Min iOS | 17.0 |
| State | `@Observable` macro (iOS 17 Observation framework) |
| Concurrency | `async/await`, no completion handlers |
| Backend | Firebase (Auth, Firestore, Storage, Functions, Cloud Messaging) |
| Functions | TypeScript on Node 20 |
| Email | Resend |
| Calendar sync | EventKit (Apple), Google Calendar API |
| Custom fonts | Fraunces (serif), DM Sans (sans) — bundled |
| Package management | Swift Package Manager only — never CocoaPods |
| Linting | SwiftLint |
| Build/test tooling | XcodeBuildMCP for Claude Code |

---

## 3. Brand Identity

This section is critical. Nestara is a brand-driven app — design, voice, and naming choices matter as much as engineering.

### Personality

Comfortable · Warm · Premium · Calm · Helpful · Organized · Modern.

### Voice

- **Calm**, not pushy
- **Encouraging**, not corrective
- **Clean**, not cluttered
- **Polished**, not cold
- **Helpful**, not overly playful

### Microcopy rules

- Use sentence case. Never SHOUTING CAPS in body text.
- Avoid exclamation marks. The brand never gets excited at the user.
- Use "let's" for collaborative moments ("let's keep the day flowing").
- Avoid demanding imperatives. Soft suggestion is the default.
- No emoji in microcopy except in badge names and the activity feed.
- Never use "Tap" or "Click" — describe the action, not the gesture.

### Approved microcopy specimens

> "A few things are waiting — let's keep the day flowing."
> "Your home rhythm is looking good today."
> "Everything's in one place, so the day feels easier."
> "Keep the day moving."
> "Add to today."
> "Done for now."
> "A calm start to the day."
> "Your home is in flow."

When writing new microcopy, ask: would it sit alongside these without feeling jarring?

---

## 4. Visual Identity

### 4.1 Foundation colors

| Token | Hex | Use |
|---|---|---|
| `Color.brandNavy` | `#1F3247` | Primary text, structure, wordmark, dark surfaces |
| `Color.brandCream` | `#FAF6ED` | Default app background |
| `Color.brandGold` | `#C99A4F` | Single accent — section labels, points, "moments that matter" |
| `Color.brandSage` | `#8FA68B` | Quiet confirmation — completed checks, success states |
| `Color.brandClay` | `#B66256` | Quiet warning — "Missed" status pills only. Never decorative. |

Supporting tints:

| Token | Hex | Use |
|---|---|---|
| `Color.surfacePaper` | `#FFFCF5` | Card surfaces (slightly lighter than cream) |
| `Color.surfaceCreamSoft` | `#F4ECDA` | Sub-surfaces, segmented controls |
| `Color.lineSoft` | `#EFE6CF` | Hairline dividers between rows |
| `Color.line` | `#E5DBC5` | Card borders |
| `Color.slate` | `#6E7681` | Secondary text |
| `Color.slateSoft` | `#9CA3AC` | Tertiary text, disabled states |

### 4.2 Member palette (8 colors)

Members pick one at signup. Muted and quiet — they coexist on screen without competing with cream and navy.

| Token | Hex | Token string (in DB) |
|---|---|---|
| `Color.memberNavy` | `#1F3247` | `"navy"` |
| `Color.memberSage` | `#8FA68B` | `"sage"` |
| `Color.memberTeal` | `#4F7D7D` | `"teal"` |
| `Color.memberTerracotta` | `#C58463` | `"terracotta"` |
| `Color.memberPlum` | `#7B5C7E` | `"plum"` |
| `Color.memberSlateBlue` | `#6B7E94` | `"slateBlue"` |
| `Color.memberRose` | `#B07D7B` | `"rose"` |
| `Color.memberOlive` | `#857C56` | `"olive"` |

**Rules:**
- No two members in the same family may have the same color. Enforce in `FamilyService.assignColor()`.
- The Member model stores a string token, not the Color itself.
- Look up via `Color.member(_ token: String) -> Color`.
- Whole-family events use `Color.brandNavy`. A member with the navy color shares the family's navy visual — that's intentional.

### 4.3 Typography

- **Display: Fraunces** — variable serif with optical sizing. Used for moments: wordmark, section headers, time-of-day greetings, hero numbers, quoted microcopy.
- **Body: DM Sans** — rounded geometric sans-serif. Used for everything else: UI labels, body copy, list items, tab bars, buttons.

Both are bundled as custom fonts. Fraunces uses opsz axis 9–144 and weights 300–600. DM Sans uses opsz 9–40 and weights 400–700.

In code:
```swift
extension Font {
    static func fraunces(_ size: CGFloat, weight: Font.Weight = .regular) -> Font {
        .custom("Fraunces", size: size).weight(weight)
    }
    static func dmSans(_ size: CGFloat, weight: Font.Weight = .regular) -> Font {
        .custom("DMSans", size: size).weight(weight)
    }
}
```

### 4.4 Design guardrails

**Do:**
- Keep layouts airy and uncluttered
- Soft shapes, rounded corners (14–24pt typical)
- One accent color (gold) per screen, on the moment that matters
- Sage only on completion states
- Generous negative space

**Don't:**
- Don't use overly busy icons — line style, 1.5pt stroke max
- Don't make the brand feel childish
- Don't use too many colors at once
- Don't add unnecessary decorative elements
- Don't introduce gradients except the navy gradient on Recognition's hero card
- Don't use emoji as decoration (badges and activity feed are the only exceptions)

---

## 5. Architecture

**MVVM with @Observable view models.**

```
View  →  ViewModel (@Observable)  →  Service  →  Firestore
                                          ↓
                                       Models
```

- **Views** observe ViewModels and call methods on them. No business logic.
- **ViewModels** are `@Observable` classes that hold view state and call services.
- **Services** are singletons (or DI-injected) wrapping Firestore, Auth, Storage, etc.
- **Models** are plain Swift structs conforming to `Codable` and `Identifiable`.

**Never:**
- Put Firestore queries inside SwiftUI views
- Use `@Published` / `ObservableObject` (we use `@Observable`)
- Force unwrap (`!`) — use `guard let` or `if let`
- Use `Task { }` inside `body` — use `.task { }` view modifier
- Catch errors silently

---

## 6. Naming Convention: Domain vs Brand

The code and the UI use different vocabularies on purpose.

**Code** uses domain terms. **UI** uses brand terms from the brand sheet.

| Concept | Code (domain) | UI (brand) | View file | Folder |
|---|---|---|---|---|
| Calendar event | `Event`, `EventService` | "Days" / "Nestara Days" | `DaysView.swift` | `Views/Days/` |
| Chore | `Chore`, `ChoreService` | "Tasks" / "Nestara Tasks" | `TasksView.swift` | `Views/Tasks/` |
| Chore template | `ChoreTemplate` | "Task library" | `TaskLibraryView.swift` | `Views/Tasks/` |
| Todo list (container) | `TodoList`, `TodoListService` | "list" / "Lists" / "Nestara Lists" | `ListsView.swift`, `ListDetailView.swift` | `Views/Lists/` |
| Todo item | `Todo` | "item" | inside `Views/Lists/` | same |
| Shopping | `ShoppingItem`, `ShoppingService` | "Shop" / "Shopping" | `ShopView.swift` | `Views/Shop/` |
| Family | `Family`, `FamilyService` | "Circle" / "Nestara Circle" | `CircleView.swift` | `Views/Circle/` |
| Member | `Member` | "Member" / "circle member" | various | `Views/Circle/` |
| Recognition | `Recognition*`, `BadgeService` | "Recognition" | `RecognitionView.swift` | `Views/Recognition/` |

Use brand labels in UI text. Use domain names in code. Don't mix them inside one identifier.

---

## 7. Folder Structure

```
Nestara/
├── NestaraApp.swift                # @main, FirebaseApp.configure()
├── ContentView.swift               # Root, switches auth ↔ main app
├── CLAUDE.md                       # This file
├── GoogleService-Info.plist
│
├── Models/
│   ├── User.swift
│   ├── Family.swift
│   ├── Member.swift
│   ├── Event.swift
│   ├── Chore.swift
│   ├── ChoreCompletion.swift
│   ├── Todo.swift
│   ├── ShoppingItem.swift
│   ├── Badge.swift
│   └── ActivityEvent.swift
│
├── Services/
│   ├── AuthService.swift
│   ├── FamilyService.swift
│   ├── EventService.swift
│   ├── ChoreService.swift
│   ├── TodoService.swift
│   ├── ShoppingService.swift
│   ├── RecognitionService.swift
│   ├── BadgeService.swift
│   ├── ActivityService.swift
│   ├── StorageService.swift
│   └── NotificationService.swift
│
├── ViewModels/
│   └── (one per major view)
│
├── Views/
│   ├── Auth/
│   ├── Onboarding/
│   ├── Home/
│   ├── Days/                       # Calendar — DaysView, MonthView, WeekView, DayView, EventEditView
│   ├── Tasks/                      # Chores — TasksView, TaskEditView, TaskLibraryView, completion flow
│   ├── Lists/                      # Multiple named lists — ListsView, ListDetailView, ListEditView, ItemEditView
│   ├── Shop/                       # ShopView
│   ├── Recognition/                # Points, badges, leaderboard
│   ├── Circle/                     # Family, members, invites
│   └── Components/                 # Reusable: AvatarView, MemberDot, etc.
│
├── Utilities/
│   ├── Color+Nestara.swift         # Foundation + member palette
│   ├── Font+Nestara.swift          # Fraunces and DM Sans helpers
│   ├── Date+Helpers.swift
│   ├── RecurrenceRule.swift
│   └── PermissionGate.swift
│
└── Resources/
    ├── Assets.xcassets
    └── Fonts/                      # Bundled Fraunces and DM Sans .ttf files
```

---

## 7.1 Starter content on first family

The Lists tab no longer uses named-list containers (rebuilt in v1.1 around a
flat to-do model with Yours / Assigned tabs and an Up for Grabs section), so
there is no per-family starter list to seed on `FamilyService.createFamily`.
The empty Yours tab shows microcopy ("Nothing on your list. Add something to
start the day.") instead.

---

## 8. Firestore Data Model

```
users/{userId}
  - displayName, email, photoURL, currentFamilyId, createdAt

families/{familyId}
  - name, inviteCode, createdAt, createdBy, settings

families/{familyId}/members/{userId}
  - displayName, role (parent|adult|teen|child),
    color (token: "navy"|"sage"|"teal"|"terracotta"|"plum"|"slateBlue"|"rose"|"olive"),
    points, monthlyPoints, badges[], streakCount, lastStreakDate,
    deviceTokens[], emailPrefs, joinedAt

families/{familyId}/events/{eventId}
  - title, description, start (Timestamp), end (Timestamp), isAllDay,
    assignedTo[], type, recurrence, source, externalId, createdBy, createdAt

families/{familyId}/chores/{choreId}
  - title, description, assignedTo, recurrence, dueTime,
    points, photoRequired, active, createdBy, createdAt,
    templateId (optional — links back to source template if instantiated from one)

families/{familyId}/choreTemplates/{templateId}
  - title, description, defaultPoints, defaultDueTime, defaultPhotoRequired,
    defaultRecurrence, createdBy, createdAt, useCount

families/{familyId}/chores/{choreId}/completions/{completionId}
  - date (yyyy-MM-dd), completedBy, completedAt, photoUrl, onTime, pointsAwarded

families/{familyId}/todoLists/{listId}
  - name, icon (optional SF Symbol), accentColor (optional member token),
    kind ("yours" | "assigned"),
    createdBy, assignedTo[] (member IDs, empty when kind == "yours"),
    sortOrder, createdAt, archived

families/{familyId}/todoLists/{listId}/items/{itemId}
  - title, description, dueDate (optional),
    completed, completedAt, completedBy,
    sortOrder, createdAt

families/{familyId}/shoppingList/{itemId}
  - name, addedBy, addedAt, purchased, purchasedBy, purchasedAt

families/{familyId}/activity/{eventId}
  - userId, type, refId, message, timestamp, points, photoUrl

families/{familyId}/badges/{userId}/earned/{badgeId}
  - badgeId, earnedAt
```

**Security rules** must enforce:
- Users only read/write data in families they belong to
- Only `parent` role can write to `chores/`
- Anyone in the family can write to `shoppingList/`, `events/`, their own `todos/`
- No client can write to `activity/` or `members/{userId}/points` — those go through Cloud Functions

---

## 9. Permission Matrix

| Action | Parent | Adult | Teen | Child |
|---|---|---|---|---|
| Create family | ✅ | ❌ | ❌ | ❌ |
| Invite/remove members | ✅ | ❌ | ❌ | ❌ |
| Edit any event | ✅ | ✅ | own only | own only |
| Create chores | ✅ | ❌ | ❌ | ❌ |
| Edit chores | ✅ | ❌ | ❌ | ❌ |
| Complete assigned chores | ✅ | ✅ | ✅ | ✅ |
| Approve photo-required chores | ✅ | ❌ | ❌ | ❌ |
| Create own to-dos | ✅ | ✅ | ✅ | ✅ |
| Assign to-dos to others | ✅ | ❌ | ❌ | ❌ |
| Edit shopping list | ✅ | ✅ | ✅ | ✅ |
| View leaderboard | ✅ | ✅ | ✅ | ✅ |
| Set point→reward conversions | ✅ | ❌ | ❌ | ❌ |
| Change family settings | ✅ | ❌ | ❌ | ❌ |

Enforce in `Utilities/PermissionGate.swift`.

---

## 10. Code Style

- Indent 4 spaces. No tabs.
- One type per file. File name matches type name.
- View names end in `View`. ViewModel names end in `ViewModel`. Service names end in `Service`.
- Avoid `Manager` and `Helper` suffixes.
- Prefer `let` over `var`. Mark types `final` unless meant to be subclassed.
- Keep view bodies under 50 lines. Extract subviews aggressively.
- Use `#Preview` for every view. Inject mock data via static `mock` on each model.
- Sparse comments — prefer self-documenting code.

---

## 11. Common Patterns

### Firestore listener in a service
```swift
@Observable
final class ChoreService {
    var chores: [Chore] = []
    private var listener: ListenerRegistration?

    func startListening(familyId: String) {
        listener?.remove()
        listener = Firestore.firestore()
            .collection("families/\(familyId)/chores")
            .whereField("active", isEqualTo: true)
            .addSnapshotListener { [weak self] snapshot, _ in
                guard let docs = snapshot?.documents else { return }
                self?.chores = docs.compactMap { try? $0.data(as: Chore.self) }
            }
    }

    func stopListening() { listener?.remove(); listener = nil }
}
```

### Member color lookup
```swift
extension Color {
    static func member(_ token: String) -> Color {
        switch token {
        case "navy":       return .memberNavy
        case "sage":       return .memberSage
        case "teal":       return .memberTeal
        case "terracotta": return .memberTerracotta
        case "plum":       return .memberPlum
        case "slateBlue":  return .memberSlateBlue
        case "rose":       return .memberRose
        case "olive":      return .memberOlive
        default:           return .memberNavy
        }
    }
}
```

### Standard label/header pattern
```swift
// Section label — gold caps tracking
Text("TODAY'S FLOW")
    .font(.dmSans(11, weight: .medium))
    .tracking(2)
    .foregroundStyle(Color.brandGold)

// Section header — Fraunces serif, navy
Text("Tasks")
    .font(.fraunces(32, weight: .light))
    .foregroundStyle(Color.brandNavy)
```

---

## 12. Verification Protocol — DO THIS AFTER EVERY TASK

You must verify your work before saying you're done. Use the XcodeBuildMCP tools.

1. **Build** — call `build_sim`. Fix all errors. Treat warnings as errors unless the task says otherwise.
2. **Lint** — run `swiftlint`. Fix violations.
3. **Run** — call `build_run_sim` to launch in simulator.
4. **Screenshot** — call `screenshot` and verify the UI matches what the task asked for AND the brand. Specifically check: cream background, navy text, gold only on "moments that matter", sage only on completion, no exclamation marks in copy, no random colors.
5. **Test the flow** — use `tap` and `screenshot` to step through any new feature.
6. **Tests** — if unit tests exist, run them via `test_sim`.
7. **Diff review** — `git diff` and self-review. Look for force unwraps, prints, hardcoded strings, missing error handling, microcopy that violates voice rules.
8. **Commit** — descriptive message tied to step number.

**If any step fails after 3 self-correction attempts, STOP and ask the human.**

---

## 12.1 Deploying Firebase rules

When a step modifies `firestore.rules` or `storage.rules`:

1. **Print the full updated rules file in chat** as a fenced code block, before
   asking the human to deploy. Don't make them ask for it.
2. Tell them exactly where to paste it (Firebase Console → Firestore → Rules,
   or Console → Storage → Rules).
3. **Pause and wait for explicit confirmation** that publishing is done before
   continuing with verification. Don't kick off any UI flow that depends on the
   new rules until they say it's published.

---

## 13. Git Workflow

- Commit after every successful step.
- Tag at the end of each phase: `git tag phase-1-foundation`, etc.
- Default to `main` branch.
- Commit messages:
  ```
  [Step N] Short summary

  - What was added/changed
  - What was tested
  ```

---

## 14. What NOT To Do

- ❌ Do NOT use UIKit unless a SwiftUI equivalent doesn't exist
- ❌ Do NOT use CocoaPods. SPM only.
- ❌ Do NOT use `@Published` / `ObservableObject`
- ❌ Do NOT use force unwraps (`!`)
- ❌ Do NOT create files outside the folder structure
- ❌ Do NOT skip the verification protocol
- ❌ Do NOT add libraries without asking
- ❌ Do NOT use exclamation marks in microcopy
- ❌ Do NOT use generic system fonts where Fraunces/DM Sans should be used
- ❌ Do NOT introduce new colors outside the foundation + member palette
- ❌ Do NOT use gradients except the one navy gradient on Recognition's hero
- ❌ Do NOT decorate with emoji (badges and activity feed are exceptions)
- ❌ Do NOT silently catch errors
- ❌ Do NOT modify CLAUDE.md without asking the human first

---

## 15. When to Stop and Ask

Stop and ask if:
- A task is ambiguous or missing critical info
- You'd need to deviate from the architecture in this file
- A microcopy choice doesn't match the voice specimens — show options first
- Tests fail in a way you can't quickly fix
- Build fails after 3 self-correction attempts
- A feature requires a service/library not listed
- Something in the prompt seems to contradict CLAUDE.md
- You're about to delete or significantly refactor existing working code

It is always better to ask than to guess.

---

*End of CLAUDE.md. Re-read sections 3, 4, 12, and 14 before every commit. The brand is what makes Nestara Nestara — it's not optional polish, it's the product.*

---
> Source: [nestaraapp/nestara](https://github.com/nestaraapp/nestara) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
