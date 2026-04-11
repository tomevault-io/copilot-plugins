## campus-event-organization-hub

> CEOH is a specialized Android application designed for UCC (University of the Cordilleras) students and organizations to centralize event management, registration, and attendance tracking.

# Campus Event & Organization Hub (CEOH) - Project Overview

CEOH is a specialized Android application designed for UCC (University of the Cordilleras) students and organizations to centralize event management, registration, and attendance tracking.

## 🚀 Main Technologies
- **Platform:** Android (Exclusively)
- **Language:** Java
- **Build System:** Gradle (Kotlin DSL)
- **UI Framework:** Android Jetpack (AppCompat, Material Design, ConstraintLayout)
- **Navigation:** Fragment-based navigation using `BottomNavigationView` in `MainActivity`.
- **Backend (Architecture):** Mention of Firebase/Backend API in documentation for real-time updates and push notifications.

## 🏛️ Architecture & Structure
- **Core Package:** `com.example.campus_event_org_hub`
- **Key UI Components:**
    - `MainActivity`: Host for navigation and fragments.
    - `EventsFragment`: Centralized feed with search and chip-based filtering.
    - `NotificationsFragment`: Placeholder for push notifications.
    - `SettingsFragment`: Placeholder for user preferences.
    - `EventDetailActivity`: Detailed view for event information.
- **Data Models:**
    - `Event`: Serializable model containing title, description, date, tags, organizer, category, and image resources.
- **UI Logic:**
    - `EventAdapter`: Manages the `RecyclerView` for events, including real-time filtering logic for search and categories.

## 🛠️ Building and Running
To build and run the project from the CLI:

- **Build Debug APK:**
  ```powershell
  ./gradlew assembleDebug
  ```
- **Run Unit Tests:**
  ```powershell
  ./gradlew test
  ```
- **Run Instrumentation Tests:**
  ```powershell
  ./gradlew connectedAndroidTest
  ```
- **Install on Device:**
  ```powershell
  ./gradlew installDebug
  ```

## 📝 Development Conventions
- **Code Style:** Standard Java Android conventions.
- **Resource Management:**
    - Layouts: Defined in `app/src/main/res/layout/`.
    - Strings: Centralized in `app/src/main/res/values/strings.xml`.
    - Styles/Themes: Managed in `app/src/main/res/values/themes.xml`.
    - **Graphics:** Use 9-patch images (`*.9.png`) for stretchable UI elements like button backgrounds and container borders to prevent distortion.
- **Navigation:** Use fragments for top-level navigation and activities for detail views.
- **State Management:** Currently uses simple `Intent` passing for event details.
- **Filtering:** Search and category filtering are handled within the `EventAdapter` using `Filterable`.

## 📌 Technical Logic (Inferred from Docs)
- **Attendance System:** A "Single Redemption Code" logic for "Time In" and "Time Out" validation.
- **Tag-Based Filtering:** Events are restricted to students with matching course/department tags.
- **Notification Pipeline:** Officers trigger push notifications when publishing events, filtered by student tags.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rcmtreb)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rcmtreb)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
