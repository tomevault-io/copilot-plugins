## grahak

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SSH Hotels Customer Frontend - a Flutter cross-platform mobile application for hotel booking (hourly/nightly reservations and co-living spaces). Targets iOS, Android, macOS, Linux, Windows, and Web.

**Live API:** `https://d2wpac93v0uoqt.cloudfront.net/api`
**Web URL:** `https://sshhotels.in`

## Build & Development Commands

```bash
# Install dependencies
flutter pub get

# Run the app (ensure .env file exists)
flutter run

# Run on specific device
flutter run -d chrome          # Web
flutter run -d ios             # iOS simulator
flutter run -d android         # Android emulator

# Lint/analyze
flutter analyze

# Build for production
flutter build web --release --base-href /
flutter build ios --release
flutter build apk --release

# Run tests
flutter test
flutter test test/specific_test.dart    # Single test file

# Generate launcher icons
flutter pub run flutter_launcher_icons
```

## Environment Setup

Create a `.env` file in the project root (required for asset bundling):
```
RAZORPAY_KEY=your_razorpay_key
googlemapsapi=your_google_maps_key
BACKEND_LOCATION_ENDPOINT=your_endpoint
```

## Architecture

### State Management
Uses **Provider with ChangeNotifier** pattern. Four main providers initialized in `main.dart`:
- `AuthService` - Authentication state and token management
- `BookingService` - Booking operations and state
- `FeedbackService` - User feedback handling
- `FavoritesService` - Wishlist management (auto-initializes on creation)

Access providers via: `Provider.of<ServiceName>(context, listen: false)` or `context.watch<ServiceName>()`.

### Directory Structure
```
lib/
├── main.dart              # App entry, Firebase init, MultiProvider setup, theme
├── config/                # API keys configuration
├── models/                # Data models (Property, Booking, User, Complaint, Notification)
├── screens/               # 34+ screen widgets
├── services/              # Business logic services (16 files)
├── utils/                 # Constants, design system tokens, validators, helpers
└── widgets/               # Reusable UI components
```

### API Service Pattern
- Singleton `ApiService` class with `apiService` global instance (`lib/services/api_service.dart`)
- Dio-based HTTP client with interceptors for auth token injection
- Request caching with 5-minute TTL
- **Important:** API expects snake_case field names (e.g., `firebase_token`, `phone_number`)
- 401 errors propagate to UI - no automatic token refresh

### Models
All models implement `fromJson`/`toJson` with snake_case to camelCase conversion:
```dart
factory Property.fromJson(Map<String, dynamic> json) {
  return Property(
    propertyName: json['property_name'],
    // ...
  );
}
```

### Design System
Use constants from `lib/utils/design_system.dart` for consistent styling:
- `AppSpacing` - 8px base spacing scale (xs, sm, md, lg, xl, xxl)
- `AppSizes` - Touch targets, icons, buttons, cards
- `AppRadius` - Border radius values
- `AppFontSizes` - Typography scale
- `AppElevations` - Shadow/elevation levels

Colors defined in `lib/utils/constants.dart`:
- `Constants.primaryColor` - Red (#E31E24)
- `Constants.backgroundColor` - Light gray (#F5F7FA)

### Deep Linking
Supports two URL formats:
- Web: `https://sshhotels.in/property/{phone}-{location}-{id}`
- Custom scheme: `sshcus://property/{id}`

Deep link handling is coordinated through `AppLoader` widget in `main.dart` with a global navigator key.

### Platform-Specific Considerations
- Web keyboard overlay fix applied via `_KeyboardFixWrapper` in `main.dart`
- Razorpay has web-specific stub implementation
- Use `kIsWeb` from `package:flutter/foundation.dart` for platform detection
- Import guards needed for platform-specific code (HTML/IO imports)

## Key Files

- `lib/main.dart` - App entry, theme, providers, deep linking
- `lib/services/api_service.dart` - HTTP client singleton, all API methods
- `lib/services/auth_service.dart` - Auth state, login/logout, token storage
- `lib/utils/constants.dart` - API URLs, colors, storage keys, endpoints
- `lib/utils/design_system.dart` - Spacing, sizing, typography tokens
- `lib/screens/bottom_navigation.dart` - Main navigation shell

## Deployment

**Web (Vercel):** `bash build.sh` - handles Flutter install, .env creation, and build
**CI/CD:** Codemagic configured for iOS releases (`.codemagic.yaml`)
**Firebase:** Project ID `sshhotels-724c1`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sankshipt-vishraam) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
