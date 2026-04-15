## gimlee-ui

> This file serves as a comprehensive guide for AI agents and developers working on the **Gimlee Web UI** project. It summarizes the project's goals, technical stack, and the specific engineering practices and "preached" principles followed during development.

# AGENTS.md

This file serves as a comprehensive guide for AI agents and developers working on the **Gimlee Web UI** project. It summarizes the project's goals, technical stack, and the specific engineering practices and "preached" principles followed during development.

---

### 1. Project Overview: Gimlee
**Gimlee** is a decentralized, peer-to-peer (P2P) cryptocurrency marketplace.
- **Goal**: Connect buyers and sellers directly for exchanging goods and services using crypto.
- **Core Innovation**: Trustless, non-custodial payment verification. Sellers provide a **viewing key** (read-only), allowing the platform to verify payments on the blockchain without ever taking custody of funds.
- **User Spaces**: Users can create their own "Spaces" (personalized public profile pages) to showcase their ads, build reputation, and establish a unique identity within the marketplace.
- **Current Focus**: Integration with **PirateChain (ARRR)**, with plans for Monero (XMR) and Firo (FIRO).
- **Vision**: A community-driven, secure, and transparent economy that eliminates financial intermediaries.
- **Ambition**: Gimlee is not a small-scale project; it is built to compete with global tech giants by providing a superior, privacy-centric, and decentralized alternative.

---

### 2. Technical Stack
- **Framework**: React 19 (TypeScript)
- **Build Tool**: Vite
- **UI Framework**: UIkit 3
- **Animations**: Framer Motion (`motion/react`)
- **Forms**: React Hook Form
- **I18n**: react-i18next
- **State Management**: Redux Toolkit & `react-redux`
- **API Client**: `fetch` (wrapped in `apiClient`)
- **PWA**: `vite-plugin-pwa` for offline support and "Add to Home Screen" capability.
- **Native Wrapper**: **Ionic Capacitor** for Android/iOS native builds.

---

### 3. Core Development Practices

#### **A. UIkit React Wrappers**
Instead of using raw HTML and UIkit classes, we use specialized React wrappers located in `src/components/uikit/`.
- **`forwardRef`**: Every component must use `forwardRef` to allow integration with libraries like `react-hook-form`.
- **`useUIKit` Hook**: Components that require UIkit's JavaScript logic (e.g., `Grid`, `Navbar`, `Sticky`) must use the custom `useUIKit` hook to manage the lifecycle (initialization and `$destroy`) of the UIkit instance.
    - **Event Handling**: When listening for UIkit events (e.g., `show`, `hide`), use `useEffect` with the `instance` or `ref` provided by `useUIKit` to ensure proper cleanup and React-state synchronization.
- **Prop Logic**: Components should map UIkit's class-based options (like `uk-button-primary` or `uk-form-width-medium`) to clean React props (`variant="primary"`, `formWidth="medium"`).
- **State Synchronization**: When syncing React state with complex UIkit instances (e.g., `Slider`), avoid re-initializing the component (which happens if the `index` prop changes frequently). Instead, use `useEffect` to call programmatic methods like `instance.show(index)` only when the internal UIkit index differs from the React state.
- **Handling Cloned Elements**: Components like `Slider` with `infinite: true` clone DOM elements. To robustly identify items in event listeners (like `itemshow`), use `data-index` attributes on the items. Clones will carry these attributes, allowing you to accurately map them back to your data.

#### **B. Animation & Motion Design**
- **Spring Physics**: We use Framer Motion to create a "tactile" feel. Transitions should use `type: 'spring'` with standardized parameters (**`stiffness: 400, damping: 40`**) to achieve a snappy, critically damped physical response.
- **Staggered Entrances**: When navigating to a new page or revealing a list of items (e.g., in Profile or Ads), use staggered animations (`staggerChildren` in container variants) to orchestrate a smooth, non-jarring entrance for children components.
- **Layout Transitions**: Use the `layout` prop on `motion` components. This ensures that when elements appear or disappear (like error messages or expanded card details), the surrounding UI smoothly adjusts rather than jumping instantly.
- **AnimatePresence**: Always wrap conditional elements (like `FormMessage` or expanded details) in `AnimatePresence` to enable smooth exit animations.
- **Detail Expansion**: For expandable cards, combine `layout` on the wrapper and `AnimatePresence` for the content to create a "tactile" expansion effect.

