## state-management

> State management architecture for Pet Circle. Covers store patterns, access conventions, seeding from mock data, and how screens should read/write app state.


# Pet Circle -- State Management Rules

## Architecture Overview

Pet Circle uses **global `ChangeNotifier` stores** for all shared app state. Global preferences (locale, dark mode) use `ValueNotifier`. No third-party state management packages are required.

```
lib/stores/
├── pet_store.dart            # Pet profiles, active pet, care circle ops
├── measurement_store.dart    # SRR measurements per pet
├── note_store.dart           # Clinical notes per pet
├── medication_store.dart     # Medications per pet
├── notification_store.dart   # In-app notifications
├── user_store.dart           # Current user and role
└── settings_store.dart       # Thresholds, preferences
```

## Store Pattern

Every store follows the same structure:

```dart
import 'package:flutter/foundation.dart';

class ExampleStore extends ChangeNotifier {
  List<Item> _items = [];

  List<Item> get items => List.unmodifiable(_items);

  void addItem(Item item) {
    _items.add(item);
    notifyListeners();
  }

  void seed(List<Item> initial) {
    _items = List.of(initial);
    notifyListeners();
  }
}
```

### Rules

- IMPORTANT: Stores extend `ChangeNotifier`, **not** `ValueNotifier`
- IMPORTANT: All public getters return **unmodifiable** views (`List.unmodifiable`, `Map.unmodifiable`)
- IMPORTANT: Every mutation method MUST call `notifyListeners()` at the end
- State fields are private (`_items`), exposed only through getters
- Stores live in `lib/stores/`, one store per file, `snake_case.dart`
- Each store has a `seed()` method for initialization from mock data

## Global Store Instances

Each store file exports its own global singleton instance:

```dart
// In lib/stores/pet_store.dart:
final petStore = PetStore();

// In lib/stores/measurement_store.dart:
final measurementStore = MeasurementStore();
```

Stores are seeded in `main()` before `runApp()` via `_seedMockStores()` (only when `kEnableFirebase == false`). When Firebase is enabled, `PetStore` subscribes to Firestore streams via `subscribeForUser(uid)` in `AuthGate` and coordinates Firestore subcollection subscriptions for measurements, notes, and medications.

## Accessing Stores in Widgets

### Reading (rebuilds on change)

Use `ListenableBuilder` to rebuild a widget subtree when a store changes:

```dart
@override
Widget build(BuildContext context) {
  return ListenableBuilder(
    listenable: petStore,
    builder: (context, _) {
      final pets = petStore.ownerPets;
      return ListView(
        children: pets.map((p) => PetCard(pet: p)).toList(),
      );
    },
  );
}
```

For multiple stores, use `Listenable.merge`:

```dart
ListenableBuilder(
  listenable: Listenable.merge([petStore, measurementStore]),
  builder: (context, _) {
    // Rebuilds when either store changes
  },
)
```

### Writing (mutations)

Call store methods directly -- no context needed:

```dart
onPressed: () {
  measurementStore.addMeasurement(petStore.activePet!.id!, Measurement(
    bpm: calculatedBpm,
    recordedAt: DateTime.now(),
  ));
}
```

### One-time reads (no rebuild)

Access store getters directly when you don't need reactive rebuilds:

```dart
final currentUser = userStore.currentUser;
final activePet = petStore.activePet;
```

## Import Convention

```dart
import 'package:pet_circle/stores/pet_store.dart';
import 'package:pet_circle/stores/measurement_store.dart';
```

The global instance (e.g. `petStore`) is exported from the store file itself, NOT from `main.dart`.

## Store Registry

| Store | Global | Key Methods |
|-------|--------|-------------|
| `PetStore` | `petStore` | `addPet`, `createPetWithFirestore`, `updatePet`, `updatePetWithFirestore`, `removePet`, `removePetWithFirestore`, `getPetByName`, `getPetById`, `activePet`, `activePetIndex`, `setActivePetIndex`, `currentUserRoleFor`, `removeCareCircleMember`, `removeCareCircleMemberWithFirestore`, `subscribeForUser`, `cancelSubscription` |
| `MeasurementStore` | `measurementStore` | `addMeasurement`, `removeMeasurement`, `getMeasurements`, `latestForPet`, `countForPet`, `thisWeekCount`, `subscribeForPets`, `cancelSubscriptions` |
| `NoteStore` | `noteStore` | `addNote`, `getNotes`, `subscribeForPets`, `cancelSubscriptions` |
| `MedicationStore` | `medicationStore` | `addMedication`, `updateMedication`, `removeMedication`, `toggleMedication`, `getMedications`, `getActiveMedications`, `subscribeForPets`, `cancelSubscriptions` |
| `NotificationStore` | `notificationStore` | `seed`, `reset`, `subscribeForUser`, `cancelSubscription`, `addNotification`, `markRead`, `markAllRead`, `unreadCount` |
| `UserStore` | `userStore` | `seed`, `seedFromAppUser`, `setUser`, `setRole`, `currentUser`, `appUser`, `role`, `isVet`, `isOwner`, `currentUserUid`, `currentUserEmail`, `currentUserDisplayName`, `currentUserAvatarUrl` |
| `SettingsStore` | `settingsStore` | `seedFromAppUser`, `reset`, `updateThresholds`, `setPushNotifications`, `togglePushNotifications`, `setEmergencyAlerts`, `toggleEmergencyAlerts`, `setVisionRREnabled`, `toggleVisionRR`, `setAutoExport`, `toggleAutoExport`, `classifyStatus` |

