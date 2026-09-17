## dosify

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dosify is a Flutter-based medication management application that helps users track medications, doses, and schedules. The app supports both online (Firebase) and offline functionality with a sophisticated three-tier storage architecture.

## Development Commands

### Core Flutter Commands
- `flutter run` - Run the app in development mode
- `flutter build apk` - Build Android APK
- `flutter build ios` - Build iOS app
- `flutter clean` - Clean build artifacts
- `flutter pub get` - Install dependencies
- `flutter pub upgrade` - Update dependencies
- `flutter analyze` - Run static analysis
- `flutter test` - Run tests

### Platform-Specific Commands
- `flutter run -d android` - Run on Android device
- `flutter run -d ios` - Run on iOS device
- `flutter run -d chrome` - Run on web browser

### Build Commands
- `flutter build apk --release` - Build release APK
- `flutter build appbundle` - Build Android App Bundle
- `flutter build ios --release` - Build iOS release

## Architecture Overview

### Service-Oriented Architecture
The app uses a layered architecture with dependency injection:

1. **Service Layer**: Core business logic (Firebase, Encryption, Cache)
2. **Screen Layer**: UI components that inherit from `BaseServiceScreen`
3. **Widget Layer**: Reusable UI components
4. **Model Layer**: Data models with serialization support

### Key Services
- **FirebaseService**: Handles cloud and local data storage with encryption
- **EncryptionService**: Manages AES-256 encryption for sensitive data
- **CacheManager**: Multi-level caching for performance optimization
- **QueryOptimizer**: Optimizes Firestore queries with intelligent caching
- **ServiceLocator**: Dependency injection using GetIt

### Data Flow
```
User Action → Screen → Service Layer → Storage (Firebase/Local/Cache)
```

## Storage Architecture

### Three-Tier Storage System
1. **Firebase Firestore** (Primary): Cloud storage with real-time sync
2. **Encrypted Local Storage** (Secondary): AES-256 encrypted SharedPreferences
3. **Unencrypted Local Storage** (Fallback): Basic local storage for emergency access

### Data Persistence Strategy
- All operations work offline-first
- Data syncs to cloud when available
- Automatic failover between storage tiers
- Graceful degradation when services unavailable

## Key Models

### Medication Model
- Supports multiple medication types (tablet, capsule, injection variants)
- Includes inventory tracking and reconstitution calculations
- Handles injection-specific fields (route, diluent, concentration)

### Dose Model
- Tracks dose amounts with unit conversion
- Supports calculation-based dosing
- Links to medication for inventory management

### Schedule Models
- **Schedule**: Basic scheduling functionality
- **MedicationSchedule**: Medication-specific scheduling with inventory integration
- Both models support dose status tracking and calendar integration

## Screen Architecture

### BaseServiceScreen Pattern
All screens inherit from `BaseServiceScreen` which provides:
- Automatic service injection
- Loading state management
- Error handling with retry logic
- Consistent UI patterns

Example usage:
```dart
class MyScreen extends BaseServiceScreen {
  @override
  State<MyScreen> createState() => _MyScreenState();
}

class _MyScreenState extends BaseServiceScreenState<MyScreen> {
  @override
  Widget build(BuildContext context) {
    return buildServiceScaffold(
      appBar: AppBar(title: Text('My Screen')),
      body: () => _buildContent(),
    );
  }
}
```

## Key Dependencies

### Core Dependencies
- `firebase_core`, `firebase_auth`, `cloud_firestore` - Firebase integration
- `encrypt`, `crypto`, `flutter_secure_storage` - Security and encryption
- `provider`, `get_it` - State management and dependency injection
- `shared_preferences` - Local storage
- `uuid` - Unique ID generation

### UI Dependencies
- `flutter_form_builder`, `form_builder_validators` - Form handling
- `table_calendar` - Calendar components
- `intl` - Internationalization support

## Testing

### Running Tests
- `flutter test` - Run all tests
- `flutter test test/widget_test.dart` - Run specific test file
- `flutter test --coverage` - Generate coverage report

### Test Structure
Tests are located in the `test/` directory following Flutter conventions.

## Common Development Patterns

### Service Access
```dart
// Access services through BaseServiceScreen
final medication = await firebaseService.getMedication(medicationId);

// Or through ServiceLocator
final firebaseService = ServiceLocator.get<FirebaseService>();
```

### Error Handling
```dart
await executeWithLoading(
  () async => await firebaseService.addMedication(medication),
  onSuccess: (result) => Navigator.pop(context),
  onError: (error) => setError('Failed to save medication'),
);
```

### Loading States
```dart
setLoading(true);
try {
  // Perform operation
} finally {
  setLoading(false);
}
```

## Performance Considerations

### Caching Strategy
- Memory cache for frequently accessed data (30-minute TTL)
- Persistent cache for offline access
- Query optimization with intelligent cache invalidation

### Firestore Query Optimization
- Use compound queries sparingly
- Implement pagination for large datasets
- Cache query results with appropriate TTL

### Memory Management
- Properly dispose of streams and controllers
- Use const constructors where possible
- Implement lazy loading for large lists

## Security Best Practices

### Data Encryption
- All sensitive data encrypted with AES-256
- Secure key storage using platform security features
- Transparent encryption/decryption in service layer

### Authentication
- Firebase Authentication integration
- User-specific data isolation
- Permission-based access control

## Code Style Guidelines

### Naming Conventions
- Use descriptive variable names
- Follow Dart naming conventions (camelCase for variables, PascalCase for classes)
- Prefix private members with underscore

### Documentation
- Document all public APIs
- Include usage examples for complex functionality
- Maintain up-to-date README and architectural documentation

### Error Handling
- Use specific exception types
- Provide meaningful error messages
- Implement proper logging for debugging

## Troubleshooting

### Common Issues
1. **Firebase Connection**: Check internet connectivity and Firebase configuration
2. **Encryption Errors**: Verify encryption service initialization
3. **Local Storage**: Clear SharedPreferences if data corruption occurs
4. **Build Issues**: Run `flutter clean` and `flutter pub get`

### Debug Commands
- `flutter logs` - View device logs
- `flutter doctor` - Check development environment
- `flutter devices` - List connected devices

## Development Environment

### Requirements
- Flutter SDK 3.32.4+ (Dart 3.8.1+)
- Android Studio or VS Code with Flutter extensions
- Firebase project with Firestore and Authentication enabled
- Platform-specific SDKs (Android SDK, Xcode for iOS)

### Setup Steps
1. Clone repository
2. Run `flutter pub get`
3. Configure Firebase (add `google-services.json` for Android, `GoogleService-Info.plist` for iOS)
4. Run `flutter run` to start development

## Important Notes

### Firebase Database Setup
- The app includes fallback mechanisms for when Firebase database doesn't exist
- Firestore availability is checked and cached to prevent repeated connection attempts
- All operations gracefully degrade to local storage when Firebase is unavailable

### Medication Types
- The app supports various medication types including tablets, capsules, and multiple injection variants
- Injection types include: liquid vials, powder vials, pre-filled syringes, pre-filled pens, cartridges, and ampules
- Each type has specific handling for reconstitution, concentration calculations, and inventory management

### Data Migration
- The app handles legacy data formats and performs automatic migrations
- Multiple schedule model support for backward compatibility
- Gradual migration from unencrypted to encrypted storage

---
> Source: [kinemspa/dosify](https://github.com/kinemspa/dosify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
