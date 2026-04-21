## skillmap

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

**Development:**
- `npm start` - Start Expo dev server
- `npm run android` - Run on Android emulator/device
- `npm run ios` - Run on iOS (macOS only)
- `npm run web` - Run on web (limited functionality)

**TypeScript:**
- `npx tsc --noEmit` - Type check without building

**Cleaning:**
```bash
rm -rf node_modules package-lock.json
npm install
npx expo start -c  # Clear cache
```

## Environment Setup

Create `.env` file in project root:
```env
OPENAI_API_KEY=sk-proj-...  # Optional - app works in mock mode without it
NODE_ENV=development
```

The app gracefully falls back to mock ChatBot responses if `OPENAI_API_KEY` is not set.

## Architecture Overview

### Persistence Layer Architecture

This app uses a **hybrid persistence strategy** combining three storage mechanisms:

1. **SQLite (expo-sqlite)** - Primary database for all entities
   - Tables: `users`, `roadmaps`, `skills`, `roadmap_skills`, `chat_messages`
   - Managed via singleton `DatabaseService` (`src/services/DatabaseService.ts`)
   - Initialized on app startup in `App.tsx`

2. **SecureStore (expo-secure-store)** - Encrypted authentication tokens only
   - Stores: `AUTH_TOKEN` (user ID for session validation)
   - Never store sensitive data beyond tokens here

3. **AsyncStorage** - Cache layer for performance
   - Stores: `USER_DATA` (non-sensitive user object for quick access)
   - Synced from SQLite after mutations

**Critical Pattern:** All data mutations must update ALL three layers:
```typescript
// Example from AuthService.ts:77-106
await DatabaseService.createUser({...});           // 1. SQLite (source of truth)
await SecureStore.setItemAsync('AUTH_TOKEN', id);     // 2. SecureStore (session)
await AsyncStorage.setItem('USER_DATA', JSON.stringify(user)); // 3. Cache
```

### Authentication Flow

Authentication spans multiple services and contexts:

**Login/Signup:**
1. `AuthService.ts` validates credentials against SQLite
2. SHA-256 password hash comparison via `utils/validation.ts`
3. Session token stored in SecureStore
4. User object cached in AsyncStorage
5. `AuthContext.tsx` updates React state triggering navigation guard

**Session Restoration (App Start):**
1. `useAuth` hook calls `AuthService.verificarSessao()` on mount
2. Token retrieved from SecureStore and validated against SQLite
3. If valid: user restored; if invalid: token cleared and user logged out
4. Navigation guard in `AppNavigator.tsx` routes to login/main based on auth state

**Auth Guard Logic (`AppNavigator.tsx:101-194`):**
```
Auth Flow:
  Loading → Check SecureStore token → Validate in SQLite
    ├─ Invalid/None → Show: OnboardingCadastro → Login → Cadastro
    └─ Valid → Check onboarding status
         ├─ Not seen → Show: OnboardingLogin → MainTabs
         └─ Seen → Show: MainTabs (authenticated)
```

### Navigation Architecture

**Two-tier navigation system:**

1. **Stack Navigator** (AppNavigator.tsx) - Auth-aware routing
   - Conditionally renders auth screens vs. MainTabs based on `user` from AuthContext
   - Handles onboarding flow using AsyncStorage flags keyed by user ID
   - Prevents gestures on authenticated screens (`gestureEnabled: false`)

2. **Bottom Tab Navigator** (MainTabs component) - Authenticated screens only
   - Home, GeradorRoadmap, RoadmapTracker, ChatBot
   - Safe area insets handled for iOS notch/Android navigation

**Onboarding Pattern:**
```typescript
// AppNavigator.tsx:110-142
const onboardingKey = `${STORAGE_KEYS.ONBOARDING}_login_${user.id}`;
const seen = await AsyncStorage.getItem(onboardingKey);
setHasSeenOnboarding(!!seen);
```
Each user tracks onboarding separately. Reset on logout.

### Service Layer Patterns

All services follow these conventions:

**DatabaseService (Singleton):**
- `getInstance()` for single instance across app
- All methods async, return promises
- CRUD methods: `create*`, `get*`, `update*`, `delete*`
- SQL queries use parameterized statements to prevent injection
- Foreign keys with CASCADE deletes maintain referential integrity

**AuthService (Singleton):**
- Does NOT directly access storage - delegates to DatabaseService
- All methods return `AuthResult` or `IUser | null`
- Password operations delegated to `utils/validation.ts`
- Extensive console logging for debugging (remove in production)

**RoadmapService:**
- Handles roadmap generation (currently mock IA logic)
- Coordinates between DatabaseService (roadmaps + skills) and XP system
- Skills awarded on completion: +50 XP per skill, +500 XP per 100% roadmap

### State Management

**No Redux/Zustand** - uses React Context API only:

