## the-accountant

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**The Accountant** is a Flutter-based personal finance management application with a focus on privacy, local-first storage, and beautiful UI/UX. It features a monochromatic dark theme design and includes both free and premium AI-powered features.

## Common Development Commands

### Development & Running
```bash
# Install dependencies
flutter pub get

# Run code generation (for Riverpod and Drift)
dart run build_runner build

# Run with code generation watch (development)
dart run build_runner watch --delete-conflicting-outputs

# Run the app
flutter run

# Run on specific device
flutter run -d chrome    # Web
flutter run -d windows   # Windows
flutter run -d android   # Android
```

### Testing
```bash
# Run all tests
flutter test

# Run specific test file
flutter test test/widget_test.dart

# Run tests with coverage
flutter test --coverage
```

### Code Quality
```bash
# Analyze code
flutter analyze

# Format code
dart format .

# Check for outdated dependencies
flutter pub outdated
```

### Building
```bash
# Build APK for Android
flutter build apk

# Build app bundle for Google Play
flutter build appbundle

# Build for Windows
flutter build windows

# Build for Web
flutter build web
```

## Architecture Overview

### State Management
- **Riverpod** is used throughout for state management
- Providers are organized by feature in `lib/features/[feature]/providers/`
- Core providers (theme, notifications, etc.) are in `lib/core/providers/`
- Code generation is used with `riverpod_generator` for providers

### Database Layer
- **Drift** is used as the local-first database solution
- Database schema defined in `lib/data/datasources/local/app_database.dart`
- Models are in `lib/data/models/` with Drift table definitions
- Database operations use DAOs (Data Access Objects) pattern
- Run `dart run build_runner build` after schema changes

### Project Structure
```
lib/
├── app/                     # App-level configuration
├── core/                    # Core utilities and services
│   ├── constants/          # App constants
│   ├── providers/          # Core providers (theme, notifications, etc.)
│   ├── services/           # Core services (backup, notifications, etc.)
│   ├── themes/             # App theming
│   └── utils/              # Utility functions
├── data/                   # Data layer
│   ├── datasources/local/  # Local database (Drift)
│   ├── models/             # Data models
│   └── repositories/       # Data repositories
├── features/               # Feature modules
│   ├── [feature]/
│   │   ├── providers/      # Feature-specific providers
│   │   ├── screens/        # UI screens
│   │   ├── services/       # Feature services
│   │   └── widgets/        # Feature widgets
└── shared/                 # Shared components
    ├── models/
    ├── services/
    └── widgets/
```

### Key Features Structure
- **Authentication**: Firebase Auth with Google Sign-In support
- **Transactions**: Full CRUD with categories and payment methods
- **Budgets**: Weekly/monthly budget tracking with notifications
- **Wallets**: Multi-wallet support (personal, business, family)
- **AI Features**: Premium features using Gemini API (OCR, insights, chat)
- **Themes**: Custom dark theme implementation with premium theme options

### Environment Configuration
- Uses `flutter_dotenv` for environment variables
- Environment files should contain API keys (Gemini, etc.)
- Firebase configuration handled through `firebase_options.dart`

### Premium Features
- AI-powered receipt scanning with OCR
- Gemini API integration for insights and chat assistant
- Monthly summaries and spending comparisons
- One-time purchase model through Google Play

## Development Guidelines

### Code Generation
Always run code generation after changes to:
- Riverpod providers (annotated with `@riverpod`)
- Drift database models or DAOs
- Any files using `part` directive with `.g.dart` files

### Database Changes
1. Update model files in `lib/data/models/`
2. Update `app_database.dart` to include new tables
3. Increment `schemaVersion` in `AppDatabase`
4. Run `dart run build_runner build`
5. Test database migrations

### Adding New Features
1. Create feature directory under `lib/features/`
2. Follow the established pattern: providers, screens, services, widgets
3. Add routes to `lib/app/app.dart` if needed
4. Update database schema if data persistence is required

### Testing
- Widget tests should use `ProviderScope` wrapper
- Mock providers for testing isolated functionality
- Test database operations with in-memory databases
- Integration tests for Firebase and API integrations

### Dependencies
Key dependencies include:
- `flutter_riverpod` - State management
- `drift` - Local database
- `firebase_core`, `firebase_auth` - Authentication
- `fl_chart` - Charts and graphs
- `google_mlkit_text_recognition` - OCR for receipts
- `flutter_secure_storage` - Secure token storage

---
> Source: [theprantadutta/the_accountant](https://github.com/theprantadutta/the_accountant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
