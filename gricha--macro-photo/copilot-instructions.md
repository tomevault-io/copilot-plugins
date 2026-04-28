## macro-photo

> This document provides instructions for AI coding agents working on the MacroPhoto project. Follow these guidelines to effectively contribute to the codebase.

# AI Agent Instructions

This document provides instructions for AI coding agents working on the MacroPhoto project. Follow these guidelines to effectively contribute to the codebase.

## Project Overview

MacroPhoto is a mobile app (React Native + Expo) that uses AI (Claude or GPT-5) to analyze food photos and provide macro nutrient breakdowns. The project uses a **task-based development model** where each task in `TASKS.md` is designed to be completed independently.

## Architecture Snapshot (read first)

- Expo Router app with screens in `/app` and shared UI in `/components`.
- State: Zustand stores (e.g., analysis, settings) under `/lib`.
- AI: `services/ai.ts` uses Vercel AI SDK for Claude/OpenAI; environment keys loaded via `.env.local` and `expo-constants`.
- Media: `hooks/useCamera` + `expo-camera`, `hooks/useImagePicker` for library; tests can use a bundled sample image when `APP_ENV=test`.
- Styling: NativeWind/Tailwind classes; constants in `/constants/theme`.
- Tests: Jest for unit, Maestro for E2E; E2E flows live in `.maestro`.

## Before Starting Any Work

### 1. Read the Core Documentation

Always start by reading these files:

- `SPEC.md` - Complete product specification and requirements
- `TASKS.md` - All development tasks and their status
- This file (`AGENTS.md`) - Agent instructions

### 2. Understand the Current State

- Check the git status to see what files exist
- Look at recently modified files
- Understand what has been completed vs. what's pending

### 3. Identify Your Task