**AuthContext (`src/contexts/AuthContext.tsx`):**
- Provides: `user`, `isLoading`, `login`, `cadastrar`, `logout`, `atualizarXP`
- Single source of truth for auth state
- Wraps entire app in `App.tsx`

**Local State:**
- Screens use `useState` for UI state (loading, errors, form data)
- Custom hooks (`useAuth`, `useRoadmap`) encapsulate business logic
- No prop drilling - prefer context or component composition

### TypeScript Patterns

**Strict typing enforced:**
- All interfaces in `src/types/models.ts`
- DTOs separate from domain models (e.g., `LoginDTO` vs `IUser`)
- Service return types use discriminated unions (`{ success: boolean; error?: string; ... }`)
- No `any` types - use `unknown` and narrow with type guards
- Styles typed as `ViewStyle`, `TextStyle`, `ImageStyle` from React Native

**Alias handling:**
```typescript
// models.ts:28,43 - Supports both field names for compatibility
title?: string;  // Alias for name_carreira
is_concluded?: boolean;  // Alias for status === 'concluido'
```

### Constants Pattern

**All configuration in `src/constants/index.ts`:**
```typescript
COLORS.brand.primary, COLORS.bg.secondary, etc.
TYPOGRAPHY.fontSize.md, TYPOGRAPHY.fontWeight.bold, etc.
VALIDATION.email, VALIDATION.senha (regex patterns)
MESSAGES.auth.loginError, MESSAGES.roadmap.criado (user-facing strings)
STORAGE_KEYS.AUTH_TOKEN, STORAGE_KEYS.USER_DATA (storage keys)
```

**Never hardcode:** colors, font sizes, validation patterns, or user messages in components.

## Critical Constraints

### Database Initialization
DatabaseService MUST be initialized before any screen renders:
```typescript
// App.tsx:18-28
await DatabaseService.init();  // Creates tables if not exist
setDbInitialized(true);        // Only then render navigator
```

### Password Security
- Never store plaintext passwords
- SHA-256 hashing via `expo-crypto` (development only)
- For production: migrate to backend with bcrypt/Argon2
- Salt defined in `utils/validation.ts` (should be env var in production)

### XP System Rules
- Skills: 50 XP each (awarded on completion toggle)
- Roadmaps: 500 XP on 100% completion
- Level thresholds: 1000 XP per level (simplified linear progression)
- XP updates must propagate: SQLite → AsyncStorage → AuthContext state

### Roadmap-Skill Relationship
Skills are shared across roadmaps via junction table:
```sql
roadmap_skills (id, roadmap_id, skill_id, order, is_concluded)
  FK roadmap_id → roadmaps(id) ON DELETE CASCADE
  FK skill_id → skills(id) ON DELETE CASCADE
```
Deleting a roadmap cascades to roadmap_skills, but NOT to skills table.

## Common Patterns

### Form Validation
```typescript
// Real-time validation pattern (LoginScreen.tsx, CadastroScreen.tsx)
const [errors, setErrors] = useState<FormErrors>({});

const handleChange = (field: string, value: string) => {
  setFormData({ ...formData, [field]: value });
  setErrors({ ...errors, [field]: '' }); // Clear error on input
};

const validate = (): boolean => {
  const newErrors: FormErrors = {};
  if (!VALIDATION.email.test(email)) newErrors.email = 'Email inválido';
  // ... more validations
  setErrors(newErrors);
  return Object.keys(newErrors).length === 0;
};
```

### Loading States
```typescript
const [isLoading, setIsLoading] = useState(false);

const handleSubmit = async () => {
  setIsLoading(true);
  try {
    await someAsyncOperation();
  } finally {
    setIsLoading(false); // ALWAYS in finally
  }
};
```

### Safe Area Handling
All screens use `SafeAreaView` with flex:1 and backgroundColor matching `COLORS.bg.primary`.

## Troubleshooting

### "Database not initialized" errors
- DatabaseService.init() not awaited before screen render
- Check App.tsx initialization sequence

### Auth state not updating after login
- Verify AuthContext wraps NavigationContainer
- Check SecureStore permissions on physical device
- Confirm all three storage layers updated (SQLite + SecureStore + AsyncStorage)

### Onboarding loops infinitely
- AsyncStorage key must include user ID: `ONBOARDING_login_{userId}`
- Check AsyncStorage.setItem called on onboarding screen completion

### TypeScript errors on SQLite queries
- SQLite returns `any` - always type-cast results: `db.getFirstAsync<IUser>(...)`
- Handle null cases: `const user = await db.get(...); if (!user) return null;`

## Testing Notes

**Development Database Reset:**
```typescript
// In any screen, temporarily add:
import DatabaseService from '../services/DatabaseService';
await DatabaseService.clearAllData();  // Nuclear option
```

**Mock Data Population:**
See `BACKEND_DEV.md` for SQL insert examples and test user creation patterns.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/freitasbtw) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
