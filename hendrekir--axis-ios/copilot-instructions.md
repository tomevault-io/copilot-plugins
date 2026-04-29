## axis-ios

> Axis is an ambient AI agent layer for iPhone. It bonds to iOS at the OS level,

# AXIS — .cursorrules
# Load this in Cursor IDE for all iOS Swift development
# Equivalent to CLAUDE.md but Cursor-native

## WHAT THIS PROJECT IS
Axis is an ambient AI agent layer for iPhone. It bonds to iOS at the OS level,
reads every app passively (email, messages, Instagram, Snapchat, calendar, health,
location, notifications), and communicates back through:
- A persistent iMessage-style thread
- Lock screen WidgetKit widget (one signal, always visible)
- Dynamic Island (ActivityKit — live agent status)
- Notification shelf (UNNotificationServiceExtension — rewrites every notification)
- Apple Watch complication

Slogan: Everything you're trying to hold in your head. Held.
Acquirer target: Apple. Timeline: 18 months.

## iOS TECHNICAL CONSTRAINTS
- iOS 17.0+ minimum target
- SwiftUI-first, UIKit only where SwiftUI cannot do it
- SF Symbols exclusively for all icons — no third-party icon sets
- Privacy-first: raw personal data NEVER leaves device. Only generated text and
  metadata hits the backend. All classification runs on-device or via Claude API call.
- Test EVERYTHING on a real iPhone. Never simulator-only for:
  - WidgetKit widgets
  - ActivityKit Live Activities
  - UNNotificationServiceExtension
  - Background processing
  - CoreLocation
  - HealthKit

## BACKEND CONTRACT
Backend: FastAPI on Railway (api.axis.app)
Auth: Sign in with Apple + Clerk JWT tokens
All API calls authenticated with Bearer token in Authorization header.

Key endpoints:
- POST /thread — create message, returns Axis response
- GET /signal — returns current top 3 tasks for user
- POST /tasks — create task
- PATCH /tasks/{id} — update (done, reprioritise)
- POST /brain-dump — Claude API call, returns ranked task JSON
- GET /brief — morning digest generation
- GET /skills/{skill_id}/chat — talk to a specific skill

## FOLDER STRUCTURE
```
Axis/
  Views/
    ThreadView.swift         # iMessage-style persistent thread
    SignalView.swift         # Hero task + queue + battle plan + timer
    BrainDumpView.swift      # Free text → ranked task list
    SkillsView.swift         # 6 skill cards, talk to each
    DayPlanView.swift        # Time-blocked day structure
    OnboardingView.swift     # Name + mode + first 3 tasks
  Extensions/
    NotificationExtension/   # UNNotificationServiceExtension — DO NOT TOUCH without contractors
    WidgetExtension/         # WidgetKit lock screen widget
    ShareExtension/          # Share sheet for capturing from any app
  Services/
    APIService.swift         # All backend API calls
    CloudKitService.swift    # Team signal sync
    HealthKitService.swift   # Apple Health reads
    EventKitService.swift    # Calendar reads
    RevenueCatService.swift  # Subscription management
  Models/
    Thread.swift
    Signal.swift
    Task.swift
    Skill.swift
    User.swift
```

## KEY DEPENDENCIES (Swift Package Manager)
- RevenueCat/purchases-ios — in-app subscriptions
- PostHog/posthog-ios — analytics
- Clerk/clerk-ios — auth helper (or use Sign in with Apple directly)

## CODING CONVENTIONS
- SwiftUI views: use @StateObject for view models, @EnvironmentObject for global state
- Async/await for all API calls — no completion handlers
- Error handling: always handle errors, never use try! in production code
- Naming: camelCase properties, PascalCase types, imperative method names (fetchSignal, markTaskDone)
- Comments: only for non-obvious logic — don't comment the obvious

## WHAT'S ALREADY BUILT (update this as you go)
- [ ] Xcode project skeleton
- [ ] ThreadView
- [ ] SignalView
- [ ] BrainDumpView
- [ ] SkillsView
- [ ] DayPlanView
- [ ] Sign in with Apple
- [ ] RevenueCat integration
- [ ] Basic WidgetKit widget

## WHAT REQUIRES CONTRACTORS (DO NOT attempt in Cursor)
- UNNotificationServiceExtension (notification interception)
- ActivityKit Live Activities (Dynamic Island)
- Social Skill notification parsing
- Screen Time API app intercept
- On-device CoreML model

## CURRENT PHASE
Phase 2 — Basic iOS app. Build the SwiftUI shell. Get to TestFlight.
Do NOT attempt notification extensions or Dynamic Island yet.

## LAST UPDATED
March 2026 — initial setup. Update after every session.

---
> Source: [hendrekir/axis-ios](https://github.com/hendrekir/axis-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
