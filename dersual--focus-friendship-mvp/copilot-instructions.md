## focus-friendship-mvp

> This project is for "Focus Friendship," a web application that gamifies focus sessions using a digital pet.

# Focus Friendship

This project is for "Focus Friendship," a web application that gamifies focus sessions using a digital pet.

## Approved Technical Plan (Updated)

### Phase 1: Core Pet & Progress MVP (Completed)
This phase focused on the essential loop: focusing to evolve a single pet.
-   **Backend & Setup:** The initial Express backend has been deprecated. All backend logic for session processing and XP calculation has been migrated to Firebase Cloud Functions.
-   **Core Services:** Services (XP, Pet, Goal, Shop) have been refactored to use Firestore for data persistence instead of `localStorage`.
-   **Core UI:** The `Timer` component and `CutieBean` pet component are now integrated with Firebase Firestore.

### Phase 2: Customization & Goals (Completed)
This phase expanded the core experience with more personalization.
-   **Expand Core Services:** Services now manage multiple pets and user-defined goals using Firestore.
-   **New UI - Goals:** The `GoalManager` component is integrated with Firestore.
-   **New UI - Pet Switching:** The `PetSelector` component is integrated with Firestore.

### Phase 3: Firebase Backend & Social Features
This phase formalizes the full Firebase backend and looks towards future social features.
1.  **Firebase Backend Implementation:**
    *   **User Authentication:** Anonymous Authentication using Firebase Auth is implemented.
    *   **Database Integration:** Firebase Firestore is fully integrated for user, pet, goal, and shop data persistence.
    *   **Server-side Logic:** Firebase Cloud Functions handle authoritative session processing and XP calculation.
2.  **Future Social & Scheduling Features:**
    *   **User Authentication:** Expand to include "Sign in with Google."
    *   **Friends System & Calendar:** Implement friends lists and Google Calendar integration (requires further development).

## Current Architecture

The application now features a Firebase-centric architecture:
-   **Frontend (Client):** A React application (`client/`) handles the user interface and interactions.
-   **Backend (Firebase Services):</b>
    -   **Firebase Authentication:** Manages user authentication (currently anonymous).
    -   **Firestore:** Serves as the primary database for all user data (XP, pets, goals, shop items).
    -   **Cloud Functions:** Hosts server-side logic, such as authoritative XP calculation and session processing, triggered by Firestore writes.
-   The previous Node.js/Express.js backend has been deprecated and its core logic migrated to Cloud Functions.

## Color Palette

-   **Text:** `#060f0f`
-   **Background:** `#f2f9fa`
-   **Primary:** `#5ab6bd`
-   **Secondary:** `#a4bedb`
-   **Accent:** `#7990c9`

## User-Defined Rules
- also explain and show everything step by step, after showing the plan, allow me a chance to approve or not approve it, and then do this step by step
- keep the 2 phase structure, just push back scheduling with friends to phase 3 ig (interpreted as a 3-phase structure)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dersual)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dersual)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