- Find the specific task you're working on in `TASKS.md`
- Ensure you understand the task's:
  - Description
  - Dependencies (complete those first if needed)
  - Acceptance criteria (how to know you're done)
  - Files to create/modify

## Task Selection Guidelines

### How to Pick a Task

1. **Check Dependencies**: Only work on tasks whose dependencies are completed
2. **Start Small**: If new to the project, start with smaller, UI-focused tasks
3. **One Task at a Time**: Complete one full task before moving to the next
4. **Follow the Order**: Tasks are generally ordered by logical flow

### Task Status Updates

**IMPORTANT**: Keep `TASKS.md` clean and focused on pending work only.
All active and future tasks live in `TASKS.md`; remove completed tasks entirely instead of marking them done.

When you start a task:

```markdown
### [~] TASK-XXX: Task name
```

When you complete a task:

1. **Remove it entirely from TASKS.md** - don't mark as [x], just delete the entire task section
2. This keeps the file focused only on what needs to be done
3. Avoids context pollution for the next agent

**Why remove completed tasks?**

- Agents pick up tasks from TASKS.md
- Completed tasks clutter the file and waste tokens
- Fresh agents should see only what's pending, not what's done
- Git history preserves what was completed if needed

## Development Guidelines

### Code Style

1. **TypeScript**: All code must be TypeScript (`.ts`, `.tsx`)
2. **Functional Components**: Use functional components with hooks
3. **NativeWind/Tailwind**: Use Tailwind classes for styling (once configured)
4. **Naming Conventions**:
   - Components: `PascalCase` (e.g., `Button.tsx`, `CameraView.tsx`)
   - Utilities: `camelCase` (e.g., `imageUtils.ts`, `storage.ts`)
   - Constants: `UPPER_SNAKE_CASE` for values, `camelCase` for files
   - Types/Interfaces: `PascalCase` (e.g., `MacroBreakdown`, `AIProvider`)

5. **File Organization**:
   ```
   /app          - Expo Router pages
   /components   - Reusable UI components
   /lib          - Utilities and helpers
   /services     - API clients and external services
   /hooks        - Custom React hooks
   /types        - TypeScript definitions
   /constants    - App configuration and constants
   /assets       - Images, fonts, etc.
   ```

### Component Structure

Follow this pattern for components:

```typescript
// components/Button.tsx
import { TouchableOpacity, Text, ActivityIndicator } from 'react-native';
import { ReactNode } from 'react';

interface ButtonProps {
  onPress: () => void;
  children: ReactNode;
  variant?: 'primary' | 'secondary';
  loading?: boolean;
  disabled?: boolean;
}

export function Button({
  onPress,
  children,
  variant = 'primary',
  loading = false,
  disabled = false
}: ButtonProps) {
  return (
    <TouchableOpacity
      onPress={onPress}
      disabled={disabled || loading}
      className={`p-4 rounded-lg ${
        variant === 'primary' ? 'bg-blue-600' : 'bg-gray-600'
      }`}
    >
      {loading ? (
        <ActivityIndicator color="white" />
      ) : (
        <Text className="text-white font-semibold text-center">
          {children}
        </Text>
      )}
    </TouchableOpacity>
  );
}
```

### TypeScript Types

Define types in `/types`:

```typescript
// types/nutrition.ts
export interface MacroBreakdown {
  mealName: string;
  calories: number;
  proteinG: number;
  carbsG: number;
  fatG: number;
  confidence: 'low' | 'medium' | 'high';
  notes?: string;
}

export type AIProvider = 'claude' | 'gpt5';

export interface AppSettings {
  aiProvider: AIProvider;
  claudeApiKey?: string;
  openaiApiKey?: string;
}
```

### API Integration Pattern

Follow this pattern for API clients:

```typescript
// services/claude.ts
import { MacroBreakdown } from '@/types/nutrition';

export class ClaudeClient {
  private apiKey: string;
  private baseUrl = 'https://api.anthropic.com/v1';

  constructor(apiKey: string) {
    this.apiKey = apiKey;
  }

  async analyzeFoodImage(imageBase64: string): Promise<MacroBreakdown> {
    try {
      const response = await fetch(`${this.baseUrl}/messages`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'x-api-key': this.apiKey,
          'anthropic-version': '2023-06-01',
        },
        body: JSON.stringify({
          model: 'claude-3-5-sonnet-20241022',
          max_tokens: 1024,
          messages: [
            {
              role: 'user',
              content: [
                {
                  type: 'image',
                  source: {
                    type: 'base64',
                    media_type: 'image/jpeg',
                    data: imageBase64,
                  },
                },
                {
                  type: 'text',
                  text: 'Analyze this food and return a JSON with macro breakdown...',
                },
              ],
            },
          ],
        }),
      });

      if (!response.ok) {
        throw new Error(`Claude API error: ${response.statusText}`);
      }

      const data = await response.json();
      // Parse and return MacroBreakdown
      return this.parseResponse(data);
    } catch (error) {
      console.error('Claude API error:', error);
      throw error;
    }
  }

  private parseResponse(data: any): MacroBreakdown {
    // Implementation
  }
}
```

### State Management

For app settings (TASK-024), use either:

**Option 1: Zustand (recommended for simplicity)**

```typescript
// lib/settingsStore.ts
import { create } from 'zustand';
import AsyncStorage from '@react-native-async-storage/async-storage';
import { AppSettings, AIProvider } from '@/types';

interface SettingsStore extends AppSettings {
  setAIProvider: (provider: AIProvider) => void;
  setApiKey: (provider: AIProvider, key: string) => void;
  loadSettings: () => Promise<void>;
  saveSettings: () => Promise<void>;
}

export const useSettings = create<SettingsStore>((set, get) => ({
  aiProvider: 'claude',
  claudeApiKey: undefined,
  openaiApiKey: undefined,

  setAIProvider: (provider) => {
    set({ aiProvider: provider });
    get().saveSettings();
  },

  setApiKey: (provider, key) => {
    set(provider === 'claude' ? { claudeApiKey: key } : { openaiApiKey: key });
    get().saveSettings();
  },

  loadSettings: async () => {
    const stored = await AsyncStorage.getItem('settings');
    if (stored) {
      set(JSON.parse(stored));
    }
  },

  saveSettings: async () => {
    const { aiProvider, claudeApiKey, openaiApiKey } = get();
    await AsyncStorage.setItem(
      'settings',
      JSON.stringify({
        aiProvider,
        claudeApiKey,
        openaiApiKey,
      })
    );
  },
}));
```

**Option 2: React Context**

```typescript
// contexts/SettingsContext.tsx
import { createContext, useContext, useState, useEffect, ReactNode } from 'react';
import { AppSettings, AIProvider } from '@/types';
import AsyncStorage from '@react-native-async-storage/async-storage';

const SettingsContext = createContext<{
  settings: AppSettings;
  updateProvider: (provider: AIProvider) => void;
  updateApiKey: (provider: AIProvider, key: string) => void;
} | null>(null);

export function SettingsProvider({ children }: { children: ReactNode }) {
  const [settings, setSettings] = useState<AppSettings>({
    aiProvider: 'claude',
  });

  // Load and save logic...

  return (
    <SettingsContext.Provider value={{ settings, updateProvider, updateApiKey }}>
      {children}
    </SettingsContext.Provider>
  );
}

export const useSettings = () => {
  const context = useContext(SettingsContext);
  if (!context) throw new Error('useSettings must be used within SettingsProvider');
  return context;
};
```

## Common Patterns

### Navigation (Expo Router)

```typescript
// app/index.tsx
import { router } from 'expo-router';

export default function CameraScreen() {
  const handleCapture = (imageUri: string) => {
    router.push({
      pathname: '/preview',
      params: { imageUri },
    });
  };

  return <CameraView onCapture={handleCapture} />;
}

// app/preview.tsx
import { useLocalSearchParams } from 'expo-router';

export default function PreviewScreen() {
  const { imageUri } = useLocalSearchParams<{ imageUri: string }>();

  return <ImagePreview imageUri={imageUri} />;
}
```

### Error Handling

```typescript
// Always handle errors gracefully
try {
  const result = await analyzeFoodImage(image);
  setResults(result);
} catch (error) {
  console.error('Analysis failed:', error);

  // Show user-friendly message
  Alert.alert(
    'Analysis Failed',
    'Unable to analyze the image. Please check your internet connection and API key.',
    [{ text: 'OK' }]
  );

  // Or use toast/snackbar
  Toast.show({
    type: 'error',
    text1: 'Analysis failed',
    text2: 'Please try again',
  });
}
```

### Loading States

```typescript
const [loading, setLoading] = useState(false);
const [error, setError] = useState<string | null>(null);

const handleAnalyze = async () => {
  setLoading(true);
  setError(null);

  try {
    const result = await analyzeFoodImage(image);
    router.push({ pathname: '/results', params: { result } });
  } catch (err) {
    setError(err.message);
  } finally {
    setLoading(false);
  }
};
```

## Code Quality Requirements

### CRITICAL: Quality Checks are MANDATORY

Before marking ANY task complete, you MUST:

1. **Run all quality checks** and ensure they pass
2. **Write tests** for all new code
3. **Fix all linting/formatting errors**
4. **Ensure type safety** (no TypeScript errors)

Failure to follow these requirements will result in poor code quality and broken builds.

### ⚠️ CRITICAL: NEVER SKIP OR DISABLE TESTS ⚠️

**YOU MAY NOT SKIP TESTS OR USE `.skip()` TO BYPASS FAILING TESTS.**

If changes you've introduced break existing tests, you MUST:

1. **Fix the test** to work with the new implementation
2. **Fix your code** if the test is catching a real bug
3. **Update the test** if requirements have legitimately changed

**DO NOT:**

- Use `.skip()`, `.only()`, or `x` prefixes to disable tests
- Comment out tests
- Delete tests without replacement
- Ignore test failures

**If you need to remove or modify a test, you MUST:**

1. **STOP and ask for permission first** by saying:

   ```
   ⚠️ IM ABOUT TO DO SOMETHING DANGEROUS AND POTENTIALLY STUPID. SHOULD I PROCEED?
   ```

2. **Explain clearly:**
   - Which test(s) you want to modify/remove
   - Why the test is failing
   - What you plan to do instead
   - Why this is the best solution

3. **Wait for explicit approval** before proceeding

**Why this is critical:**

- Skipping tests hides bugs and regressions
- Tests document expected behavior
- Breaking changes should be intentional and understood
- Future developers rely on tests to understand the codebase

**Remember:** If a test is failing, it's either catching a bug in your code or needs to be updated legitimately. Never silence it as a shortcut.

### Quality Tooling

This project uses a comprehensive quality tooling setup:

- **TypeScript**: Strict mode enabled with extra strictness rules
- **ESLint**: Linting for TypeScript, React, and React Native
- **Prettier**: Code formatting
- **Jest**: Unit and integration testing

### Running Quality Checks

⚠️ **IMPORTANT**: "Running tests" means BOTH unit tests AND e2e tests!

```bash
# Run ALL quality checks (type-check + lint + format + unit tests)
npm run quality

# Auto-fix what can be fixed, then run checks
npm run quality:fix

# Individual checks
npm run type-check  # TypeScript type checking
npm run lint        # ESLint (find issues)
npm run lint:fix    # ESLint (fix issues automatically)
npm run format      # Prettier (format all files)
npm run format:check # Check if files are formatted
npm run test        # Run unit tests
npm run test:watch  # Watch mode for tests
npm run test:coverage # Generate coverage report

# E2E Tests (Maestro)
npm run test:e2e           # Run all e2e tests
npm run test:e2e:smoke     # Run quick smoke test only
maestro test .maestro/<test-name>.yaml  # Run specific test
```

### When to Run E2E Tests

**ALWAYS run e2e tests when you:**

- Modify UI/screens (`app/*.tsx`, `components/*.tsx`)
- Change navigation logic
- Update button/interaction behavior
- Change testID values
- Complete tasks involving user flows

**E2E test workflow:**

1. Run unit tests first: `npm test`
2. Run smoke test: `npm run test:e2e:smoke` (validates basic app launch)
3. Run relevant e2e tests based on what you changed
4. If all pass, run full suite: `npm run test:e2e`

**E2E Test Documentation:**

- `.maestro/TEST_STATUS.md` - Current test status (READ THIS FIRST!)
- `.maestro/E2E_TEST_SUMMARY.md` - Complete test documentation

### Writing Tests (MANDATORY)

**EVERY new feature MUST have tests** - BOTH unit tests AND e2e tests where applicable.

See `TESTING.md` for comprehensive testing guide and `.maestro/E2E_TEST_SUMMARY.md` for e2e testing guide.

#### Test Requirements by Component Type:

**Utility Functions** (100% coverage required):

```typescript
// lib/__tests__/imageUtils.test.ts
describe('compressImage', () => {
  it('compresses image to target size', async () => {
    // Test implementation
  });

  it('handles errors gracefully', async () => {
    // Test error cases
  });
});
```

**API Clients** (90%+ coverage required):

```typescript
// services/__tests__/claude.test.ts
describe('ClaudeClient', () => {
  it('sends correct request format', async () => {
    // Mock fetch and test
  });

  it('parses response correctly', async () => {
    // Test response parsing
  });

  it('handles API errors', async () => {
    // Test error handling
  });
});
```

**UI Components** (70%+ coverage required):

```typescript
// components/__tests__/Button.test.tsx
import { render, fireEvent } from '@testing-library/react-native';

describe('Button', () => {
  it('renders correctly', () => {
    const { getByText } = render(<Button>Click Me</Button>);
    expect(getByText('Click Me')).toBeTruthy();
  });

  it('calls onPress when pressed', () => {
    const onPress = jest.fn();
    const { getByText } = render(
      <Button onPress={onPress}>Click Me</Button>
    );
    fireEvent.press(getByText('Click Me'));
    expect(onPress).toHaveBeenCalled();
  });
});
```

**Screens** (Integration tests AND e2e tests required):

```typescript
// app/__tests__/preview.test.tsx (unit/integration tests)
describe('PreviewScreen', () => {
  it('displays image preview', () => {
    // Test screen rendering
  });

  it('navigates to results on analyze', () => {
    // Test navigation
  });
});
```

**E2E Tests** (Maestro - for UI/screen changes):

```yaml
# .maestro/preview-screen.yaml
appId: host.exp.Exponent
---
- launchApp
- tapOn: 'temp-expo'
- waitForAnimationToEnd

# Test the actual user flow
- tapOn:
    id: 'capture-btn'
- waitForAnimationToEnd
- assertVisible:
    id: 'preview-screen'
- tapOn:
    id: 'analyze-button'
- assertVisible:
    id: 'loading-screen'
```

**When to write e2e tests:**

- ✅ New screens or major UI changes
- ✅ Navigation flow changes
- ✅ User interaction flows (photo capture, analysis, etc.)
- ✅ Critical user paths
- ❌ Pure utility functions (unit tests only)
- ❌ Internal helper functions (unit tests only)

### Test Coverage Requirements

**Unit Test Coverage:**

- **Minimum**: 70% overall coverage (enforced by jest.config.js)
- **Critical paths**: 90%+ coverage
  - API clients
  - Data processing
  - Business logic
- **UI components**: 70%+ coverage

**E2E Test Coverage:**

- **All user-facing screens**: Must have at least one e2e test
- **Critical user flows**: Must be fully covered (camera → preview → results)
- **Navigation paths**: Each navigation route should be tested

View coverage reports:

```bash
# Unit test coverage
npm run test:coverage
open coverage/lcov-report/index.html

# E2E test status
cat .maestro/E2E_TEST_SUMMARY.md
```

### Code Style Requirements

#### TypeScript

- ✅ Always use TypeScript (`.ts`, `.tsx`)
- ✅ No `any` types (will cause ESLint error)
- ✅ No non-null assertions (`!`) unless absolutely necessary
- ✅ All functions should have return type inference or explicit types
- ✅ Props interfaces for all components

```typescript
// ✅ Good
interface ButtonProps {
  onPress: () => void;
  children: ReactNode;
  variant?: 'primary' | 'secondary';
}

export function Button({
  onPress,
  children,
  variant = 'primary',
}: ButtonProps) {
  // ...
}

// ❌ Bad
export function Button(props: any) {
  // ...
}
```

#### React/React Native

- ✅ Functional components only
- ✅ Use hooks for state and side effects
- ✅ Follow React hooks rules (enforced by ESLint)
- ✅ No inline styles (use NativeWind classes)
- ✅ testID prop for elements that need testing

```typescript
// ✅ Good
export function Counter() {
  const [count, setCount] = useState(0);

  return (
    <View className="p-4">
      <Text className="text-lg">Count: {count}</Text>
      <Button testID="increment-btn" onPress={() => setCount(c => c + 1)}>
        Increment
      </Button>
    </View>
  );
}

// ❌ Bad
export function Counter() {
  const [count, setCount] = useState(0);

  return (
    <View style={{ padding: 16 }}>  {/* Inline styles */}
      <Text>Count: {count}</Text>
      <Button onPress={() => setCount(count + 1)}>  {/* Stale closure */}
        Increment
      </Button>
    </View>
  );
}
```

#### Error Handling

- ✅ Always handle errors in async functions
- ✅ Display user-friendly error messages
- ✅ Log errors for debugging
- ✅ Test error cases

```typescript
// ✅ Good
async function analyzeImage(uri: string) {
  try {
    const result = await apiClient.analyze(uri);
    return result;
  } catch (error) {
    console.error('Image analysis failed:', error);
    throw new Error('Unable to analyze image. Please try again.');
  }
}

// ❌ Bad
async function analyzeImage(uri: string) {
  const result = await apiClient.analyze(uri); // Unhandled error
  return result;
}
```

## Testing Your Changes

Before marking a task complete:

1. **Run Quality Checks**:

   ```bash
   npm run quality
   ```

   All checks must pass! If they don't, run `npm run quality:fix` and fix remaining issues.

2. **Run Unit Tests**:

   ```bash
   npm test
   ```

   All tests MUST pass. Coverage should not decrease.

3. **Run E2E Tests** (for UI/screen changes):

   ```bash
   # Quick smoke test first
   npm run test:e2e:smoke

   # Then run relevant e2e tests
   maestro test .maestro/camera-capture.yaml
   maestro test .maestro/preview-screen.yaml

   # Or run all e2e tests
   npm run test:e2e
   ```

   **When to run e2e tests:**
   - ✅ Modified any screen in `app/*.tsx`
   - ✅ Modified any UI component in `components/*.tsx`
   - ✅ Changed navigation logic
   - ✅ Updated testID values
   - ✅ Completed a task involving user flows

4. **Test on Both Platforms** (if possible):

   ```bash
   # iOS
   npx expo start --ios

   # Android
   npx expo start --android
   ```

5. **Check for TypeScript Errors**:

   ```bash
   npm run type-check
   ```

6. **Manual Testing**:
   - Navigate through all affected screens
   - Test error cases (no internet, invalid API key)
   - Test edge cases (large images, unknown food)

7. **Verify Acceptance Criteria**:
   - Go back to `TASKS.md`
   - Check each acceptance criterion
   - Only mark complete if ALL criteria met

## Common Pitfalls to Avoid

1. **Don't Skip Dependencies**: If TASK-X depends on TASK-Y, complete Y first
2. **Don't Hardcode Values**: Use constants, environment variables, or config
3. **Don't Ignore Errors**: Always handle errors gracefully
4. **Don't Over-Engineer**: Build exactly what the task requires, no more
5. **Don't Forget Types**: All functions, props, and state should be typed
6. **Don't Mix Patterns**: Be consistent with existing code style
7. **Don't Commit Secrets**: Use `.env` files, never commit API keys

## Package Installation

When installing packages for a task:

```bash
# Use expo install for expo packages (auto-matches versions)
npx expo install expo-camera expo-image-picker

# Use npm/yarn for other packages
npm install zustand
npm install @anthropic-ai/sdk

# Always install types if available
npm install -D @types/package-name
```

## Environment Variables

For tasks involving API keys (TASK-006, TASK-019, TASK-020):

1. **Create `.env.example`**:

   ```env
   CLAUDE_API_KEY=your_claude_api_key_here
   OPENAI_API_KEY=your_openai_api_key_here
   ```

2. **Use `app.config.ts`** (not `app.json`):

   ```typescript
   // app.config.ts
   import 'dotenv/config';

   export default {
     expo: {
       name: 'MacroPhoto',
       // ... other config
       extra: {
         claudeApiKey: process.env.CLAUDE_API_KEY,
         openaiApiKey: process.env.OPENAI_API_KEY,
       },
     },
   };
   ```

3. **Access in app**:

   ```typescript
   import Constants from 'expo-constants';

   const apiKey = Constants.expoConfig?.extra?.claudeApiKey;
   ```

## Git Workflow

1. **Check Status**: Before starting, run `git status`
2. **Create Commits**: Make logical commits per task or sub-feature
3. **Write Clear Messages**:

   ```
   git commit -m "TASK-009: Implement Camera screen UI

   - Created CameraView component
   - Added capture button and settings icon
   - Implemented basic layout with NativeWind"
   ```

4. **Don't Commit**:
   - `.env` files (should be in `.gitignore`)
   - `node_modules/`
   - Build artifacts
   - IDE config (unless intentional)

## When You're Stuck

If you encounter issues:

1. **Check Dependencies**: Is the required task actually complete?
2. **Read Docs**:
   - [Expo Docs](https://docs.expo.dev/)
   - [React Native Docs](https://reactnative.dev/)
   - [NativeWind Docs](https://www.nativewind.dev/)
3. **Check Examples**: Look at similar code in the project
4. **Ask for Help**: Document the blocker in `TASKS.md`

   ```markdown
   ### [!] TASK-XXX: Task name

   **Blocker**: Camera permissions not working on Android. Need to investigate...
   ```

## Communication

When completing a task, you can optionally add a completion note:

```markdown
### [x] TASK-014: Create reusable Button component

**Completed**: 2025-11-23
**Note**: Implemented with primary/secondary variants, loading state, and haptic feedback. Used NativeWind for styling. Component is fully typed and exported from components/index.ts.
```

## Quality Checklist

Before marking any task complete, verify:

### Code Quality

- [ ] **All quality checks pass** (`npm run quality`)
- [ ] Code compiles without TypeScript errors (`npm run type-check`)
- [ ] No ESLint errors (`npm run lint`)
- [ ] Code is properly formatted (`npm run format:check`)
- [ ] No `any` types or non-null assertions
- [ ] All imports use path aliases (e.g., `@/components/*`)

### Testing

- [ ] **Unit tests written** for all new code
- [ ] **All unit tests pass** (`npm test`)
- [ ] Coverage maintained or improved (70%+ overall)
- [ ] Critical paths have 90%+ coverage
- [ ] Edge cases tested (errors, loading states, empty data)
- [ ] Tests are isolated and don't depend on execution order
- [ ] **E2E tests written** for UI/screen changes
- [ ] **E2E tests pass** (`npm run test:e2e:smoke` minimum)
- [ ] E2E tests updated if testIDs changed
- [ ] E2E test documentation updated (E2E_TEST_SUMMARY.md)

### Functionality

- [ ] All acceptance criteria met
- [ ] Tested on iOS and Android (or both platforms where applicable)
- [ ] No console errors or warnings (except intentional logs)
- [ ] Error handling implemented with user-friendly messages
- [ ] Loading states work correctly
- [ ] UI matches design spec

### Code Standards

- [ ] Code is readable and maintainable
- [ ] No hardcoded values (use constants/config)
- [ ] No inline styles (use NativeWind)
- [ ] Props interfaces defined for components
- [ ] Error boundaries where appropriate
- [ ] Accessibility considered (testID, labels)

### Security & Best Practices

- [ ] No secrets committed (API keys, passwords)
- [ ] Input validation where needed
- [ ] No SQL injection/XSS vulnerabilities
- [ ] Sensitive data not logged

### Documentation

- [ ] Task status updated in `TASKS.md`
- [ ] Complex logic has comments
- [ ] New patterns documented in AGENTS.md or TESTING.md
- [ ] README updated if needed

**IMPORTANT**: You should NOT create git commits. Your job is to implement the code and ensure quality checks pass. The human developer will review and commit the changes.

## Project-Specific Notes

### AI Provider Support

- Always support BOTH Claude and GPT-5
- Make the provider switchable (don't favor one over the other)
- Parse responses into a normalized `MacroBreakdown` format

### Image Handling

- Compress images before sending to API (save costs and time)
- Support both camera capture and photo library selection
- Handle permissions gracefully

### UX Principles

- **Speed**: Minimize taps from camera to result
- **Simplicity**: Don't add features not in the spec
- **Beauty**: Use consistent spacing, typography, and animations
- **Feedback**: Always show loading/error states

## Summary

1. **Read** `SPEC.md` and `TASKS.md` first
2. **Pick** a task with completed dependencies
3. **Understand** the acceptance criteria
4. **Build** according to guidelines in this file
5. **Test** thoroughly on both platforms
6. **Update** `TASKS.md` when done
7. **Move** to the next task

Remember: Each task should be fully complete before moving on. Quality over speed.

---

**Last Updated**: 2025-11-23
**For Questions**: Review SPEC.md or check existing code patterns

---
> Source: [gricha/macro-photo](https://github.com/gricha/macro-photo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
