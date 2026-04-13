## movo-hb-ios

> This is a production Apple platform project maintained in Xcode.

# Project Context
This is a production Apple platform project maintained in Xcode.
All changes must be treated as production-grade and App Store safe.

Claude is assisting a human developer and must prioritize correctness,
safety, and clarity over speed.

Claude must stop immediately and ask for clarification if instructions conflict.

---

## Interaction Rules (MANDATORY)

- If a request is ambiguous, ASK clarifying questions before writing code.
- Do not assume architecture, frameworks, or intent.
- Ask one question at a time if clarification is required.
- For changes affecting more than one file:
  1. Ask clarifying questions
  2. Propose a short plan
  3. Wait for explicit approval

No code changes are allowed until questions are answered.

---

## Read-First Rule (MANDATORY)

Before making any change, Claude MUST:
- Read and understand relevant existing files
- Summarize current behavior briefly
- Avoid duplicating existing logic

---

## Change Impact Summary (MANDATORY)

Before implementing changes, Claude MUST provide:
- Files that will be modified
- Risk level (Low / Medium / High)
- Possible side effects or regressions

Proceed only after approval.

---

## Scope Control

Claude MUST:
- Implement only what is explicitly requested
- Avoid “nice-to-have” additions
- Avoid opportunistic refactors unless approved

---

## Git Workflow (MANDATORY)

Before making any code change (feature or bug fix), Claude MUST:

1. Ensure the working tree is clean.
2. Create a new Git branch:
   - `feature/<short-description>` for new features
   - `bugfix/<short-description>` for bug fixes
3. Commit the current state before making changes (baseline commit).
4. Implement the requested changes.
5. Commit changes with clear, descriptive messages.

Rules:
- Never commit directly to `main` or `develop`.
- Never rebase, squash, or push unless explicitly instructed.
- Use small, logical commits.
- If branch creation or committing is not possible, STOP and ask.

---

## Documentation Requirement (MANDATORY)

For every new feature developed, Claude MUST:

1. Create a documentation file explaining the feature.
2. Include:
   - Purpose of the feature
   - High-level design and user flow
   - Key files and components involved
   - Any configuration, permissions, or setup required
3. Place documentation in:
   - `docs/<feature-name>.md`, or
   - Update `README.md` if the feature is small

Rules:
- Documentation must be written in clear, simple English.
- A feature is NOT complete without documentation.
- Bug fixes require documentation ONLY if behavior changes.

Documentation is NOT required for:
- Minor UI tweaks
- Styling-only changes
- Refactors without behavior changes

If documentation location is unclear, ASK before proceeding.

---

## Dependency Rules

- Do not add third-party libraries unless explicitly approved.
- Prefer native Apple frameworks.
- If a dependency is suggested, explain WHY and list alternatives.

---

## Safety & Change Control

Claude MUST NOT modify the following unless explicitly instructed:
- `project.pbxproj`
- Bundle Identifier
- Signing & Capabilities
- Entitlements
- Build Settings
- Privacy-related `Info.plist` keys

Prefer minimal, reversible changes.

---

## Security & Privacy

- Do not log PII, tokens, or credentials.
- Do not store secrets in code.
- Highlight handling of authentication, tokens, or sensitive data.
- Call out App Store privacy implications when relevant.

---

## iOS / Swift Standards

- Swift 5.9 or later
- Prefer SwiftUI over UIKit unless specified
- Use async/await (no completion handlers)
- Avoid force unwraps (`!`)
- No deprecated APIs
- Follow Apple Human Interface Guidelines

---

## Architecture Guidelines

- Use MVVM unless stated otherwise
- Views must not contain networking or business logic
- ViewModels must be UI-agnostic
- Networking must be isolated in service layers
- Prefer protocol-based abstractions

---

## Error Handling & Logging

- Handle errors explicitly
- Do not swallow errors silently
- Prefer typed errors over generic `Error`
- Avoid `print()` in production code

---

## Testing Rules

- Use XCTest
- Prefer unit tests over UI tests
- New logic should include tests unless instructed otherwise
- Existing tests must not be broken

---

## Breaking Change Guardrail

If a change could:
- Break existing APIs
- Change user-facing behavior
- Affect persisted data

Claude MUST stop and ask before proceeding.

---

## AI Behavior Constraints

- Do not hallucinate Apple APIs, frameworks, or files
- Do not invent files that do not exist
- Read existing code before suggesting changes
- If uncertain, say “I’m not certain” and ask
- Explain reasoning briefly when proposing changes