#### **C. Form & Validation Philosophy**
Our forms prioritize a "friendly" user experience over immediate error shouting:
- **Focus-Aware UX**:
    - **Don't Punish Typing**: Do not show the red "danger" status (`uk-form-danger`) while a user is actively focused on a field.
    - **Trigger on Blur**: Apply the "danger" status only after the user moves away from the field (`onBlur`).
    - **Helpful Guidance**: Show requirements (e.g., password rules) as `info` messages (using `uk-text-primary`) when a field is focused, even if the current input is valid.
- **Smart Debouncing**:
    - **Instant**: Local validation like Regex, password matching, and length checks should be immediate.
    - **Debounced (500ms)**: API-heavy checks (like checking if a username or email is available) must be debounced to reduce server load.
- **Submit Button**: The submit button should remain `disabled` until `react-hook-form`'s `isValid` state is true.

#### **D. Internationalization (I18n)**
- No hardcoded text in components. All strings must be extracted to `src/i18n.ts`.
- **Regional Settings**: Group internationalization-related preferences (e.g., Language, Preferred Currency) into a single, cohesive **"Regional Settings"** card. This provides a cleaner user experience than scattering these settings across multiple sections.
- **IETF Tags**: Use full IETF BCP 47 language tags (e.g., `en-US`, `pl-PL`) for resources and backend communication. This is strictly required for compatibility with backend `Accept-Language` headers and user preferences.
- Support both English (`en-US`) and Polish (`pl-PL`).

#### **E. Storybook-Driven Development**
- Every UIkit component wrapper must have a corresponding `.stories.tsx` file.
- Stories should demonstrate not just the basic component, but complex states like "Animated Validation" or "Focus-aware behavior" to ensure UI consistency.

#### **F. SASS & Styling Strategy**
- **No Inline Styles**: Avoid `style={{ ... }}`. Use UIkit utility classes or SASS.
- **Component Styles (CSS Modules)**:
    - Use CSS Modules for component-specific styles.
    - Name the file `[ComponentName].module.scss` and co-locate it with the component (e.g., `src/ads/pages/AdDetailsPage.module.scss`).
    - Import directly into the component: `import styles from "./[ComponentName].module.scss"`.
    - This ensures class names are scoped (shortened/hashed in production) and maintenance is localized.
    - **Centralized Variable Access**: To use UIKit variables and mixins in a SCSS module, import the centralized `src/styles/_uikit-imports.scss` partial. This ensures all variables are available without violating import order or duplicating code.
- **Global Overrides**:
    - Global UIkit variable overrides go in `src/styles/uikit-variables.scss`.
    - Global theme customizations go in `src/styles/main.scss`.
- **UIKit Variable & Feature Reuse**:
    - **Think Twice**: It is absolutely crucial to think twice before using any specific number, color, or value in a SCSS file. Always check if it is already provided by a UIKit variable.
    - **Responsive Consistency**: Use UIKit's SCSS media breakpoint variables (e.g., `$breakpoint-small-max`, `$breakpoint-medium`) instead of hardcoded pixel values in media queries to ensure consistent behavior across the app.
    - **Mixin Hooks**: Leverage UIKit hooks (e.g., `@mixin hook-card()`) and miscellaneous hooks (e.g., `@mixin hook-card-misc()`) to add custom rules. This approach avoids repeating selectors and maintains the correct CSS cascade.
    - **Import Order**: Follow the strict UIKit import order in `main.scss` (Custom variables → UIKit variables/mixins → Custom mixins/hooks → UIKit core) to ensure variables and mixins are correctly applied.

