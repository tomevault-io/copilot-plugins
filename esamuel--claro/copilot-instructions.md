## claro

> Claro is a native SwiftUI iOS app (iOS 17+) for AI-powered phone storage cleaning.

# CLAUDE.md — Claro iOS App

## Project
Claro is a native SwiftUI iOS app (iOS 17+) for AI-powered phone storage cleaning.
Target: Apple App Store (primary). Always build with App Store submission in mind.

---

## 🍎 App Store Compliance Rules — ALWAYS FOLLOW

### In-App Purchases
- **Always use StoreKit 2** (`Product`, `Transaction`, `AppStore.sync()`) — never external payment links
- **Never hardcode prices** — always display `product.displayPrice` fetched from StoreKit
- **Product IDs** are defined in `StoreKitService.swift` — must match App Store Connect exactly
- **Restore Purchase button** must always be visible on the paywall (Apple guideline 3.1.1)
- Free trial terms must state: duration, price after trial, how to cancel

### Paywall / Subscriptions
- **Close button must be easy to find** — no hidden or tiny dismiss buttons (Apple rejects dark patterns)
- **No weekly subscriptions** — we use: Annual (3-day trial), Lifetime, Monthly
- Renewal boilerplate text required: "Subscriptions renew automatically unless cancelled 24h before renewal"
- Must link to real **Terms of Service** and **Privacy Policy** URLs (tappable, not just text)
- Cancel instructions must say: Settings → Apple ID → Subscriptions

### Ratings & Reviews
- Use `SKStoreReviewController.requestReview(in:)` only — never custom review prompts
- Apple allows max 3 review prompts per year — the system handles this automatically
- Never gate features behind a review

### Permissions
- All permission strings must be in `Info.plist`:
  - `NSPhotoLibraryUsageDescription`
  - `NSPhotoLibraryAddUsageDescription`
  - `NSContactsUsageDescription`
  - `NSUserNotificationsUsageDescription`
- Never request permissions until the user takes an action that needs them
- Always handle `.denied` state gracefully with a "Open Settings" fallback

### iCloud / CloudKit
- **Never use `CKContainer` or `import CloudKit`** — requires `com.apple.developer.icloud-services` entitlement which needs a paid developer account
- To check if the user is signed into iCloud use `FileManager.default.ubiquityIdentityToken` (no entitlement needed)
- For iCloud storage management, direct users to Settings → Apple ID → iCloud (no public API for quota management)
- If CloudKit is ever needed in future, it requires: paid account + iCloud capability in Xcode + entitlement in project.yml

### Data & Privacy
- A `PrivacyInfo.xcprivacy` file is **required** before App Store submission
- Declare all "required reason APIs" (UserDefaults, file timestamps, etc.)
- No tracking without explicit user consent (ATT framework if needed)
- If user accounts are added: must offer account deletion (guideline 5.1.1)

### Photo / Contact Cleanup
- **Never auto-delete** without explicit user confirmation (review each item before deletion)
- Show a confirmation alert/sheet before any destructive action
- Deleted photos should go to "Recently Deleted" album (use `PHAssetChangeRequest.deleteAssets`)
- This protects against App Store rejection for data loss

### Content & UI
- No placeholder content in shipping builds — all features must work or be hidden
- All tappable URLs must point to real, live pages
- App must function without network (core cleaning features are local)

---

## Architecture

### Services (inject via `@Environment`)
| Service | Purpose |
|---------|---------|
| `AppSettings` | Color scheme + language preferences |
| `PermissionsService` | Photo / Contact / Notification permissions |
| `StoreKitService` | IAP products, purchases, entitlements |

### Product IDs (App Store Connect)
```
com.samueleskenasy.claro.pro.annual    → Annual subscription (3-day trial)
com.samueleskenasy.claro.pro.monthly   → Monthly subscription
com.samueleskenasy.claro.pro.lifetime  → Non-consumable one-time purchase
```

### Key Files
```
Claro/App/ClaroApp.swift                    — App entry, injects all services
Claro/Core/Services/StoreKitService.swift   — StoreKit 2 purchases & entitlements
Claro/Core/Services/PermissionsService.swift
Claro/Core/Services/AppSettings.swift
Claro/Features/Paywall/PaywallView.swift    — App Store compliant paywall
Claro/Core/Theme/ClaroTheme.swift           — Design tokens
```

---

## Design System
- Background: `#0D1117` (claroBg)
- Card: `#161B22` (claroCard)
- Accent: `#7C3AED` violet, `#06B6D4` cyan, `#F59E0B` gold
- All fonts via `Font+Claro.swift` extensions
- RTL support: Hebrew layout direction set via `.environment(\.layoutDirection, ...)`

---

## Before Submitting to App Store
- [ ] Create IAP products in App Store Connect using the product IDs above
- [ ] PrivacyInfo.xcprivacy — already created ✅
- [ ] Add a hosted privacy policy / terms page (URL needed for App Store Connect)
- [ ] Update `rateApp()` in SettingsView with real App Store numeric ID
- [ ] Test StoreKit in Sandbox environment
- [ ] Test all permission flows (grant, deny, restricted)
- [ ] Confirm no hardcoded prices anywhere in UI
- [ ] Confirm Restore Purchase works in sandbox
- [ ] App icon + screenshots for all device sizes
- [ ] Age rating questionnaire completed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/esamuel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