## Providers

| Provider | Global | Purpose |
|----------|--------|---------|
| `AuthProvider` | `authProvider` | Firebase Auth state. Exposes `routeState` (loading/unauthenticated/needsEmailVerification/needsRole/authenticated), `firebaseUser`, `appUser`, `isAuthenticated`, `isEmailVerified`. Listens to `AuthService.authStateChanges` + `UserService.streamUser`. |

## Data Flow

```
When kEnableFirebase == false (mock mode):
  MockData (seed) --> Stores (ChangeNotifier) --> Screens (ListenableBuilder)
                           ^                            |
                           +---- User Actions ----------+

When kEnableFirebase == true (Firebase mode):
  Firestore streams --> PetStore.subscribeForUser(uid) --> Pet/Measurement/Note/Medication stores --> Screens
  AuthProvider --> AuthGate (routing) --> Screens
  User Actions --> stores/services --> PetService/UserService --> Firestore
```

## Active Pet Pattern

The active pet is tracked globally in `PetStore`:

```dart
final pet = petStore.activePet;          // Current active pet (or null)
final idx = petStore.activePetIndex;      // Current index
petStore.setActivePetIndex(newIndex);     // Switch pet (from header)
```

All data-driven screens (trends, measurement, medication) should read from `petStore.activePet` to stay in sync with the header pet switcher.

## Firebase Integration (Phase 2 -- In Progress)

`kEnableFirebase` in `main.dart` controls mock vs Firebase mode. Currently set to `true`.

### What's wired to Firestore
- `PetStore`: `subscribeForUser(uid)` streams all pets where user is in careCircle. `createPetWithFirestore()`, `removePetWithFirestore()`, `removeCareCircleMemberWithFirestore()` call `PetService` + `UserService`.
- `PetStore.updatePetWithFirestore()`: persists editable pet fields via `PetService.updatePet()`.
- `MeasurementStore`: keyed by `petId`, streams `/pets/{petId}/measurements`, writes via `PetService.addMeasurement()` / `deleteMeasurement()`.
- `NoteStore`: keyed by `petId`, streams `/pets/{petId}/notes`, writes via `PetService.addNote()`.
- `MedicationStore`: keyed by `petId`, streams `/pets/{petId}/medications`, writes via `PetService.addMedication()` / `updateMedication()` / `deleteMedication()`.
- `PetService.addMeasurement()` / `deleteMeasurement()`: resync parent `/pets/{petId}.latestMeasurement` after subcollection writes.
- `SettingsStore`: seeds from `AppUser.settings` and persists thresholds/toggles back to `/users/{uid}.settings`.
- `NotificationStore`: streams `/users/{uid}/notifications` and persists read state / newly created in-app notifications.
- `AuthProvider`: Listens to `AuthService.authStateChanges` + `UserService.streamUser`. Routes via `AuthGate`.
- `InvitationService`: Creates/accepts invitations in `/invitations` collection.
- `NotificationService`: Firestore CRUD for `/users/{uid}/notifications`.
- `DeepLinkService`: Handles `petcircle://invite?token=XYZ` deep links.
- Repo-managed Firestore rules now live in `firestore.rules` and are wired via `firebase.json`.

### What's NOT yet wired to Firestore (local store only)
- Push delivery is not wired yet; notifications are currently Firestore-backed in-app alerts only
- Invitation email delivery still needs a trusted backend sender (Cloud Functions, Trigger Email, etc.)

### Services

| Service | File | Purpose |
|---------|------|---------|
| `AuthService` | `lib/services/auth_service.dart` | Firebase Auth (email, Google, Apple) |
| `UserService` | `lib/services/user_service.dart` | Firestore CRUD for `/users/{uid}` |
| `PetService` | `lib/services/pet_service.dart` | Firestore CRUD for `/pets/{petId}` + subcollections (measurements, notes, medications) |
| `InvitationService` | `lib/services/invitation_service.dart` | Firestore CRUD for `/invitations/{token}` |
| `DeepLinkService` | `lib/services/deep_link_service.dart` | Deep link parsing for invitation tokens |

---
> Source: [HiLuLiT/pet-circle](https://github.com/HiLuLiT/pet-circle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
