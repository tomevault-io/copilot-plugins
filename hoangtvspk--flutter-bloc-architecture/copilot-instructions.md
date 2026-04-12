## flutter-bloc-architecture

> Always use the CLI command to add dependencies instead of manually editing pubspec.yaml:

 # Dependency Management Rules

## Adding Dependencies (Required)
Always use the CLI command to add dependencies instead of manually editing pubspec.yaml:

```bash
# Add a regular dependency
flutter pub add package_name

# Add a dev dependency
flutter pub add --dev package_name

# Add multiple dependencies at once
flutter pub add package1 package2 package3
```

## Common Package Groups

### State Management
```bash
# BLoC pattern
flutter pub add flutter_bloc bloc freezed freezed_annotation
flutter pub add --dev build_runner
```

### Network & API
```bash
# HTTP client
flutter pub add dio
flutter pub add --dev dio_web_adapter

# API code generation
flutter pub add json_annotation
flutter pub add --dev json_serializable
```

### Image Handling
```bash
# Image caching
flutter pub add cached_network_image

# SVG support
flutter pub add flutter_svg
```

### Storage
```bash
# Secure storage
flutter pub add flutter_secure_storage

# Shared preferences
flutter pub add shared_preferences
```

## Version Management
- Let Flutter handle version constraints automatically
- Use `flutter pub outdated` to check for updates
- Use `flutter pub upgrade` to update all dependencies
- Use `flutter pub upgrade --major-versions` for major version updates

## Version Conflicts
If you encounter version conflicts:
1. Run `flutter pub outdated` to see what's available
2. Update one package at a time using `flutter pub add package_name`
3. Test after each update
4. If conflicts persist, check the package's GitHub issues or documentation

## Security
- Always check package reputation on pub.dev
- Review security score and likes
- Verify maintenance status and last update
- Check GitHub issues for security concerns

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hoangtvspk)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hoangtvspk)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
