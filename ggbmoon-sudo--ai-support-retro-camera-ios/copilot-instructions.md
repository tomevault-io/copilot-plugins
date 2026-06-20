## ai-support-retro-camera-ios

> This repo is for an iOS-first AI Support retro camera app.

# AGENTS.md

This repo is for an iOS-first AI Support retro camera app.

The product is a retro film-style camera plus an AI photo coach. The MVP is not a full AI photo editing studio. The MVP focuses on single-photo capture, local retro presets, one-photo AI analysis, short actionable advice, Firebase-backed metadata/storage, basic history, privacy consent, and subscription scaffolding.

## Required Reading Before Every Task

Before making changes, always read:

- README.md
- docs/00-common-background-v2.md
- docs/01-product-mvp-scope.md
- docs/02-technical-architecture.md
- docs/09-codex-phase-plan.md
- docs/phase-log.md

For feature-specific work, also read the relevant report:

- Camera / filters / image pipeline: docs/03-camera-filter-image-pipeline.md
- AI photo advice: docs/04-ai-photo-advisor.md
- Firebase / Firestore / Storage / Cloud Functions: docs/05-firebase-storage-firestore-functions.md
- UI / UX / design system: docs/06-ui-ux-design-system.md
- Subscription / quota / StoreKit: docs/07-subscription-quota-storekit.md
- Privacy / security / App Store risk: docs/08-privacy-security-app-store-risk.md

For phase work, also read the relevant prompt in:

- docs/prompts/

## Core Architecture

Use the following architecture unless a later decision document explicitly changes it:

- iOS frontend: Swift + SwiftUI
- Camera: AVFoundation
- Photo import: PhotosPicker / PHPicker
- Local filters: Core Image
- Future advanced GPU work: Metal only if needed
- Local visual guidance: Vision
- Backend: Firebase-first
- Auth: Firebase Auth
- Database: Cloud Firestore
- Media storage: Firebase Storage
- Server logic: Cloud Functions v2
- Configuration: Remote Config
- Abuse protection: App Check
- Subscription: StoreKit 2 + StoreKit views
- AI: Cloud Functions server-side proxy with provider adapter
- MVP AI provider: Gemini paid tier / GeminiAnalyzer
- Future image editing provider: OpenAI image/edit adapter placeholder only

## Hard Rules

- Work on one phase only.
- Do not start the next phase unless explicitly requested.
- Do not delete existing working features.
- Do not rewrite the whole project unless explicitly requested.
- Keep changes small and reviewable.
- Do not add real API keys, secrets, Firebase project IDs, Apple credentials, Gemini keys, or OpenAI keys.
- Use placeholders, `.example` files, TODO comments, or mock providers where real external setup is required.
- Do not put Gemini / OpenAI API keys in the iOS app.
- AI API keys must be server-side only.
- Do not connect paid external services without explicit instruction.
- Do not add personalized ads SDKs in MVP.
- Do not implement true cloud real-time video streaming AI in MVP.
- Do not implement full AR, 3D pose overlay, complex skeleton tracking, social features, or a full AI editing studio in MVP.
- Do not claim a feature is complete if it only has placeholder code.
- If a manual Apple Developer / Firebase Console setup step is required, document it instead of inventing values.

## Privacy Rules

- User photos must not be assumed to be used for model training.
- `trainingConsent` should default to false.
- If future training / model improvement is added, it must be a separate, revocable, auditable opt-in.
- The MVP consent should explain third-party AI processing for photo analysis.
- Users must be able to delete a single photo.
- Users must be able to request account and data deletion.
- Logs should not contain raw image data, base64 image content, full signed URLs, or sensitive prompts.

## Documentation Rules

After every phase, update:

- docs/phase-log.md

When relevant, also update:

- README.md
- docs/decisions.md
- feature-specific docs
- docs/prompts/ if the phase plan changes

Every phase log entry should include:

- status
- date
- summary
- completed work
- changed files
- tests or manual checks
- known TODOs
- whether the repo is ready for the next phase

## Testing Rules

When code is added, include at least one of:

- unit tests
- UI tests
- emulator tests
- manual test checklist

For placeholder-only phases, include manual smoke test notes.

Before finishing a task, check:

- no real secrets committed
- project structure is still clean
- docs/phase-log.md is updated
- changed files are listed
- how to test is documented
- known TODOs are documented

## Phase Workflow

Use this workflow:

1. Read required docs.
2. Check docs/phase-log.md.
3. Confirm the requested phase.
4. Modify only the relevant files.
5. Add tests or manual test notes.
6. Update docs/phase-log.md.
7. Report changed files, how to test, known TODOs, and readiness for the next phase.

## Current Development Phases

- Phase 00: Repo setup and documentation
- Phase 01: Design system and navigation
- Phase 02: Auth
- Phase 03: Camera and photo picker
- Phase 04: Filter presets
- Phase 05: Firebase Storage and Firestore
- Phase 06: AI analyzePhoto Cloud Function
- Phase 07: AI result UI
- Phase 08: Quota system
- Phase 09: Subscription and paywall
- Phase 10: History, delete, and local download
- Phase 11: Privacy consent and account deletion
- Phase 12: Testing and release preparation

---
> Source: [ggbmoon-sudo/ai-support-retro-camera-ios](https://github.com/ggbmoon-sudo/ai-support-retro-camera-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