#### **G. Data Presentation Layouts**
- **Cards Over Tables**: Avoid using "soulless" tabular views for complex business entities (like Sales or Purchases). Instead, use rich, interactive card-based layouts.
- **Progressive Disclosure**: Use expandable cards to keep the UI clean. Show high-level info (ID, Status, Date, Total) on the card surface and detailed item lists or payment instructions within an expandable section.
- **Progressive Image Loading**: For media-heavy views (like galleries), use a progressive approach. Load lightweight thumbnails (`thumb-xs`) initially and upgrade to higher-resolution versions (`thumb-md`) only when an item becomes active or is swiped into view.
- **Hierarchical Traversal**: For navigating deep category trees, use a hybrid approach: **Miller Columns** on desktop (for high-density info) and a **Drill-Down "Stack"** on mobile (for focus and ease of use). Combine this with a breadcrumb path for context preservation.

#### **H. Backend Error Handling**
To ensure a consistent user experience, especially with localized messages, we follow a standardized approach for handling API failures:
- **`StatusResponseDto`**: The backend returns this DTO for most failures. It contains a `message` field with a localized error description and a `status` string for machine-readable error codes.
- **Priority of Messages**: Components must prioritize the `message` field from the API error response when displaying errors to the user.
- **I18n Fallbacks**: Always provide a generic fallback message using `t('auth.errors.generic')` or a domain-specific key (e.g., `t('ads.notFound')`) in case the backend does not provide a message or the request fails at the network level.
- **Global vs. Local Handling**:
    - **401 (Unauthorized)**: Handled globally by `apiClient`, which clears the token and redirects to the login page.
    - **403 (Forbidden)**: Handled locally by components. The `apiClient` throws the response body, allowing components to display specific reasons (e.g., "XYZ role required").

#### **I. Authentication & Session Management**
- **Unified Initialization**: The application must bootstrap using the `/api/session/init` endpoint. This call should include all necessary `decorators` (e.g., `accessToken`, `userProfile`, `featureFlags`) as query parameters to retrieve the essential application state in a single network round-trip.
- **Graceful Identity Fallbacks**: When a identity (user profile, seller, etc.) lacks an explicit avatar, use `GeometricAvatar`. This ensures a visually rich and unique representation for every identity in the system.
- **Async Context Strategy**: Components relying on `AuthContext` must account for its initial `loading` state. Ensure that critical UI elements (like the Navbar or protected routes) wait for the session to initialize to prevent flickering or incorrect "Guest" state flashes.

#### **J. Progressive Web App (PWA)**
- **Auto-Update**: The app is configured with `registerType: 'autoUpdate'`. This ensures that when a new version is deployed, the service worker will automatically update and the app will refresh to the latest version on the next visit (or immediately if possible).
- **Offline First**: PWA assets (HTML, JS, CSS, and core SVGs) are cached for offline use.
- **Manifest**: The `manifest` is defined in `vite.config.ts`. It MUST include `display: 'standalone'` and `start_url: '/'` to be considered installable by Chrome. It should include both SVG and PNG icons (192x192 and 512x512) for full platform compatibility.

#### **K. Native App (Capacitor)**
- **Web-to-Native Workflow**: Capacitor wraps the web build (`dist/`) into a native WebView.
    - **VCS Strategy**: Unlike `node_modules`, the `android` and `ios` directories **MUST NOT** be added to the root `.gitignore`. They contain the native project structure, configurations, and assets (icons, splashes) that are part of your source code. Capacitor provides its own `.gitignore` inside these folders to handle local build artifacts.
    - **CI/CD Build**: Since the native folders are committed, CI/CD platforms like GitHub Actions can build APKs/IPAs directly by setting up the appropriate environments (JDK for Android, Xcode for iOS) and running `./gradlew` or `xcodebuild`. Note: Capacitor 8+ requires JDK 21.
    - Use `npm run cap:sync` to sync web changes to native projects after a build.
    - Use `npm run cap:open:android` to open the native project in Android Studio/IntelliJ.
- **Status Bar**: Use `@capacitor/status-bar` to manage the status bar appearance. For light themes, ensure the background is set to `#ffffff`, `overlaysWebView` is set to `false`, and style is set to `Style.Light` (dark icons) for better visibility. Note that in Capacitor's `StatusBar` API, `Style.Light` corresponds to dark content for light backgrounds, while `Style.Dark` corresponds to light content for dark backgrounds.
- **Vibe Preservation**: Since Capacitor uses the system WebView, all UIkit styles, CSS Modules, and Framer Motion animations are preserved without modification.
- **Native APIs**: When access to native features (Biometrics, Camera, etc.) is needed, use official Capacitor plugins. Always check for platform availability (`Capacitor.isNativePlatform()`) before calling native-only code to maintain web compatibility.
- **Deep Linking**: Handle deep links within the application to provide a seamless transition from external sources or notifications to specific content within the app.

