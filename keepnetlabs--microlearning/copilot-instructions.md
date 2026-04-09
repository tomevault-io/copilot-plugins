## microlearning

> This is a **microlearning platform** built with React and TypeScript. It provides interactive, scene-based learning experiences with support for SCORM API tracking, rich text editing, multiple languages, and customizable fonts.

# MicroLearning React Platform - Claude Code Guide

## Project Overview

This is a **microlearning platform** built with React and TypeScript. It provides interactive, scene-based learning experiences with support for SCORM API tracking, rich text editing, multiple languages, and customizable fonts.

**Key Features:**
- Scene-based content delivery (8 scene types)
- Edit mode for content creation
- SCORM API integration for learning tracking
- Dark mode and language support
- Rich text editor for content creation
- Performance optimized with React.memo and Framer Motion

## Tech Stack

- **Framework:** React 18 with TypeScript
- **Styling:** Tailwind CSS with dark mode support
- **Animations:** Framer Motion
- **State Management:** React Context API + Custom Hooks
- **Editor:** TinyMCE for rich text editing
- **Video:** HLS.js and Plyr for video playback
- **Utilities:** Zustand, Axios, Sonner (toasts)
- **Build:** Create React App
- **Tracking:** SCORM API Wrapper

## Project Structure

```
src/
├── components/
│   ├── common/              # Shared components (EditableText, FontWrapper, etc.)
│   ├── scenes/              # Main scene components (8 scene types)
│   │   ├── intro/
│   │   ├── goal/
│   │   └── {other-scenes}/
│   ├── ui/                  # UI components and modals
│   ├── icons/               # Icon components
│   └── figma/               # Figma integrations
├── hooks/                   # Custom React hooks (useLanguage, useIsEditMode, etc.)
├── contexts/                # Context providers (EditMode, FontFamily, GlobalEditMode)
├── services/                # Business logic (scormService, scormDownload, logger)
├── utils/                   # Utilities (constants, helpers, deepMerge, cssClasses)
├── types/                   # TypeScript type definitions
├── App.tsx                  # Main app component
└── main.tsx                 # Entry point
```

## Key Components and Architecture

### Scene System
The platform uses 8+ main scenes for different learning content types:
- **IntroScene** - Introduction with stats and animations
- **GoalScene** - Learning goals display
- **ScenarioScene** - Scenario-based content with video
- **ActionableContentScene** - Actionable learning content
- **CodeReviewScene** - Code review validation with AI feedback and hints
- **QuizScene** - Quiz with slider-based questions
- **NudgeScene** - Quick tips and nudges
- **SurveyScene** - Surveys and feedback
- **SummaryScene** - Content summary and takeaways

Each scene is wrapped with `React.memo` for performance and uses `EditModePanel` for editing capabilities.

