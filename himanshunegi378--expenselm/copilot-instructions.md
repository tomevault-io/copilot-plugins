## expenselm

> AI Instruction: Adhere to Feature-Based Folder Structure for React Projects


AI Instruction: Adhere to Feature-Based Folder Structure for React Projects

Objective: Implement and maintain a feature-based folder structure for React applications. This structure prioritizes modularity, colocation, and clear boundaries, enhancing scalability, maintainability, and developer experience.

Core Principles:
1.  Modularity: Features are self-contained units.
2.  Colocation: All files for a feature (components, hooks, API, types, tests) reside in its folder.
3.  Clear Boundaries: Features expose a minimal public API via an index.ts (or index.js). Internals are private.

Primary Directory Structure (src/):
-   src/features/: Contains feature-specific modules (e.g., src/features/UserProfile/).
-   src/shared/: Contains reusable, business-agnostic code.
-   src/pages/: Top-level route components, composing elements from features/ and shared/.
-   src/app/: (Optional) Global setup, providers, core routing, main entry points.

Internal Structure of a Feature Folder (e.g., src/features/FeatureName/):
-   index.ts: Mandatory. Public API of the feature. (e.g., export { MyComponent } from './components';)
-   components/: Feature-specific React components (e.g., UserProfileCard.tsx). Not for external reuse.
-   hooks/: Feature-specific React hooks (e.g., useUserProfile.ts).
-   api/ (or services/): Feature-specific API logic (e.g., userProfileApi.ts).
-   types/ (or interfaces/): Feature-specific TypeScript types (e.g., UserProfile.types.ts).
-   utils/: Feature-specific utility functions (e.g., profileHelpers.ts).
-   styles/: (Optional) Feature-specific global styles if not co-located.
-   assets/: Feature-specific static assets (e.g., default-avatar.png).
-   tests/ (or __tests__/): Tests for the feature's modules.
-   pages/: (Optional) For sub-pages if a feature is a mini-app.

Internal Structure of src/shared/:
Organize by type:
-   ui/: Generic UI components (Button.tsx, Modal.tsx).
-   lib/hooks/: Generic utility hooks (useDebounce.ts).
-   lib/utils/: Generic utility functions (formatDate.ts).
-   api/: Global API client setup, base configs.
-   config/: Application-wide configurations.
-   assets/: Globally shared static assets.
-   styles/: Global styles, themes.
-   types/: Global or widely shared types.

Rules for Code in src/shared/:
1.  Generic & Decoupled: Business-agnostic, not tied to specific feature logic.
2.  Cross-Feature Utility: Used by multiple (2-3+) distinct features. Avoid premature abstraction.
3.  Stable API.
4.  No Feature Dependencies: MUST NOT import from src/features/ or src/pages/.

Dependency Flow:
-   pages/ -> features/ (via index.ts), shared/
-   features/ -> other features/ (via index.ts), shared/
-   shared/ -> (CANNOT import from features/ or pages/)
-   app/ -> features/, shared/, pages/

Naming Conventions (Strict Adherence):
-   Feature Folders: kebab-case (user-profile) or camelCase (userProfile). Be consistent.
-   Component Files & Names: PascalCase.tsx (UserProfileCard.tsx), name UserProfileCard.
-   Hook Files & Names: useCamelCase.ts (useUserProfileData.ts), name useUserProfileData.
-   Service/API Files: camelCase.api.ts or entityName.service.ts.
-   Utility Files: camelCase.utils.ts or kebab-case.utils.ts.
-   Type Files: FeatureName.types.ts, entityName.types.ts, or types.ts (in feature types/).
-   Test Files: ComponentName.test.tsx, useHookName.test.ts.

Workflow Considerations:
-   Adding New Feature: Create src/features/NewFeatureName/, populate subdirectories, develop modules, export public elements via index.ts, integrate into pages/.
-   Modifying Existing Feature: Locate feature folder; changes stay within its boundaries.
-   Promoting to shared/: Verify criteria (generic, multi-feature use), refactor out feature specifics, move to shared/ subtype, update imports.

Pitfall Mitigation:
-   Bloated Features: Decompose into smaller, nested sub-features if a feature grows too large.
-   Circular Dependencies: Enforce dependency flow. Use ESLint rules if possible. Inter-feature communication via public index.ts APIs.
-   Premature Abstraction: Prioritize colocation. Abstract to shared/ only with clear reuse patterns (3+ features).

Tooling (Configuration Generation - if applicable):
-   Path Aliases (tsconfig.json / jsconfig.json):
    {
      "compilerOptions": {
        "baseUrl": "src",
        "paths": {
          "@/features/*": ["features/*"],
          "@/shared/*": ["shared/*"],
          "@/pages/*": ["pages/*"],
          "@/app/*": ["app/*"]
        }
      }
    }

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/himanshunegi378) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