#### **L. UIkit Dialogs & Stacking**
- **Prefer UIkit over Native**: Always use `UIkit.modal.confirm` and `UIkit.modal.alert` instead of browser `confirm()` and `alert()`. UIkit dialogs offer better visual consistency and don't block the main thread.
- **Stacking Support**: When opening a dialog on top of another modal (e.g., a confirmation inside a purchase flow), always use the `{ stack: true }` option. This ensures proper layering and event handling in SPA and mobile environments.
- **React Event Delegation**: By default, UIKit moves modal elements to the end of the `<body>`. For React 17+, this breaks event delegation. Always set `container: false` in the modal options (both in the component prop and when using `useUIKit`) to keep the modal within the React root (`#root`).

#### **M. Global State Management (Redux)**
- **Domain State**: Use Redux (via Redux Toolkit) for complex, long-running business processes that must persist across page navigation (e.g., active purchases, shopping baskets).
- **Global UI Components**: State that controls high-level UI elements rendered at the root (like the `PurchaseModal` in `App.tsx`) must be managed via Redux.
- **LocalStorage Sync**: Persist critical global state to `localStorage` to ensure continuity after page refreshes or app restarts. Re-hydrate this state during store initialization.

#### **N. Focused Navbar & Navigation Context**
- **Focused Mode**: Use the "focused" navbar mode for detail pages and user-centric views (like User Spaces) to declutter the UI. This mode replaces the logo/main menu with a back button and a slot for page-specific content (e.g., ad titles, usernames, breadcrumbs).
- **`useNavbarMode` Hook**: Activate focused mode using this hook. It should be called at the top of the page component.
- **`NavbarPortal`**: Use this component to "teleport" custom content into the navbar's focused slot.
- **Context Preservation**:
    - **Passing State**: When linking to a details page, always pass the current URL (including search params) in the `state.from` property: `<Link to="..." state={{ from: location.pathname + location.search }}>`.
    - **Smart Back Button**: The `useNavbarMode` hook automatically detects this state and configures the navbar's back button to return the user to their exact previous context (preserving filters, sorting, and pagination).
    - **Auth Redirects**: Ensure that authentication redirects (e.g., to the login page) preserve and pass along this `from` state so that the user can return to their original context after logging in.
---

#### **O. Image Rendering & Placeholders**
- **Enhanced `Image` Component**: To provide a polished, high-end feel, always use the custom `Image` component (`src/components/Image/Image.tsx`) instead of raw `<img>` tags or the base UIKit wrapper.
- **Loading Placeholders**: This component automatically provides a subtle "shimmer" animation while images are loading, preventing jarring layout jumps and giving immediate visual feedback.
- **Layout Preservation**: Always provide `containerClassName` or `containerStyle` to the `Image` component to ensure the placeholder correctly matches the intended dimensions of the image.

#### **P. Safe Content Handling**
- **External Link Safety**: When displaying user-provided links (e.g., in descriptions), always warn the user before they leave the platform. Use `UIkit.modal.confirm` with `stack: true` and standardized i18n keys (`common.externalLinkWarning`) to provide a consistent safety checkpoint.
- **Sanitized Markdown Rendering**: For user-generated descriptions, use the standardized `Markdown` component. It ensures that:
    - **External Images are Prohibited**: Only images from the platform's API or local paths are rendered; others are replaced with a placeholder to prevent tracking and bandwidth hijacking.
    - **UIkit Styling**: Tables, lists, and headers are automatically wrapped in UIkit classes for visual consistency.
    - **Link Behavior**: External links automatically open in a new tab (`target="_blank"`) with appropriate security headers (`rel="noopener noreferrer"`) and an external link icon.

