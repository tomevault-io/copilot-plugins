## user-story-map

> Living user story map for Pet Circle. Tracks implementation status of every user story by role. Consult when working on any screen to understand what is done, what is partial, and what is missing. UPDATE this file after completing work on any story.


# Pet Circle -- User Story Map

Last updated: 2026-03-17 (Phase 2 in progress -- Firebase Auth + Firestore pets + Firestore subcollections)

## Update Protocol

After completing work on any user story listed below:
1. Change its status to reflect the new state (DONE / PARTIAL / MISSING)
2. Update the counts in the Summary section
3. Move resolved PARTIAL items to the DONE section
4. Add any newly discovered gaps to the appropriate section

---

## App Architecture

Two app-level roles: **Owner** and **Vet** (selected at sign-in).
Per-pet sharing via care circle with three roles:
- **Admin**: Pet creator. Full control (edit, delete, manage circle, measure).
- **Member**: Invited family/sitter. Can measure, view, add notes. Cannot edit pet or manage circle.
- **Viewer**: Typically the vet. Can view everything, add clinical notes. Cannot measure or edit.

Permissions enforced via `CareCircleRole` enum with `CareCirclePermissions` extension in `lib/models/care_circle_member.dart`.

Global active pet tracked in `petStore.activePetIndex` -- shared across all screens via header pet switcher.

---

## Status: DONE (63 stories) -- Phase 1 Complete + Phase 2 Partial

### Authentication & Onboarding
- A1: Welcome screen with sign-up and sign-in buttons
- A2: Role selection (Vet / Pet Owner) -- creates Firestore user doc if already authenticated
- A3-A7: Email auth, Google, Apple, password reset -- LIVE (kEnableFirebase = true)
- A8-A10: Email verification with resend and account switch -- LIVE
- A11: AuthGate routing widget -- routes based on AuthProvider.routeState
- A12: AuthProvider global singleton -- streams Firebase Auth + Firestore user profile
- B1: Onboarding Step 1 -- name, searchable breed dropdown (BreedSearchField), age
- B2: Onboarding Step 2 -- diagnosis saved to `Pet.diagnosis` field
- B3: Onboarding Step 3 -- target rate saved to `settingsStore`
- B4: Onboarding Step 4 -- care circle emails + role selection (Admin/Member/Viewer)
- B25: Onboarding Back/Next buttons with persistent state (AutomaticKeepAliveClientMixin)
- B26: Pet creation writes to Firestore via `PetStore.createPetWithFirestore()`, auto-adds owner as Admin

### Owner Experience
- B5: Owner dashboard with pet cards from `petStore` + empty state CTA
- B6: Pet card tap navigates to pet detail
- B7: Quick-action Measure / Trends buttons per pet
- B8: Manual tap-to-count with haptics and BPM calculation
- B9: Configurable timer durations (15s / 30s / 60s)
- B10: "Add to Graph" saves measurement for active pet via `measurementStore` + Firestore subcollection
- B11: Unified health trends -- single scrollable view with stats, chart, history
- B12: Trends time-range filtering -- period dropdown filters chart + stats
- B13: Medication add/edit wired to `medicationStore` + Firestore subcollection
- B14: Export measurement data -- CSV preview dialog
- B15: Pet detail reads from Firestore-backed `noteStore` and `measurementStore`
- B16: "View Graph" navigates to trends tab
- B17: Notifications from Firestore-backed `notificationStore` with tap-to-mark-read
- B18: Dark mode toggle
- B19: Language switcher (EN / HE)
- B20: SRR thresholds saved to Firestore-backed `settingsStore`
- B21: Care circle invite dialog from settings (role selection: Member/Viewer)
- B22: Export all data dialog from settings
- B23: Share with vet dialog from settings
- B24: Push / emergency notification toggles wired to Firestore-backed `settingsStore`
- M1: "Add Pet" button on owner dashboard navigates to onboarding
- M2: Edit pet profile -- admin-only, bottom sheet with searchable breed dropdown + Firestore persistence
- M3: Delete pet -- admin-only, long-press on pet card with confirmation
- M5: Global pet switcher in header -- `petStore.activePetIndex` shared across all screens
- M6: User profile management -- edit name/photo from settings
- M7: Sign out from settings drawer with confirmation
- M9: Care circle management -- remove members with confirmation
- M10: Measurement deletion -- swipe-to-delete in history with confirmation