---
# MOVOCASH MOBILE APP ARCHITECTURE
## Document 3 — MovoCash Engineering Core
**Classification:** Required Reading — All Mobile Engineering Personnel

---

## 1. Platform Overview

| Platform | Language | Status | Priority |
|----------|----------|--------|----------|
| iOS | Swift (with Objective-C bridge) | Active development — Herring migration in progress | MVP |
| Android | Kotlin (planned) | Not started | Post-MVP |

iOS is the priority platform. Android development begins after iOS MVP is complete. Both platforms will call the same Movo Skinny Processor middleware — mobile apps never call Herring Bank directly.

---

## 2. iOS Application



### Architecture Pattern
The iOS app uses a mixed architecture resulting from legacy development and the ongoing Herring migration:

- **Legacy Layer:** Objective-C `APIManager.h/m` — original API client built against the Admin API (i2c/MCPSDK)
- **Migration Layer:** New Swift files for Herring Consumer API integration
- **KYC Layer:** `HerringMovoKycSDK` with BlinkIDUX 7.3.1 (Microblink) for document scanning
- **Wallet Layer:** PassKit (native Apple Pay) replacing MCPSDK push provisioning

### Key Principle
The iOS app is a **presentation layer**. It collects user input, displays data, and handles device-specific features (camera, NFC, biometrics, Apple Pay). All business logic, API orchestration, and data processing happens in the Skinny Processor middleware.

---

## 3. iOS File Map — Herring Integration Files

These are the new Swift files created for the Herring Bank integration. They replace the legacy MCPSDK components.

### Authentication
| File | Location | Purpose |
|------|----------|---------|
| `HerringAuth.swift` | `Utilities/Network/` | Authentication with Herring Consumer API — handles OTP → JWT flow, token storage, token refresh |

**Replaces:** MCPSDK authentication in `APIManager.h/m`

### KYC / Identity
| File | Location | Purpose |
|------|----------|---------|
| `IdentityData.swift` | `Models/General Models/` | Data models for KYC identity information (name, address, DOB, DL data, SSN) |
| `HerringVerifyIdRequest.swift` | `Models/Request Response Models/` | Request model for `POST /id/verify-id` — maps to `VerifyIdRequest` swagger schema |

**Replaces:** MCPSDK KYC flow

### Network / Callbacks
| File | Location | Purpose |
|------|----------|---------|
| `HerringCallbackProtocol.swift` | `Utilities/Network/` | API callback protocol — standardized response handling for all Herring API calls |

**Replaces:** `MCPSDKCallBack` protocol

### Apple Pay / Wallet Provisioning
| File | Location | Purpose |
|------|----------|---------|
| `HerringWalletProvisioning.swift` | `Utilities/Network/` | Apple Pay card provisioning via PassKit — calls `POST /vcards/provision` with provider: "APPLE" |

**Replaces:** `MCPInAppPushProvisioning` (MCPSDK push provisioning)

### KYC SDK Bridge
| File | Location | Purpose |
|------|----------|---------|
| `KycBridge.swift` | `Bridges/` | Bridge between BlinkIDUX document scanner and Herring KYC flow — enhanced with 20+ field schema, error handling (10+ categories) |

---

## 4. Legacy Code — What's Being Replaced

### APIManager (Objective-C Bridge)
| File | Purpose | Status |
|------|---------|--------|
| `APIManager.h` | Header for legacy API client | Being replaced by Herring Swift files |
| `APIManager.m` | Implementation — calls Admin API at `https://appapi.movo.cash/api/v5` | Active but deprecated |

**Critical:** `APIManager` calls the **Admin API** (68 endpoints). All new work targets the **Consumer API** through the Skinny Processor. Do not add new functionality to `APIManager`. New features go through `HerringAuth.swift` and the middleware.

### MCPSDK (i2c SDK) — Being Removed
| Component | Replacement |
|-----------|-------------|
| MCPSDK Authentication | `HerringAuth.swift` → OTP/JWT flow |
| MCPSDK KYC | `HerringMovoKycSDK` + `KycBridge.swift` |
| MCPSDKCallBack | `HerringCallbackProtocol.swift` |
| MCPInAppPushProvisioning | `HerringWalletProvisioning.swift` + PassKit |
| MCPSDK Card Operations | Skinny Processor → Consumer API |

---

## 5. KYC SDK Integration