#### **Q. Handling Absolute Children in Animated Containers**
When using Framer Motion's `layout` or height animations on a container, `overflow: hidden` is often required to prevent jitter or scrollbars during the transition. However, this clips absolute-positioned children like dropdowns or autocomplete suggestions.
- **Convention**: Use a local state (e.g., `isExpanded`) and Framer Motion's `onAnimationComplete` prop to toggle the container's class between `overflow: hidden` (during animation) and `overflow: visible` (after expansion).
- This ensures that UI elements like `CitySelector` suggestions remain fully visible once the filter drawer is open.

#### **R. Localized Form Ornaments**
For inputs that require localized decorations (like currency symbols), the placement (prefix vs. suffix) must strictly follow the conventions of the user's current locale.
- **Implementation**: Use the `getCurrencyFormatInfo` utility from `src/utils/currencyUtils.ts`. It leverages the `Intl.NumberFormat` API to identify the correct symbol and its conventional position.
- **UI Logic**: Conditionally render the symbol span before or after the `Input` component based on the `isPrefix` boolean returned by the utility.

#### **S. URL-Driven Filter Synchronization**
Complex listing pages (like Browse Ads) must treat the URL as the "Single Source of Truth" for their filtering and sorting state.
- **Synchronization Pattern**:
    - Use `useSearchParams` from `react-router-dom`.
    - **URL to State**: Use a `useEffect` to sync query parameters to local component state whenever `searchParams` change.
    - **State to URL**: Use a `useCallback` (e.g., `applyFilters`) to update the `searchParams` when a user interacts with the filters.
- **Pagination Reset**: Always reset the page parameter (`p=0`) when any filter or sorting criteria is modified to prevent "empty page" states.
- **Active Filter Badges**: Provide a clear overview of applied filters using interactive badges that allow users to remove individual criteria and immediately trigger a URL update.

#### **T. Context-Aware Selection Modes**
Hierarchical selectors (e.g., `CategorySelector`) must adapt their behavior based on the task being performed:
- **Filtering Mode**: When used for searching, selectors should allow the selection of parent nodes (e.g., "All in Electronics") to broaden the search scope. Use the `allowParentSelection` prop to enable this.
- **Creation/Editing Mode**: When used for creating or editing a resource (like an ad), selectors must strictly enforce leaf-only selection to ensure the resource is accurately categorized in the database.
- **Visual Cues**: Parent selection options (e.g., "All in [Category]") should be visually distinguished (e.g., via bold text or primary color) to clarify their function.

#### **U. Theme-Aware Development (Light/Dark Mode)**
To maintain a high-quality visual experience across both light and dark themes, we use a CSS-variable-driven approach integrated with UIkit.
- **Single Source of Truth**: All core colors and shadows are defined as CSS custom properties in `src/styles/main.scss` under `:root` (light) and `[data-theme='dark']` (dark).
- **UIkit Integration**: We map these CSS variables to UIkit's SCSS variables in `src/styles/uikit-variables.scss`. This ensures that all standard UIkit components automatically adapt to the current theme.
- **No Hardcoded Colors**: Avoid using hex codes or `rgb()` values directly in SCSS modules. Always use the provided UIkit variables (e.g., `$global-background`, `$card-border-radius`, `$global-color`, `$global-border`, `$global-primary-background`, etc.).
- **Modern Transparency**: Since Sass functions (like `rgba()` or `color.adjust`) do not support CSS variables, use the native CSS `color-mix()` function for transparency: `color-mix(in srgb, $global-primary-background, transparent 95%)`.
- **Theme-Aware Shadows**: Use global shadow variables like `var(--global-medium-shadow)` and `var(--global-large-shadow)` instead of hardcoded `rgba()` values. These variables are automatically tuned for visibility in both light and dark modes.
- **Theme-Aware Assets**: Placeholders and simple graphics (like `PlaceholderImage`) should be implemented as inline SVGs using `currentColor` for fills, allowing them to adapt dynamically via CSS.
- **Anti-Flicker Strategy**: The theme is applied via a `data-theme` attribute on the `<html>` element. A small script in `index.html` ensures the correct theme is applied before React hydrates to prevent a "flash of light mode."

