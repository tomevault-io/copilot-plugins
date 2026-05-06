## screen-completion-guide

> Screen-by-screen guide for Pet Circle. Phase 1 complete, Phase 2 in progress. Tracks navigation, role system, auth flow, remaining work, and feature backlog.


# Pet Circle -- Screen Completion Guide

## Phase Status: Phase 1 COMPLETE, Phase 2 IN PROGRESS

Phase 2 adds Firebase Auth, Firestore persistence, and the full dual-layer role architecture.

## Navigation Structure

```
Auth flow (kEnableFirebase = true):
  /auth-gate --> /welcome --> /role-selection --> /auth --> /verify-email --> /auth-gate --> /onboarding or /main-shell

Auth flow (kEnableFirebase = false):
  / (welcome) --> /role-selection --> /main-shell

Bottom Nav: Home(0) | Trends(1) | Measure(2) | Medication(3)
Trends: single unified view (stats + chart + history, no tabs)
Notifications: bell icon drawer only (no dedicated tab)
Medication: standalone screen at lib/screens/medication/medication_screen.dart
Active pet: global via petStore.activePetIndex, switched from header pet chip
```

## Auth Screens

| Screen | File | Purpose |
|--------|------|---------|
| AuthGate | `lib/screens/auth/auth_gate.dart` | Routes based on `AuthProvider.routeState`. Handles deep link invitation acceptance. |
| WelcomeScreen | `lib/screens/welcome_screen.dart` | Sign Up / Sign In entry point |
| RoleSelectionScreen | `lib/screens/auth/role_selection_screen.dart` | Vet / Pet Owner. Creates Firestore profile if user already authenticated. |
| AuthScreen | `lib/screens/auth/auth_screen.dart` | Email/password + Google/Apple sign-in |
| VerifyEmailScreen | `lib/screens/auth/verify_email_screen.dart` | Polls for verification, calls `authProvider.refresh()` |

## Role System (Dual-Layer Architecture)

**Layer 1 -- App-Level Roles** (selected at sign-up, stored in Firestore `/users/{uid}`):
- **Owner**: Creates pets, manages care circles, takes measurements
- **Vet**: Views vet dashboard, adds clinical notes, cannot create pets

**Layer 2 -- Care Circle Roles** (per-pet, stored in `/pets/{petId}/careCircle`):
- **Admin**: Full control (edit, delete, measure, manage circle). Auto-assigned to pet creator.
- **Member**: Can measure, view, add notes. Cannot edit pet or manage circle.
- **Viewer**: Read-only. Can view data and add notes. Cannot measure or edit.

**Permission enforcement:**
- `CareCirclePermissions` extension: `canMeasure`, `canEditPet`, `canManageCircle`, `canAddNotes`, `canDeletePet`
- `PetStore.currentUserRoleFor(petName)` resolves user's role (matches on uid first, falls back to name)
- Pet detail edit button: admin-only
- Dashboard delete (long-press): admin-only
- Dashboard measure button: hidden for viewers
- Measurement screen: lock screen for viewers
- Settings invite/remove: admin-only
- Invite flow: offers Admin/Member/Viewer role selection

## Shared Widgets

| Widget | File | Used In |
|--------|------|---------|
| `BreedSearchField` | `lib/widgets/breed_search_field.dart` | Onboarding Step 1, Pet Detail edit sheet |
| `OnboardingShell` | `lib/widgets/onboarding_shell.dart` | All 4 onboarding steps (Back/Next buttons) |
| `BottomNavBar` | `lib/widgets/bottom_nav_bar.dart` | MainShell (Home, Trends, Measure, Medication) |

## Remaining Work

### Phase 2 -- Remaining Firebase/Data Work
| Item | Details |
|------|---------|
| Push notifications | Current notification support is Firestore-backed in-app alerts; FCM is not yet integrated |
| Firestore security rules | Repo-managed `firestore.rules` are deployed, and invitation acceptance now requires a trusted pet-side `pendingInvites` entry that the rules can verify |

### Polish (Optional)
| Item | Details |
|------|---------|
| VisionRR placeholder | `measurement_screen.dart` -- Phase 3 feature |
| Care circle role change | Can remove members but can't change existing roles |
| Care circle pending invites | No display of pending invitations in UI |
| CSV file download | Export dialogs show preview but don't write actual files |

## Feature Backlog