### Medication
- E6: Standalone Medication screen at bottom nav index 3
- E7: Medication list with add, edit (tap to open pre-filled sheet), status, export

### Care Circle Sharing
- S1: CareCircleRole enum (admin/member/viewer) with permissions extension + Firestore serialization
- S2: Role-aware UI -- edit/delete only for admins, measure only for admin+member, viewer lock screen
- S3: Invite flow offers Admin/Member/Viewer role selection (updated from Member/Viewer only)
- S4: InvitationService -- creates Firestore invitation tokens, accepts via deep link
- S5: DeepLinkService -- parses `petcircle://invite?token=XYZ`, stores pending tokens
- S6: AuthGate handles invitation acceptance after authentication
- S7: Settings invite dialog calls InvitationService + copies invite link to clipboard
- S8: Remove member calls PetService.removeCareCircleMember() via Firestore
- S9: Shared pets appear on owner dashboard with Member/Viewer role badge
- S10: Delete pet calls PetService.deletePet() + UserService.removePetFromUser() via Firestore

### Vet Experience
- C1: Clinic overview dashboard from stores with real stats
- C2: Summary stats from stores
- C3: Patient card tap navigates to pet detail
- C4: Clinical notes persisted via `noteStore` + Firestore subcollection
- C5: Vet SRR trends -- data-driven chart
- C6: Notifications from Firestore-backed `notificationStore`

### Cross-Cutting
- E1: Bottom nav: Home, Trends, Measure, Medication
- E2: Settings drawer from avatar tap
- E3: Notifications drawer from bell icon (no dedicated tab)
- E4: RTL layout for Hebrew
- E5: Full dark mode support
- E8: Searchable breed dropdown as shared widget (`lib/widgets/breed_search_field.dart`)
- E9: All user-facing strings internationalized (enforced by cursor rule)
- E10: Design system tokens enforced (enforced by cursor rule)

---

## Status: Phase 2 IN PROGRESS

### Done in Phase 2
- Firebase Auth enabled (`kEnableFirebase = true`)
- Firestore user profiles via `UserService`
- Firestore pet persistence via `PetService` (create, delete, care circle management)
- `PetStore` streams pets from Firestore via `subscribeForUser(uid)`
- Measurements, notes, and medications stream from Firestore subcollections
- Pet edit persistence via `PetService.updatePet()`
- Parent `Pet.latestMeasurement` stays in sync with measurement subcollection writes
- User thresholds and preference toggles persist via Firestore-backed `settingsStore`
- In-app notifications persist via Firestore-backed `notificationStore`
- Dual-layer role architecture (App-Level: vet/owner + Care Circle: admin/member/viewer)
- Invitation flow with deep link acceptance
- Permission enforcement across all screens

### Remaining in Phase 2
- Push notifications via FCM
- Deploy repo-managed Firestore security rules
- Harden invitation acceptance so strict rules do not need a self-join exception

## Status: FUTURE (Phase 3-5)

| Phase | Stories |
|-------|---------|
| 3 | VisionRR camera measurement, trend anomaly detection, automated SRR alerts, measurement reminders, vet-to-owner messaging |
| 4 | PDF report generation, CSV file export (actual file), shareable health summaries, vet clinic analytics |
| 5 | Apple Watch / WearOS companion, offline mode with sync |

---

## Related Rules

- `screen-completion-guide.mdc` -- screen-level checklist, auth screens, remaining work, feature backlog
- `state-management.mdc` -- store patterns, access conventions, store registry, Firebase integration status, services
- `figma-design-system.mdc` -- design tokens, colors, typography, spacing, Figma workflow
- `design-system-enforcement.mdc` -- always-apply enforcement: tokens, components, i18n
- `bug-tracking.mdc` -- always-apply: log all bugs to `docs/bug-log.md`

---
> Source: [HiLuLiT/pet-circle](https://github.com/HiLuLiT/pet-circle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