#### **V. Performant List Virtualization**
For lists containing thousands of items (e.g., Chat), we use virtualization via `react-window` and `react-virtualized-auto-sizer` to maintain 60fps performance on mobile devices.
- **Conflict with Animations**: `react-window` v2 uses CSS `transform: translateY(...)` for row positioning. If the row component also uses Framer Motion animations (like `y` or `scale`), the list's positioning will be overwritten, causing items to stack at the top.
- **Solution**: Wrap the animated content in a static `div` that applies the `style` prop from the list. This ensures the transform is preserved while allowing internal elements to animate freely.
- **Dynamic Height Measurement**: For items with variable heights (e.g., Markdown messages), use a "shadow rendering" strategy. Render unmeasured items into a hidden off-screen container with the target width, measure their `offsetHeight`, and cache the results. Allow a small timeout (e.g., `150ms`) for complex layout and images to settle before measurement.
- **Initial Load Polish**: To prevent "clumping" (where elements appear stacked before their heights are calculated), hide the list behind a loading spinner until the first batch of items has been measured.
- **Context Awareness in Measurement**: The off-screen measurement root must be wrapped in necessary providers (e.g., `MemoryRouter`, `Provider`, `AuthContext`) if the components being measured contain navigation elements or rely on global state. This ensures accurate height calculation for complex items.

#### **W. Secure Real-Time Communication (SSE)**
We use Server-Sent Events (SSE) for real-time features like chat and typing indicators.
- **Header-Based Authentication**: Standard browser `EventSource` does not support custom headers. We use `@microsoft/fetch-event-source` to ensure the `Authorization: Bearer <token>` header is sent, maintaining security consistency with our REST API.
- **Reactive Connections**: SSE hooks must be reactive to the application's `isAuthenticated` state. When a user logs in, the connection should automatically refresh to utilize the new JWT token.
- **Robust Event Parsing**: Backend event streams often include heartbeats (empty data or "heartbeat" events) to keep the connection alive. The `onmessage` handler must explicitly filter these to avoid parsing errors.

### 4. Code Structure Standards
The project follows a modular, business-oriented directory structure to ensure scalability and maintainability.

#### **A. Shared & Global Resources**
Code that is reusable across the entire application remains in top-level directories under `src/`:
- **`src/components/uikit/`**: React wrappers for UIkit components. These should be pure, not depend on global state or context, and not involve any extra dependencies (e.g., `motion/react`).
- **`src/components/`**: Custom, universal React components that are not UIKit wrappers or are extending the look and feel of the UIkit wrappers.
    - **`SmartPagination.tsx`**: A standardized component for paginated lists that handles large page counts with ellipses (`...`).
- **`src/pages/`**: High-level layouts and error pages.
- **`src/hooks/`**: Global reusable hooks (e.g., `useUIKit`, `useMergeRefs`).
- **`src/store/`**: Global Redux store configuration and slices for domain-wide state.
- **`src/context/`**: Global application state (e.g., `AuthContext`).
- **`src/utils/`**, **`src/types/`**: Shared helper functions and TypeScript definitions.
- **`src/styles/`**: Global SASS variables and theme overrides.

#### **B. Business-Oriented Modules**
Logic specific to a particular business domain (e.g., Ads, Profile, Auth, Spaces) is encapsulated within its own directory under `src/` (e.g., `src/ads/`). Each module mirrors a simplified project structure:
- **`src/[module]/pages/`**: High-level layouts and form orchestration specific to the domain.
- **`src/[module]/components/`**: Components used exclusively within this module.
- **`src/[module]/services/`**: API clients and business logic for the module. Services should use `apiClient` and typed DTOs from `src/types/api.ts`.
- **`src/[module]/context/`**, **`src/[module]/hooks/`**: State and logic scoped to the module.
- **`src/[module]/utils/`**, **`src/[module]/types/`**: Helpers and types relevant only to this domain.

#### **C. Component Layout**
Every component (whether shared or module-specific) follows the same pattern:
- **`[ComponentName].tsx`**: The main React component.
- **`[ComponentName].stories.tsx`**: Storybook documentation and visual testing.
- **`[ComponentName].module.scss`**: Scoped CSS Module (if specific styling is required).

---