### BlinkIDUX 7.3.1 (Microblink)
**Purpose:** Document scanning for identity verification (driver's license capture)

**Integration Point:** `ios/MovoCash/Bridges/KycBridge.swift`

**Data Flow:**
```
User opens KYC screen
    ↓
BlinkIDUX scanner activates (camera)
    ↓
User scans DL front + back
    ↓
KycBridge.swift processes scan
    ↓
Enriched JSON with 20+ fields generated:
  - Name, DOB, Address
  - DL number, state, expiration
  - Raw scan data
  - Quality metrics (OCR confidence, image quality, liveness)
    ↓
App calls GET /id/verify-id-url (via middleware)
    ↓
DL images uploaded to S3 (multipart, min 5MB chunks)
    ↓
App calls POST /id/verify-id with all CIP data + upload results
    ↓
Herring Bank makes approval/rejection decision
    ↓
App displays result to user
```

**Quality Metrics Captured:**
- Verification confidence score
- Liveness check result
- Document authenticity assessment
- Image quality score
- OCR confidence level

**Integration Documentation on VM:**
- `FINAL_BLINK_HERRING_INTEGRATION_COMPLETE.md`
- `SPM_BLINK_INTEGRATION_GUIDE.md`
- `KYC_BRIDGE_INTEGRATION.md`

---

## 6. Apple Pay / Wallet Provisioning

### Architecture
```
User taps "Add to Apple Wallet"
    ↓
HerringWalletProvisioning.swift
    ↓
POST /vcards/provision (via middleware)
  Request: { provider: "APPLE", nonce, nonceSignature, certificateChain }
    ↓
Herring returns:
  { activationData, ephemeralPublicKey, encryptedData }
    ↓
PassKit processes provisioning data
    ↓
Card appears in Apple Wallet
```

**Key Implementation Details:**
- Uses native PassKit framework (no third-party SDK)
- Provider enum: `APPLE`, `GOOGLE`, `SAMSUNG`
- Apple requires: nonce, nonceSignature, certificateChain
- Google/Samsung: simplified flow, returns result string
- BIN Provider: MasterCard (unchanged in Herring migration)

---

## 7. Migration Status

### Component Migration Tracker

| Component | Old (i2c/MCPSDK) | New (Herring) | Status | Notes |
|-----------|-------------------|---------------|--------|-------|
| **Skinny Processor** | i2c MCP SDK | Herring Consumer API via Python | ✅ Scaffolded | Port 8003 |
| **Authentication** | MCPSDK auth | JWT via `/auth/token-sms` | ✅ Working | OTP flow tested in QA |
| **KYC/CIP** | MCPSDK KYC | BlinkIDUX + Herring Consumer API | ✅ Code complete | Needs Xcode build test |
| **Wallet/Apple Pay** | MCPInAppPushProvisioning | HerringWalletProvisioning + PassKit | ✅ Code complete | Needs device test |
| **Login Flow** | Admin API `/Account/login` | Consumer API `/auth/token-sms` | ❌ Not migrated | High priority |
| **User Profile** | Admin API `/Account/getcardholderprofile` | Consumer API `/users/profile` | ❌ Not migrated | |
| **Transactions** | Admin API `/Card/transactionhistory/{id}` | Consumer API `/transactions` | ❌ Not migrated | |
| **ACH/Funding** | Admin API `/Banking/b2ctransfer` | Consumer API `/ach` | ❌ Not migrated | |
| **Card Activation** | Admin API `/Card/activatecard` | Consumer API `POST /vcards` | ⏳ Blocked | Blocked on KYC verification test |
| **Bill Pay** | Admin API ECheckbook (9 endpoints) | **No Consumer equivalent** | ⛔ Gap | Not in Consumer API |
| **Profile Picture** | Admin API `/Account/updateprofilepicture` | **No Consumer equivalent** | ⛔ Gap | Not in Consumer API |

### What's Left for iOS MVP

**Must complete (in order):**
1. Login flow migration (Admin → Consumer API)
2. User profile migration
3. Transaction history migration
4. ACH/funding migration
5. Xcode build test of KYC flow
6. Device test of Apple Pay provisioning
7. Card activation end-to-end test

**Known gaps (needs product decision):**
- Bill pay: not available in Consumer API — defer or find alternative?
- Profile picture: not in Consumer API — store in middleware or defer?

---

## 8. iOS Build Environment

| Parameter | Value |
|-----------|-------|
| IDE | Xcode (latest stable) |
| Language | Swift 5+ |
| Legacy Bridge | Objective-C (APIManager.h/m) |
| KYC SDK | HerringMovoKycSDK (BlinkIDUX 7.3.1 via SPM) |
| Wallet SDK | PassKit (native iOS framework) |
| Encryption | libsodium (for password encryption) |
| BIN Provider | MasterCard |
| Target | iOS 15+ (verify in project settings) |

### Dependencies to Verify
- [ ] BlinkIDUX 7.3.1 SPM package resolves correctly
- [ ] libsodium Swift wrapper is available
- [ ] PassKit entitlements configured for Apple Pay
- [ ] Push notification certificates (for OTP delivery)
- [ ] App Store Connect provisioning profiles

---

## 9. Android Application

### Current Status: NOT STARTED

**Planned Stack:**
| Parameter | Value |
|-----------|-------|
| Language | Kotlin |
| Architecture | TBD |
| KYC SDK | BlinkIDUX (Android version) |
| Wallet SDK | Google Pay SDK |

**Key Architectural Decision:**
Both iOS and Android call the same Movo Skinny Processor middleware. The middleware is the single integration point — mobile platforms only need to implement:
1. UI/UX
2. Device features (camera, NFC, biometrics, wallet)
3. Local storage (tokens, preferences)
4. API calls to the Skinny Processor

All business logic, API orchestration, and Herring integration is handled server-side.

---

## 10. Data Flow — Mobile App to Bank

```
┌──────────────────┐
│  iOS App (Swift)  │
│                   │
│  • User Input     │
│  • Camera/NFC     │
│  • Apple Pay      │
│  • Local Storage  │
└────────┬─────────┘
         │  HTTPS/JSON
         ▼
┌──────────────────────────┐
│  Movo Skinny Processor   │
│  (Python Middleware)     │
│                          │
│  • API Orchestration     │
│  • HyperBIN Logic        │
│  • Token Management      │
│  • Business Rules        │
│  • Request Validation    │
│  Port: 8003              │
└────────┬─────────────────┘
         │  HTTPS/JSON + JWT
         ▼
┌──────────────────────────┐
│  Herring Bank            │
│  Consumer API v2.21.0    │
│                          │
│  • Account Management    │
│  • KYC/CIP Decisions     │
│  • Card Issuance         │
│  • Transaction Processing│
│  • Balance (SoR)         │
└──────────────────────────┘
```

---

*MovoCash Mobile App Architecture v1.0 — "The app is the presentation layer. The middleware is the brain."*
---

## App Architecture & Workflows (MANDATORY CONTEXT)

### High-Level Architecture

This iOS app follows a layered MVVM architecture with clear separation of concerns:

- **Views**
  - SwiftUI / UIKit screens
  - Handle rendering and user interaction only
  - No business logic or networking

- **ViewModels**
  - Own screen state and UI logic
  - Expose observable properties
  - Do not import UI frameworks when possible

- **Services / Managers**
  - Handle networking, persistence, and integrations
  - Stateless where possible
  - Injected into ViewModels via protocols

- **Routing / Navigation**
  - Centralized navigation logic
  - Views do not push controllers directly
  - Navigation decisions flow through ViewModels / Router

---

### App Startup Flow

1. App launches
2. AppDelegate / SceneDelegate initializes core services
3. Dependency container is created
4. Initial route is resolved:
   - Logged-in user → Home
   - Logged-out user → Auth / Signup
5. Root view is presented

Claude must respect this flow when modifying startup logic.

---

### Authentication & Signup Flow

- Entry point: Signup / Login screen
- Personal detail collection screens
- KYC flow via `KycBridge.swift`
- Consumer API path is used (Admin API paths are deprecated)
- Successful KYC → Account activation → Home

Claude MUST NOT reintroduce deprecated or deleted flows.

---

### Navigation Rules

- Navigation must follow defined routes
- No direct storyboard segues added without approval
- Deleted controllers must not be referenced
- All navigation changes must be tested in simulator

---

### Data Flow Rules

- Data flows from Services → ViewModels → Views
- Views never mutate shared state directly
- ViewModels expose intent-based methods (e.g. `submitForm()`)

---

### What NOT to Change Without Approval

Claude MUST ask before changing:
- App startup logic
- Authentication flow
- Signup / KYC routing
- Root navigation structure
- Persistent storage schemas


## Build, Verification & Execution Rules (MANDATORY)

### iOS Build Verification

For any iOS-related change, Claude MUST verify a clean build using:

```bash
xcodebuild clean build \
  -workspace Movo.xcworkspace \
  -scheme Movo \
  -sdk iphonesimulator \
  -destination 'platform=iOS Simulator,name=iPhone 15'

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/paggarwal2016)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/paggarwal2016)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
