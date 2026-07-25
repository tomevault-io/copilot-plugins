## 100ms-react-native

> Project-specific guidance for Claude Code when working with the 100ms React Native SDK monorepo.

# CLAUDE.md

Project-specific guidance for Claude Code when working with the 100ms React Native SDK monorepo.

## Project Overview

- **Type**: React Native monorepo for video conferencing SDK
- **Main packages**:
  - `@100mslive/react-native-hms` - Core SDK with native modules (iOS/Android)
  - `@100mslive/react-native-room-kit` - Prebuilt UI components
  - `@100mslive/react-native-video-plugin` - Video player plugin
- **Sample apps**: HMSReactNativeSample, expo demo, callkeep demo, quickstart, virtual background
- **Package manager**: npm (NOT yarn)
- **Repository**: https://github.com/100mslive/100ms-react-native

## Development Environment

### Required Versions

- **Node.js**: v22.20.0 (ENFORCED via .nvmrc - always use `nvm use`)
- **npm**: v10.9.3 (comes with Node via nvm)
- **React Native**: 0.73.0+ (minimum), 0.77.3 (recommended)
- **React**: 18.2.0+
- **TypeScript**: 4.6.3 (react-native-hms), 5.0.2 (react-native-room-kit)
- **Java**: 17+ (specified in `.java-version` file)
- **Ruby**: For Fastlane and CocoaPods
- **iOS**: 16.0+ (minimum)
- **Android**: API 24+ (Android 7.0)

### Required Tools

- nvm (Node Version Manager)
- CocoaPods (`gem install cocoapods`)
- Fastlane (`gem install fastlane`)
- bundler (`gem install bundler`)
- Android SDK (for Android development)
- Xcode (for iOS development)

### Initial Setup

```bash
# Always use correct Node version
nvm use

# Install root dependencies
npm install

# For react-native-hms package
cd packages/react-native-hms
npm install
npm run prepack  # Builds the package

# For example apps
cd example
npm install
cd ios && pod install  # iOS only
```

## Monorepo Structure

```
100ms-react-native/
├── packages/
│   ├── react-native-hms/          # Core SDK with native modules
│   │   ├── src/                   # TypeScript source
│   │   ├── android/               # Android native code
│   │   ├── ios/                   # iOS native code
│   │   ├── lib/                   # Built output (gitignored)
│   │   └── example/               # Example app for testing
│   ├── react-native-room-kit/     # Prebuilt UI Kit
│   │   ├── src/                   # React components
│   │   └── example/               # Example app
│   └── react-native-video-plugin/ # Video player
├── sample-apps/                   # Standalone sample applications
├── scripts/                       # Build and utility scripts
├── .github/workflows/             # CI/CD configuration
└── release-apps.sh                # Release automation script
```

## Common Development Commands

### Package Development

```bash
# Build packages (required before testing changes)
cd packages/react-native-hms
npm run prepack  # Uses react-native-builder-bob

cd packages/react-native-room-kit
npm run prepack

# Lint code
npm run lint

# Type check
npm run typescript  # or npm run typecheck

# Run tests
npm test
```

### Example Apps

```bash
# Run Android
cd packages/react-native-hms/example
npx react-native run-android

# Run iOS
cd packages/react-native-hms/example
cd ios && pod install && cd ..
npx react-native run-ios
```

### Release Process

```bash
# Release sample apps (Android & iOS)
./release-apps.sh

# Options:
./release-apps.sh --android-only
./release-apps.sh --ios-only
./release-apps.sh --dry-run
./release-apps.sh --no-commit
```

## Code Style & Standards

### Prettier Configuration (MUST FOLLOW)

- Single quotes (not double)
- Tab width: 2 spaces
- Trailing commas: ES5 style
- No tabs (use spaces)
- Consistent quote props

### ESLint

- Extends: `@react-native-community`, `prettier`
- Pre-commit hook runs lint automatically
- Fix with: `npm run lint -- --fix`

### TypeScript

- Strict mode enabled
- Use proper types (no `any` unless necessary)
- Export types for public APIs

### Commit Messages (ENFORCED)

- Use Conventional Commits format
- Examples:
  - `feat: add screen sharing support`
  - `fix: resolve audio mute issue on Android`
  - `chore: update dependencies`
  - `docs: update installation guide`
- Pre-commit hooks run: `npm run lint && npm run typescript`

## Build & Release Process

### Package Building

- Uses `react-native-builder-bob` for compilation
- Outputs to `lib/` directory (commonjs, module, typescript)
- Always run `npm run prepack` after source changes
- Check `package.json` "files" field for published content

### Native Builds

**Android:**

- Gradle build system
- Build files: `android/build.gradle`, `android/app/build.gradle`
- Version bumping: Update `versionCode` and `versionName`

**iOS:**

- CocoaPods for dependency management
- Podspec file: `react-native-hms.podspec`
- Always run `pod install` after native changes
- Xcode project: `ios/RNExample.xcodeproj`

### Release Script

- `release-apps.sh` automates Android & iOS distribution
- Uses Fastlane lanes for build and deploy
- Automatically bumps versions
- Can commit changes or run dry-run

## Git Workflow & CI/CD

### Branches

- Main branch: `main`
- Feature branches: Create from `main`
- Always sync with remote before creating PRs

### Pre-commit Hooks

- Managed by Husky/Lefthook
- Automatically runs: lint, typescript check
- Must pass before commit is created

### CI/CD Workflows

- `build.yml` - Validates PR builds
- `release_apps.yml` - Automates app distribution
- `trunk-check.yml` - Code quality checks
- `vale.yml` - Documentation linting
- All workflows must pass for PR merge

### Publishing Packages

