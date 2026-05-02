## logpress-public

> Every file must be located in its designated folder according to its purpose. This ensures the project remains predictable and organized.

# The Sacred Folder Structure (Separation of Concerns)
Every file must be located in its designated folder according to its purpose. This ensures the project remains predictable and organized.

src
├── api/             # API clients and calls (Supabase, etc.)
├── assets/          # Images, fonts, animations
├── components/      # Reusable, pure UI components
├── config/          # Environment variables and configuration
├── constants/       # App-wide constants (route names, etc.)
├── hooks/           # Custom React hooks
├── i18n/            # Internationalization (translation files)
├── navigation/      # React Navigation logic and routers
├── screens/         # Screen components (Each screen can have its own folder)
├── store/           # Redux Toolkit state management
├── theme/           # Styling and theme (colors, fonts, spacing)
├── types/           # Global TypeScript types
└── utils/           # Helper functions (formatDate, validators, etc.)

Rule 2: The Single Source of Truth
All global application state (user session, settings, app data, etc.) will be managed exclusively using Redux Toolkit. The use of Context API for global state is forbidden. Context may only be used for library integrations (ThemeProvider) or for complex, component-specific state that does not require Redux.

Rule 3: Modular Navigation (Clean Navigation)
The App.tsx file must remain clean, containing nothing more than provider wrappers and the main navigation container. All navigation graphs, screen definitions, and routing decisions will be managed modularly within the src/navigation folder (e.g., AuthNavigator, AppNavigator, RootNavigator).

Rule 4: Consistent and Scalable Styling
styled-components will be the library of choice for all styling.
All design decisions, such as colors, fonts, and spacing, will be sourced from theme objects located in the src/theme folder.
The use of the traditional StyleSheet.create will be avoided, except in rare cases where performance is an absolute priority.

Rule 5: Lean and Purposeful Dependencies
Before adding a new library, an evaluation will be made to determine if the problem can be solved with existing tools. A single, well-vetted library will be chosen for each specific function (e.g., image picker, notifications). Library duplication and unnecessary packages, as seen in the lscursor project, will be avoided.

Rule 6: Clean Code Discipline
Absolute Imports: Absolute paths configured via tsconfig.json and babel.config.js (e.g., @/components/Button) will be used throughout the project instead of relative paths like ../../.
Strict Typing: The use of the any type will be actively avoided. The project will aim for full type-safety from the beginning.
Descriptive Naming: Variables, functions, components, and files will have clear, descriptive names that explain their purpose.

# Smart Multilingual Development Guidelines with i18n

## Core Translation Rules

### 1. i18n File Structure
- Store all translations in individual JSON files in `src/i18n/` folder
- Each language has its own JSON file: `tr.json`, `en.json`, `de.json`, etc.
- Use meaningful, consistent keys: `screen_element_description` format
- Maintain alphabetical order within files for easy maintenance
- Keep all language files synchronized with identical keys

### 2. Smart Translation Checks
When working with translations, ALWAYS:
- Check for missing keys across language files
- Verify all languages have the same translation keys
- Look for unused translation keys in the codebase
- Ensure consistent naming patterns
- Validate that dynamic content placeholders match

### 3. Optimized i18n Usage
```typescript
// Import i18n hook
import { useTranslation } from 'react-i18next';

// Use in components
const { t } = useTranslation();

// For static text
const message = t('welcome_message');

// For dynamic content with interpolation
const message = t('welcome_message', { name: userName });
```

### 4. Translation Key Validation
Before adding new keys:
- Search existing keys to avoid duplicates across all JSON files
- Use consistent prefixes (screen_element_action)
- Ensure all supported languages get the translation
- Check for similar existing keys that could be reused

### 5. Performance Optimizations
- Only load translations for current language via i18n
- Use React.memo for translation-heavy components
- Batch translation updates through i18n
- Avoid inline translation calls in render loops

### 6. Quality Assurance
- Run duplicate key detection before commits
- Validate translation completeness across language files
- Check for hardcoded strings in components
- Test UI layout with longest translations (German often longest)

## Smart Development Workflow

1. **Before Adding Text**: Search for existing similar translations in JSON files
2. **Key Creation**: Follow naming convention and check for conflicts across all languages
3. **Multi-language**: Add to ALL supported language JSON files immediately
4. **Validation**: Run checks for duplicates and missing keys
5. **Testing**: Verify UI works with all language text lengths

## i18n Configuration
- Main configuration in `src/i18n/index.ts`
- Fallback language: English (`en`)
- Default language: Turkish (`tr`)
- Supported languages: tr, en, zh, de, fr, id, it, ja, ms, ru, es, hi, ko, pl, th, vi

## Automated Checks to Implement
- Missing key detection across JSON files
- Unused key identification in codebase
- Hardcoded string detection
- Translation length validation for UI compatibility
- JSON file synchronization validation 

---
> Source: [hasaneyldrm/logpress-public](https://github.com/hasaneyldrm/logpress-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
