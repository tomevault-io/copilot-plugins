## notepro-viet

> <!-- GSD:project-start source:PROJECT.md -->

<!-- GSD:project-start source:PROJECT.md -->
## Project

**ColorNote Pro Việt — App Ghi Chú Âm Lịch Việt Nam**

A note-taking app for Vietnamese users (targeting homemakers and middle-aged adults, 35-60) that combines ColorNote-style colored notes/checklists with built-in lunar calendar reminders. The core wedge: users set a death anniversary (giỗ) date once in the lunar calendar, and the app automatically converts it to the correct Gregorian date every year and sends notifications — eliminating the painful annual manual conversion.

**Core Value:** **Automatic yearly lunar-to-Gregorian reminder conversion** — if everything else fails, this must work. Users must never miss a family death anniversary because the app got the date wrong.

### Constraints

- **Tech Stack**: React Native + Expo SDK, `lunar-calendar` npm (or Hồ Ngọc Đức algorithm fallback), `react-native-mmkv` for storage, `expo-notifications`, `react-native-reanimated`
- **Platform**: Android-only v1 (phones 360dp–480dp width)
- **Budget**: Bootstrap — no paid services, fully offline architecture
- **Lunar accuracy**: Must pass 20 known lunar date test cases before shipping (trust killer if wrong)
- **No backend**: Fully offline v1 — accepted trade-off that reminders may fail if app not opened >1 year
- **Typography**: NotoSans (Google Fonts) — excellent Vietnamese Unicode/diacritical support
- **Cultural**: No celebration animations (inappropriate for giỗ context), red = giỗ meaning not decoration
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Recommended Stack (2025-2026)
### Core Framework
| Component | Recommendation | Version | Confidence | Rationale |
|-----------|---------------|---------|------------|-----------|
| Framework | React Native + Expo | SDK 52+ | ✅ High | Managed workflow, OTA updates, EAS builds. Cross-platform ready for iOS v2 |
| Language | TypeScript | 5.x | ✅ High | Type safety critical for lunar date math. First-class Expo support |
| State Management | Zustand | 5.x | ✅ High | Lightweight, no boilerplate. Perfect for simple note store. Overkill: Redux |
### Lunar Calendar (CRITICAL)
| Library | Recommendation | Confidence | Notes |
|---------|---------------|------------|-------|
| **`@dqcai/vn-lunar`** | ✅ PRIMARY | High | Vietnamese-specific. Solar↔Lunar 1200-2199 CE. Can Chi. Leap months. Zero deps. Cross-platform RN/Expo compatible |
| **`@forvn/vn-lunar-calendar`** | FALLBACK | High | Also Vietnamese-specific with bidirectional conversion. TypeScript support |
| `lunar-calendar` (npm) | ❌ AVOID | — | Chinese-focused, may not handle Vietnamese-specific edge cases. Design doc recommended this but newer Vietnamese-specific packages exist |
| Hồ Ngọc Đức algorithm | LAST RESORT | — | Manual implementation. Only if both npm packages fail spike |
### Storage
| Component | Recommendation | Version | Confidence | Rationale |
|-----------|---------------|---------|------------|-----------|
| Local Storage | `react-native-mmkv` | 3.x | ✅ High | 30x faster than AsyncStorage. Key-value perfect for notes JSON. Already in design doc |
| ❌ NOT | AsyncStorage | — | — | Too slow for frequent auto-save (500ms debounce) |
| ❌ NOT | SQLite/Realm | — | — | Overkill for simple Note[] schema |
### Notifications
| Component | Recommendation | Version | Confidence | Rationale |
|-----------|---------------|---------|------------|-----------|
| Notifications | `expo-notifications` | SDK 52+ | ✅ High | Managed workflow compatible. Local scheduling. Already in design doc |
| ❌ NOT | `react-native-push-notification` | — | — | Requires bare workflow. Not compatible with Expo managed |
### UI Components
| Component | Recommendation | Confidence | Rationale |
|-----------|---------------|------------|-----------|
| WheelPicker | `react-native-wheel-pick` or `@react-native-picker/picker` | Medium | Need to spike for Expo compatibility. Custom implementation may be needed |
| View capture | `react-native-view-shot` | Medium | Share card PNG generation. Spike needed for Expo managed compatibility |
| Animations | `react-native-reanimated` | ✅ High | Already in design doc. Spring animations for note card appearance |
| Icons | `@expo/vector-icons` | ✅ High | Built into Expo. MaterialIcons for toolbar |
| Font | NotoSans via `expo-font` | ✅ High | Excellent Vietnamese diacritical support |
### Backup
| Component | Recommendation | Confidence | Rationale |
|-----------|---------------|------------|-----------|
| Google Drive | `@react-native-google-signin/google-signin` + Drive REST API | Medium | OAuth + `drive.appdata` scope. Setup friction: GCP Console, SHA-1 |
| Local fallback | `expo-sharing` + `expo-file-system` | ✅ High | JSON export for devices without Google Play Services |
### What NOT to Use
- ❌ **Redux** — Overkill for simple Note[] state
- ❌ **MongoDB Realm** — Sync features not needed for offline-only v1
- ❌ **Firebase** — Adds complexity, not needed for offline-first
- ❌ **`@umalqura/core`** — Islamic (Hijri) calendar, NOT Vietnamese lunar
- ❌ **Moment.js** — Deprecated, use `date-fns` or native `Intl` if needed
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jokerlin135)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jokerlin135)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
