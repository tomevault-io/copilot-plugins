## kabete-poly

> - Release: https://github.com/shelad3/Kabete-Poly/releases/tag/v2.8.6%2B1

# Agent Context

## Current Version
- pubspec: `2.8.6+1`
- Release: https://github.com/shelad3/Kabete-Poly/releases/tag/v2.8.6%2B1

## Infrastructure
- Firebase project: `kabete-94936`
- Cloudinary cloud: `dpa8tbxdj`, upload preset: `Kabete_uploads`
- GitHub API for update checking: `https://api.github.com/repos/shelad3/Kabete-Poly/releases/latest`

## Key Firestore Rules
- `field_indices`: `allow read, create: if true` (public — needed because registration runs before auth user is created)
- `houses`, `cubes`: `allow read: if true` (public — guests can see availability)
- `cube_bookings`: `allow read: if isAuthenticated()`, `allow create, update: if isAuthenticated()`
- All other collections: `if isTeacherOrAbove()` or owner-based checks

## Composite Indexes (firestore.indexes.json)
- `messages` (conversationId + createdAt)
- `notifications` (studentId + type + read + createdAt)
- `lessons` (batchId + createdAt)
- `schedules` (batchId + date)
- `auth_codes` (studentId + used + createdAt)
- `auth_code_usage` (codeId + createdAt)
- `alerts` (studentId + read + createdAt)
- `lesson_verifications` (studentId + lessonId)
- `cubes` (houseId + cubeNumber)
- `cube_bookings` #1 (studentId + term + year + houseName) — for getMyBookingsStream
- `cube_bookings` #2 (term + year + houseName + cubeNumber) — for getAllBookingsStream
- `cube_bookings` #3 (cubeId + term + year + status) — for getBookedCountForCube
- `houses` (category + name) — for getHousesByCategoryStream

## Remaining Known Bugs / UX Issues
- (none currently tracked)

## Key Service Notes
- `UpdateService` clears `update_available` flag after download completes (not after install confirmation).
- `StorageService` wraps Cloudinary for all new uploads. Legacy image URLs may still reference Firebase Storage.
- `CubeService.getBookedCountForCube` uses 4-equality Firestore query (needs index #3 above).
- `HouseListScreen` filters out `reservedForNewStudents` houses for non-new students (checked via `UserProfile.enrolledTerm == currentTerm && enrolledYear == currentYear`).
- `BookingReceiptScreen` shown when user opens "Book a Cubicle" and already has an active booking.
- Guest houses view (`GuestHousesWidget`) shows occupancy via public-read Firestore queries.

---
> Source: [shelad3/Kabete-Poly](https://github.com/shelad3/Kabete-Poly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