### 5. Summary of Preached Principles
1. **Never break the "Spring"**: UI changes should feel physical and smooth.
2. **Be helpful, not aggressive**: Use info messages and primary colors for guidance; save red danger colors for when the user has finished interacting with a field.
3. **Types Matter**: Use TypeScript interfaces for all component props and API payloads.
4. **Lifecycle Management**: Always clean up UIkit JS instances to prevent memory leaks in the SPA.
5. **Keep API Docs Current**: Supplement Controllers with example `.http` files and ensure they are updated alongside any controller modifications.
6. **Standardize Common Patterns**: Reuse standardized components (like `SmartPagination`) and logic (like `useUIKit` event patterns) across all modules to maintain a unified user experience.
7. **Avoid "Soulless" Tables**: Favor rich, card-based interfaces for complex business objects to provide a more engaging and interactive user experience.
8. **Meaningful Test Naming**: Test cases should describe the feature or behavior being validated. Avoid redundant or temporary-sounding phrases like "without crashing".
9. **Optimize for Bandwidth**: Use progressive image loading for galleries to ensure fast initial loads and reduced data usage.
10. **Robustness in Clones**: Always use stable identifiers (like `data-index`) when working with UI libraries that clone DOM elements for infinite scrolling or looping.
11. **JSDOM Compatibility**: Be aware of JSDOM limitations (e.g., lack of `DOMMatrix`, `IntersectionObserver`). Use appropriate polyfills in `src/vitest-setup.ts` to ensure UIkit's layout logic works correctly during testing.
12. **Prioritize Backend Messages**: Always display the localized `message` returned by the API (via `StatusResponseDto`).
13. **Global Continuity**: Long-running or critical business processes (like payments) must use global Redux state and be rendered at the root (`App.tsx`) to ensure they are not interrupted by user navigation.
14. **Strict IETF Tags**: Always use full IETF BCP 47 language tags (`en-US`, `pl-PL`) in both code and tests to ensure backend compatibility.
15. **Layering Excellence**: Use UIkit modal stacking (`{ stack: true }`) and localized button labels to maintain a professional, high-quality user interface in complex flows.
16. **Robust Portal Targeting**: When teleporting content into elements that appear dynamically (e.g., during animations), use `MutationObserver` to ensure the portal target is detected as soon as it mounts.
17. **Preserve Navigation Context**: When navigating to detail pages, pass the current location (including filters/pagination) in the router's `state`. Use this state to implement "smart" back buttons that return users to their exact previous context.
18. **Focused UI for Depth**: Use "focused" UI modes (like the Focused Navbar) to remove clutter and emphasize core content when users are deep in a specific task or viewing details.
19. **Think Twice Before Hardcoding**: Always check if a value (color, spacing, breakpoint) is already provided by a UIKit SASS variable before defining it manually to ensure design consistency and easy theme maintenance.
20. **Optimistic Continuity**: When updating user preferences (like language or currency), update the application state manually upon a successful API response to avoid unnecessary full-session re-initialization calls.
21. **Partial Updates via PATCH**: Prefer the `PATCH` verb for partial updates to resources (like user preferences or profile settings) to minimize payload size and avoid overwriting fields not intended for modification.
22. **Think Big**: Every feature, component, and line of code must be designed to meet the quality and scale expected of a platform competing with industry leaders like Amazon or eBay. We are not building a hobby project; we are building a global marketplace.
23. **Immersive Task Focus**: For complex, multi-step tasks (like category selection), use full-screen, immersive modals on mobile to maximize usable space and provide a "native" app feel.
24. **React-Compatible Modals**: Always set `container: false` for UIKit modals to keep them within the React root. This is strictly required to ensure React event delegation (for clicks, inputs, etc.) works correctly within the modal.
25. **Hybrid Traversal Speed**: Combine high-density Miller Columns (desktop) with focused Drill-Down stacks (mobile) to provide the fastest possible navigation through deep hierarchies.
26. **Favor Enhanced Images**: Never use raw `<img>` tags. Always use the project's enhanced `Image` component to ensure consistent loading placeholders and smooth transitions across the platform.
27. **Identity-First Trust**: Build trust through personalized identity. Use "User Spaces" and unique visual identifiers like `GeometricAvatar` to give every participant a professional and recognizable presence.
28. **Safe Passage**: Always warn users when they are about to leave the "safety" of the platform's ecosystem via external links. User trust is maintained by acknowledging the boundary between Gimlee and the wild web.
29. **Content Integrity**: Enforce strict sanitization and platform-consistent styling for all user-generated content (like descriptions) using the `Markdown` component. This prevents visual fragmentation, bandwidth hijacking, and potential tracking via external images.
30. **URL as Single Source of Truth**: For all listing, filtering, and sorting logic, treat the URL search parameters as the primary state. Synchronize local UI state to the URL to ensure deep linking, refresh persistence, and consistent browser history behavior.
31. **Pagination Reset**: Always reset the pagination index (e.g., `p=0`) when the user modifies any filter or sorting criteria. Failing to do so can lead to "empty states" if the user was on a high page number that doesn't exist for the new filtered results.
32. **Zero-Indexed Pagination**: The backend and `SmartPagination` component use **0-indexed** page numbers. Always ensure this is respected in search parameters and API calls to avoid off-by-one errors.
33. **Localized Currency Ornaments**: Do not hardcode the position of currency symbols in forms. Use `getCurrencyFormatInfo` to dynamically determine if the symbol should be a prefix (common for `en-US`) or a suffix (common for `pl-PL`) based on the current locale.
34. **Animation-Safe Overflows**: When using Framer Motion to animate height (e.g., in advanced filters or expandable cards), toggle the container to `overflow: visible` using `onAnimationComplete`. This prevents absolute-positioned children, such as autocomplete dropdowns, from being clipped.
35. **Branch vs. Leaf Selection**: In hierarchical selectors, distinguish between searching (where selecting a parent branch like "All in Electronics" is desired) and resource creation (where strict leaf-only selection is required for data integrity).
36. **PopLayout for Lists**: When filtering list items, use `AnimatePresence mode="popLayout"` on the container. This ensures that items being removed from the DOM don't cause the remaining items to "jump" instantly, allowing for a smooth layout transition.
37. **Themeability is First-Class**: Never hardcode colors. Every UI element must be designed to look perfect in both light and dark modes by using the centralized theme variables and UIkit variables.
38. **Color Mixing over Sass Adjust**: Prefer the CSS native `color-mix()` function for dynamic transparency and color adjustments. This maintains compatibility with CSS-variable-based themes where traditional Sass color functions fail.
39. **Physical Theme Transitions**: When switching themes, ensure smooth transitions for background and text colors to maintain a high-end, polished feel. Global transition rules in `main.scss` handle this for the base layers.
40. **Separate Positioning from Animation**: When using virtualization libraries that rely on CSS transforms for positioning, always wrap animated elements in a static container to prevent property conflicts.
41. **Shadow Rendering for Accuracy**: For dynamic content that needs precise measurement (like virtualization heights), use off-screen shadow rendering to capture true dimensions before items enter the visible DOM.
42. **Authenticated Streams**: Never pass sensitive tokens in URL query parameters for real-time streams. Use library wrappers that support standard `Authorization` headers to maintain security and consistency with the rest of the API.
43. **Initial Load Polish for Virtualization**: When using virtualized lists with dynamic heights, hide the initial list behind a loading spinner until the first batch of items is fully measured. This prevents "clumping" or jarring layout shifts during the initial render.
44. **Stick-to-Bottom Threshold**: For real-time chat interfaces, use a generous threshold (e.g., 50px) to detect if the user is at the bottom. This ensures the "stick-to-bottom" logic is robust against minor scroll variations and prevents unwanted jumps when the user is trying to read slightly older messages.
45. **Intelligent Unread Notifications**: In virtualized lists, manage unread counts by tracking which unread items have actually entered the viewport (e.g., via `onRowsRendered`). This provides a more natural UX where the notification count decreases as the user manually scrolls through new messages.
46. **Measurement Provider Context**: When performing off-screen "shadow rendering" for measurements, ensure the measurement container is wrapped in all necessary context providers (Redux, Router, Auth). This is critical for components that rely on these contexts (e.g., for `Link` or user-specific formatting) to render and measure correctly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gimlee-com)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gimlee-com)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