- Uses `release-it` with conventional changelog
- Semantic versioning (major.minor.patch)
- Publishes to npm registry

## Platform-Specific Notes

### iOS Development

- **Native code**: `ios/` directory in each package
- **Podspec**: Defines CocoaPods package (iOS 16.0+ required)
- **Permissions**: Add to Info.plist (Camera, Microphone, Network)
- **Minimum iOS**: 16.0
- **Common issues**:
  - Run `pod install` after dependency changes
  - Clean build folder if facing cache issues
  - Check signing & capabilities in Xcode

### Android Development

- **Native code**: `android/` directory in each package
- **Gradle files**: `build.gradle` (project and app level)
- **Permissions**: Add to AndroidManifest.xml
- **Minimum SDK**: 24 (Android 7.0)
- **Target SDK**: 35 (recommended)
- **Architectures**: arm64-v8a, x86_64 (64-bit only, 32-bit dropped)
- **Common issues**:
  - Gradle sync after dependency changes
  - Clean build: `cd android && ./gradlew clean`
  - Check JDK version matches `.java-version`

### Native Modules

- Both iOS and Android implementations required
- Use TurboModules pattern for new modules
- Test on both platforms before PR
- Document native setup in package README

## Testing Strategy

### Unit Tests

- Jest configuration in `package.json`
- Run with `npm test`
- Place tests in `__tests__` directories
- Mock native modules when needed

### Integration Tests

- Test package integration in example apps
- Verify both iOS and Android
- Test common user flows

### Manual Testing Checklist

- [ ] Test on iOS simulator/device
- [ ] Test on Android emulator/device
- [ ] Verify permissions work correctly
- [ ] Check video/audio functionality
- [ ] Test with different React Native versions

## Common Pitfalls & Troubleshooting

### Node Version Mismatch

- **ALWAYS** run `nvm use` before any command
- Project requires Node v22.20.0 (specified in .nvmrc)
- Shell should auto-switch if configured correctly

### Pod Install Issues

- Delete `ios/Pods` and `ios/Podfile.lock`
- Run `pod install --repo-update`
- Check Ruby version compatibility

### Gradle Build Failures

- Clean build: `cd android && ./gradlew clean`
- Check Java version matches `.java-version`
- Verify Android SDK is properly installed
- Check gradle wrapper version

### Metro Bundler Issues

- Clear cache: `npx react-native start --reset-cache`
- Delete `node_modules` and reinstall
- Check for conflicting dependencies

### Package Not Updated

- Always run `npm run prepack` after source changes
- Check `lib/` directory has latest compiled code
- Verify example app is using correct package version

### Native Module Linking

- React Native 0.60+ uses autolinking
- For manual linking issues, check:
  - iOS: Podfile includes package
  - Android: `settings.gradle` and `build.gradle` include package

## File Pointers

### Example Implementations

- SDK usage examples: `packages/react-native-hms/example/`
- Room Kit examples: `packages/react-native-room-kit/example/`
- Sample apps: `sample-apps/`

### Configuration Files

- ESLint/Prettier: Check `eslintConfig` and `prettier` in package.json
- TypeScript: `tsconfig.json`, `tsconfig.build.json`
- Build configuration: `react-native-builder-bob` in package.json
- CI/CD: `.github/workflows/`

### Build Scripts

- Release automation: `release-apps.sh`
- Fastlane: `packages/react-native-room-kit/example/android/fastlane/`
- Fastlane: `packages/react-native-room-kit/example/ios/fastlane/`

### Documentation

- Main README: `README.md`
- Package READMEs: `packages/*/README.md`
- Contributing: `CONTRIBUTING.md`
- Code of Conduct: `CODE_OF_CONDUCT.md`

## Developer Notes

### Before Starting Work

1. ✓ Run `nvm use` to ensure correct Node version
2. ✓ Pull latest changes from `main`
3. ✓ Run `npm install` in relevant directories
4. ✓ Check CI/CD status on GitHub

### When Making Changes

- **Native modules**: Always test both iOS and Android
- **Package changes**: Run `npm run prepack` before testing
- **Dependencies**: Update both package.json and package-lock.json
- **Breaking changes**: Update major version, document migration

### Before Committing

1. ✓ Run `npm run lint` - Fix any linting errors
2. ✓ Run `npm run typescript` - Fix any type errors
3. ✓ Run `npm test` - Ensure tests pass
4. ✓ Test on both platforms (if native changes)
5. ✓ Write clear commit message (Conventional Commits)

### Before Creating PR

1. ✓ Ensure all CI checks would pass
2. ✓ Update relevant documentation
3. ✓ Add tests for new features
4. ✓ Verify example apps work correctly
5. ✓ Check that no sensitive data is committed

### Package Publishing Checklist

1. ✓ Bump version following semver
2. ✓ Update CHANGELOG
3. ✓ Run `npm run prepack` to build
4. ✓ Test package in example app
5. ✓ Create git tag
6. ✓ Publish to npm
7. ✓ Create GitHub release

## Important Reminders

- 🚨 **NEVER** commit `node_modules/` or `lib/` directories
- 🚨 **NEVER** commit secrets, API keys, or credentials
- 🚨 **ALWAYS** use npm (not yarn) for consistency
- 🚨 **ALWAYS** test native changes on both platforms
- 🚨 **ALWAYS** run `npm run prepack` before testing package changes
- 🚨 Node version MUST be v22.20.0 (check with `node --version`)
- ⚡ Pre-commit hooks are mandatory (don't skip with --no-verify)
- 📱 Maximum 3-4 HMSView components per screen (performance)
- 🔄 Use conventional commits (enforced by commitlint)

---
> Source: [100mslive/100ms-react-native](https://github.com/100mslive/100ms-react-native) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
