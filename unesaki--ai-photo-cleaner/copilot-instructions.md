## ai-photo-cleaner

> This file contains important context for Claude Code when working on this project.

# Claude Code Development Context

This file contains important context for Claude Code when working on this project.

## Project Overview

AI Photo Cleaner is a React Native mobile app built with Expo that helps users automatically organize their photo libraries by detecting and removing duplicate photos.

## Key Technologies

- **Frontend**: React Native + Expo 53.0
- **Language**: TypeScript
- **Database**: SQLite with expo-sqlite
- **Navigation**: Expo Router (file-based routing)
- **Styling**: StyleSheet with custom design system
- **Photo Access**: expo-media-library
- **File System**: expo-file-system
- **Crypto**: expo-crypto for hashing

## Project Structure

```
app/(tabs)/
├── index.tsx          # Dashboard screen with analysis controls
└── explore.tsx        # Duplicate results screen with deletion UI

src/
├── services/
│   ├── DatabaseService.ts           # SQLite database operations
│   ├── PhotoService.ts             # Photo library access and metadata
│   └── DuplicateDetectionService.ts # Duplicate detection logic
├── types/index.ts      # TypeScript type definitions
└── utils/constants.ts  # Design system constants

components/ui/
├── ActionButton.tsx    # Reusable button component
├── StatCard.tsx       # Statistics display card
└── ProgressCard.tsx   # Progress indicator card
```

## Key Commands

```bash
npm start              # Start Expo development server
npm run lint          # Run ESLint
npx tsc --noEmit      # TypeScript type checking
npm run android       # Run on Android
npm run ios          # Run on iOS
```

## Development Guidelines

1. **Always run TypeScript and lint checks** before committing
2. **Use the existing design system** defined in `src/utils/constants.ts`
3. **Follow file-based routing** with Expo Router
4. **Maintain type safety** with TypeScript
5. **Use existing service patterns** for new features

## Database Schema

- `photos`: Photo metadata with hash values
- `duplicate_groups`: Groups of duplicate photos
- `duplicate_group_photos`: Junction table for group membership
- `analysis_sessions`: Analysis run history
- `user_settings`: App configuration
- `migrations`: Database migration tracking

## Migration System

The app uses a custom migration system for safe database schema updates:

- **MigrationService**: Handles database schema versioning and migrations
- **Migration tracking**: Records applied migrations in the migrations table
- **Rollback support**: Some migrations support rollback functionality
- **Version management**: Automatic detection of pending migrations

### Key Migration Features:
- Transactional migrations (rollback on failure)
- Sequential migration execution
- Migration status reporting
- Database reset functionality
- Comprehensive error handling

## Testing Strategy

- TypeScript compilation must pass
- ESLint must pass without warnings
- Manual testing on both iOS and Android
- Photo library permission testing
- Database operations testing

## Current Status

✅ **Completed MVP Features:**
- Photo library access with permissions
- Hash-based duplicate detection
- Interactive results UI with photo selection
- Safe deletion with confirmation dialogs
- Progress tracking and error handling
- SQLite database with full CRUD operations

🚧 **Next Development Phases:**
- Settings screen implementation
- Database migration system
- AI-based similarity detection (TensorFlow.js)
- Photo quality assessment
- Performance optimizations

## Important Notes

- App uses hash-based duplicate detection (MD5 of file metadata)
- All photo operations require proper permission handling
- Database operations are wrapped in transactions for safety
- UI follows iOS/Android platform conventions
- Error handling includes user-friendly messages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/unesaki)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/unesaki)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