| ID | Feature | Details | Priority |
|----|---------|---------|----------|
| FB-001 | Pet photo upload | Replace photo URL text field with image picker + Firebase Storage upload. Affects onboarding step 1, pet detail edit sheet, and pet card display. Uses `image_picker` (already in pubspec). Requires adding `firebase_storage` dependency and a `StorageService` for upload/download URLs. | Medium |
| FB-002 | Invitation email delivery | Currently, care circle invitations generate a deep link copied to clipboard but no email is sent. Need to integrate an email delivery mechanism (Firebase Extensions "Trigger Email", a Cloud Function with SendGrid/Mailgun, or similar) so that when an invitation is created via `InvitationService.createInvitation()`, the invited person receives an email with the invite link, pet name, inviter name, and assigned role. Affects onboarding step 4 invites, Settings > Care Circle > Invite, and Settings > Share with Vet. | High |
| FB-003 | Pet form validation | Add proper validation to pet onboarding and edit forms: required pet name (non-empty, max length), breed selection required, age validation (numeric, reasonable range), photo URL format check. Affects `onboarding_step1.dart`, `onboarding_step2.dart`, `pet_detail_screen.dart` edit sheet. Currently no fields are validated -- user can submit empty/invalid data. | Medium |
| FB-004 | Invite abuse prevention | Add safeguards to care circle invitations: email format validation before sending, prevent duplicate invites to same email for same pet, rate-limit invitations per user (e.g., max 10 per day), cap care circle size (e.g., max 20 members per pet), show warning when re-inviting an email that already has a pending invitation. Affects `onboarding_step4.dart`, `settings_screen.dart` invite dialog, and `InvitationService`. Consider Firestore security rules for server-side enforcement. | Medium |
| FB-005 | Diary view | Pet health diary for logging daily activities. Bottom nav adds a 5th "Diary" tab. Tapping opens a category picker sheet (Poop, Meal, Water, Weight, Vomit, Grooming, Mood, Custom) with color-coded icons. Each category opens a detail form with Date, Time, category-specific fields (e.g. Consistency/Color for Poop), optional Notes, and Save/Cancel. Requires new `DiaryEntry` model, `diary_store`, Firestore subcollection, diary list/timeline screen, and bottom nav update. Figma refs: `136-888`, `137-296`, `137-772`. | Low |
| FB-006 | Verify email spam hint | Update the verify-email screen (`verify_email_screen.dart`) to inform users that the verification email may land in their spam/junk folder. Add a new l10n key (e.g. `checkSpamFolder`) to both `app_en.arb` and `app_he.arb`, and display it below the existing `clickLinkToVerify` text. | High |
| FB-007 | Push notifications (FCM) | Integrate Firebase Cloud Messaging for real-time push notifications. Current notifications are Firestore-backed in-app alerts, and the settings toggle should be treated as an in-app preference until FCM exists. Requires: add `firebase_messaging` dependency, create device-token registration + permission prompts, store FCM tokens in Firestore user docs, implement Cloud Functions (or another trusted backend) to send notifications on events (new measurement, care circle invite accepted, medication reminder, SRR threshold alert), handle foreground/background notification display, wire settings toggle to enable/disable, and support notification tap routing to relevant screens. Affects `main.dart`, `notification_store.dart`, `settings_store.dart`, and new `lib/services/notification_service.dart`. | High |

## Phase 2 Completion Checklist

- [x] Firebase Auth enabled (kEnableFirebase = true)
- [x] AuthGate routing (unauthenticated/needsRole/needsEmailVerification/authenticated)
- [x] AuthProvider global singleton with routeState
- [x] UserStore unified API (currentUserUid, seedFromAppUser, etc.)
- [x] Firestore user profiles (UserService)
- [x] Firestore pet CRUD (PetService)
- [x] PetStore streams from Firestore (subscribeForUser)
- [x] Onboarding creates pet in Firestore + adds owner as Admin
- [x] Pet deletion via Firestore
- [x] Care circle member removal via Firestore
- [x] Invitation model + InvitationService
- [x] Deep link handling (app_links)
- [x] Invitation acceptance in AuthGate
- [x] Settings invite dialog calls InvitationService
- [x] Dual-layer role architecture enforced across all screens
- [x] Shared pets visible on owner dashboard with role badge
- [x] Subcollection operations wired to Firestore (measurements, notes, medications)
- [x] Pet edit wired to Firestore
- [x] Pet latest measurement snapshot synced to parent pet doc
- [x] Settings/preferences persisted to Firestore
- [x] In-app notifications persisted to Firestore
- [ ] Push notifications (FCM)
- [x] Production Firestore security rules deployed

## Phase 1 Completion Checklist

- [x] All data flows wired to stores
- [x] All buttons functional
- [x] Care circle role system (Admin/Member/Viewer)
- [x] Role-aware UI (edit/delete/measure permissions)
- [x] Multi-pet support (add, edit, delete, global switcher)
- [x] User profile management
- [x] Sign out
- [x] Measurement lifecycle (add, view, filter by period, delete)
- [x] Clinical notes persistence
- [x] Medication management (add, edit, list, export)
- [x] Unified health trends (single view, no tabs)
- [x] Searchable breed dropdown (shared widget)
- [x] Onboarding Back/Next with persistent state
- [x] Dark mode + i18n (EN/HE, enforced)
- [x] Design system tokens enforced via cursor rule

---
> Source: [HiLuLiT/pet-circle](https://github.com/HiLuLiT/pet-circle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