**CodeReviewScene Features:**
- Backend validation API integration (http://localhost:4111/code-review-validate)
- Glassy-style feedback/hint UI with glass-border classes
- Real-time validation feedback with success/error icons
- Hint badge for learning tips
- Prevents scene progression until validation succeeds (unless in edit mode)
- Passes `outputLanguage` (app language) to backend alongside code language

### Edit Mode Context
The `EditModeContext` manages edit mode state globally:
- `useIsEditMode()` hook to access edit mode state
- `EditModePanel` component renders in edit mode
- Scene components toggle UI based on edit mode
- Controlled through `GlobalEditModeContext` in App.tsx

### Font Family Context
`FontFamilyContext` manages font family settings:
- `useFontFamily()` hook to access/update font
- `FontWrapper` component applies font styles to text
- Persists across scene changes

### Common Components
Reusable components in `src/components/common/`:
- **EditableText** - Editable text field with markdown support
- **FontWrapper** - Apply font family styling
- **ScientificBasisInfo** - Display scientific basis for content
- **MultiFieldEditor** - Multi-field content editor
- **EditModePanel** - Edit UI controls (shared across scenes)

## State Management Patterns

### Global State
- **EditModeContext** - Edit mode toggle
- **FontFamilyContext** - Font family selection
- **GlobalEditModeContext** - Global edit mode (optional)
- **App.tsx Scene Completion States:**
  - `quizCompleted` - Quiz validation status
  - `inboxCompleted` - Actionable content completion
  - `videoCompleted` - Scenario video completion
  - `codeReviewValidated` - Code review validation status (prevents next button unless success)

### Hooks for State Access
```typescript
const { isEditing, setIsEditing } = useIsEditMode();
const { fontFamily, setFontFamily } = useFontFamily();
const language = useLanguage();
const isDarkMode = useDarkMode();
```

### Local State
- Component-level state with `useState`
- Performance optimizations with `useCallback` and `useMemo`
- Dependencies properly managed

## Important Files and Entry Points

### Core Files
- **App.tsx** - Main component, imports all scenes with React.memo
- **main.tsx** - Entry point
- **src/services/scormService.ts** - SCORM API integration (tracking, scoring)
- **src/services/logger.ts** - Error logging utility
- **src/utils/constants.ts** - Application constants

### Scene Components
- `src/components/scenes/*.tsx` - Main scene components
- `src/components/scenes/{scene-name}/components/` - Scene-specific subcomponents
- `src/components/common/EditModePanel.tsx` - Edit mode controls

### Context and Hooks
- `src/contexts/EditModeContext.tsx` - Edit mode context
- `src/contexts/FontFamilyContext.tsx` - Font family context
- `src/hooks/useIsEditMode.ts` - Edit mode hook
- `src/hooks/useFontFamily.ts` - Font family hook
- `src/hooks/useLanguage.ts` - Language/translation hook

## SCORM Integration

The platform integrates with SCORM API for learning tracking:

**Key Service:** `src/services/scormService.ts`

**Functionality:**
- Initialize SCORM connection
- Track completion status
- Store and retrieve scores
- Log interactions and interactions
- Handle SCORM errors

**Usage Pattern:**
```typescript
if (scormService.isInitialized()) {
  scormService.setScore(score);
  scormService.setStatus("completed");
}
```

## Performance Optimizations

- **React.memo** - All scene components are memoized
- **useMemo** - Expensive computations (score calculations, complex rendering)
- **useCallback** - Stable function references for handlers
- **Lazy loading** - Non-critical components loaded with React.lazy
- **Animation optimization** - Framer Motion with proper variants
- **CSS optimization** - Tailwind with mobile-first design

## UI and Styling Patterns

- **Tailwind CSS** - All styling with responsive classes
- **Dark Mode** - CSS media queries with `dark:` prefix
- **Animations** - Framer Motion with motion variants
- **Themes** - Theme CSS classes applied via `useThemeClasses()`
- **Accessibility** - ARIA labels, semantic HTML, keyboard navigation

## Development Workflows

### Adding a New Scene
1. Create component in `src/components/scenes/NewScene.tsx`
2. Import necessary hooks and common components
3. Wrap with `React.memo` for performance
4. Include `EditModePanel` for edit mode
5. Add import in `App.tsx`
6. Export with memo: `export const NewScene = React.memo(...)`

### Working with CodeReviewScene
1. Scene validates code via backend API at `http://localhost:4111/code-review-validate`
2. Payload includes: `issueType`, `originalCode`, `fixedCode`, `language`, `outputLanguage`
3. Backend returns: `success`, `data.isCorrect`, `data.feedback`, `data.hint`
4. Use `onValidationStatusChange` callback to notify App.tsx of validation status
5. Next button disabled until validation succeeds (unless edit mode)
6. UI shows glassy-style feedback box with icon (✅/❌) and separate hint badge (💡)
7. Reset validation when code or language changes via `useEffect`

### Editing Content
1. Access `EditModePanel` via `useIsEditMode()` context
2. Use `EditableText` for text content
3. Use `RichTextEditor` for formatted content
4. Changes sync with SCORM service
5. Use logger utility for error handling

### Adding UI Components
1. Create in `src/components/ui/`
2. Type all props with interfaces
3. Use Tailwind CSS for styling
4. Export with named exports
5. Import and use in scenes/components

## Accessibility Standards

- Semantic HTML elements
- ARIA labels on interactive elements
- Keyboard navigation support
- Adequate color contrast (AA standard minimum)
- Alt text for images
- Focus management in modals/overlays

## Common Issues and Solutions

**Edit Mode not updating:** Ensure `EditModeContext` is provided in App.tsx and hooks use correct context.

**Styling not applied:** Check Tailwind CSS is built; verify class names are not dynamically constructed.

**SCORM not tracking:** Initialize with `scormService.initialize()` before using SCORM methods.

**Performance issues:** Use React DevTools Profiler; check for unnecessary re-renders; apply memo/useMemo/useCallback.

**Code Review validation not working:**
- Ensure backend API is running on http://localhost:4111/code-review-validate
- Check payload includes `language` (code language) and `outputLanguage` (app language)
- Verify response includes `success`, `data.isCorrect`, `data.feedback`, and `data.hint` fields
- CodeReviewScene must call `onValidationStatusChange` callback to update App.tsx state
- Edit mode bypasses validation (allows progression without success)


## Recent Updates

### CodeReviewScene Enhancement (Latest)
**Added:**
- Backend validation API integration for code review
- Real-time feedback UI with glassy-style design
- Hint badge for learning tips
- Scene progression guard - next button disabled until validation succeeds
- `onValidationStatusChange` callback to parent App.tsx
- `outputLanguage` payload field for app language tracking
- Validation state reset on code/language changes
- Error handling with fallback messages

**Files Modified:**
- `src/components/scenes/CodeReviewScene.tsx` - Added validation logic, UI feedback, hints
- `src/components/scenes/code-review/types.ts` - Added `onValidationStatusChange` prop
- `src/App.tsx` - Added `codeReviewValidated` state, validation check in `canProceedNext()`

## Cleanup Notes (Previous Refactoring)

**Removed:**
- 26 unused Radix UI packages
- 10 unused NPM packages (quill, cmdk, class-variance-authority, etc.)
- 3 unused files (fontFamilyUtils.ts, device.ts, useBottomSheet.ts)
- 10 unused UI component wrappers

**Result:** ~100 MB node_modules reduction, bundle size optimized

## Setup and Running

```bash
# Install dependencies
npm install

# Start development server
npm start

# Build for production
npm build

# Run tests
npm test

# Build and analyze bundle
npm run build:analyze
```

**Environment:** Node.js 16+, React 18, TypeScript 4.9+

## Resources

- [React Docs](https://react.dev) - React official documentation
- [TypeScript Docs](https://www.typescriptlang.org/docs) - TypeScript language docs
- [Tailwind CSS](https://tailwindcss.com/docs) - Tailwind styling framework
- [Framer Motion](https://www.framer.com/motion) - Animation library
- [SCORM Specifications](https://scorm.com) - SCORM API reference

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/keepnetlabs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/keepnetlabs)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
