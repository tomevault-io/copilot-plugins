## the-shows-codex-cli-gpt5

> * **Mental Model First:** The file/folder structure must mirror the visual structure of the app. A developer should be able to find code for a UI element by navigating to the page/feature it appears on.

# **Always Rules — Structure, Naming, and Theming**

## 1. Core Philosophy

* **Mental Model First:** The file/folder structure must mirror the visual structure of the app. A developer should be able to find code for a UI element by navigating to the page/feature it appears on.
* **Separation of Concerns:** Break code into small, single-responsibility files (components, hooks, utils). Avoid monolithic files.
* **DRY but Clear:** Reuse code where possible, but never at the expense of clarity or the mental model.

---

## 2. Naming Conventions

* **Folders:**

  * `kebab-case` for route segments (e.g., `/app/show-details/`)
  * `snake_case` for all other directories (`features/`, `hooks/`, etc.)
* **Files:**

  * Components: `TitleCase.tsx` (`ShowCard.tsx`)
  * Pages: Next.js App Router format `page.tsx`
  * Hooks: `use_snake_case.ts` (`use_show_state.ts`)
  * Utilities/Services: `snake_case.ts`


### 2.1. Separation of Concerns

The main interest that I have is that we do not cross concerns in given pieces of software. The idea of separation is to allow one element to only be concerned with its own responsibility. Utilizing things like feature separation, what is traditionally thought of as componentizing, allows us to have a page that has very little actual responsibility except possible coordination responsibilities or data and action responsibilities, but that page itself should not be responsible, for example, for pulling the data. is a unit itself that should be externalized, likely into something like a hook. Functionality, any kind of functional Actions that need to occur should be broken into utils. The reason for this very discrete separation is for a vastly increased testing surface area.

* Ensure software components do not have overlapping responsibilities.
* Separation allows each element to focus solely on its own tasks.
* Use feature separation/componentization for clearer structure.
* Pages should primarily handle coordination or data/action responsibilities, not tasks like data retrieval.
* Externalize data retrieval, possibly into hooks.
* Break down functional actions into utility functions (utils).
* This discrete separation enhances the ability to test individual components effectively.

---

## 3. Project Structure

### 3.1. Top-Level Directories

* **`/app`** — Main app routes (Next.js App Router).
* **`/components`** — *Only* for truly generic, reusable components used across unrelated pages.
* **`/lib` or `/utils`** — For truly global utility functions or services.

### 3.2. Page-Level

Each route has its own folder inside `/app`, containing:

```
/app
└── /some-page
    ├── features/   // Page-specific components
    ├── hooks/      // Page-specific hooks
    ├── utils/      // Page-specific utilities
    └── page.tsx    // Minimal main page component
```

### 3.3. Feature-Level

Inside `features/`, each component gets its own `snake_case` folder containing:

```
/app
└── /some-page
    └── /features
        └── /my_feature
            ├── hooks/
            ├── MyFeature.tsx
            └── ...
```

---

## 4. Styling & Theming

### 4.1. Centralized Theme

* All colors, fonts, spacing, breakpoints, and other design tokens are defined in a **single theme file**.
* Never hardcode values in components — always reference the theme.

### 4.2. Colors

* Use a consistent palette (primary, secondary, error, warning, info, success).
* No raw hex/RGB in components — reference theme variables.

### 4.3. Typography

* Font families, sizes, and weights are defined in the theme.
* Apply typography styles via theme references.

### 4.4. Component Styling

* Reference theme variables in all styles.
* Avoid inline styles except for dynamic behavior.
* Use predictable patterns for hover, active, disabled, and focus states.

### 4.5. Spacing & Sizing

* Use a predefined spacing scale (e.g., 4px or 8px units).
* No arbitrary pixel values — reference spacing constants.

### 4.6. Responsive Design

* Use a centralized breakpoint system in the theme.
* Avoid one-off media queries — use theme breakpoints.

### 4.7. Accessibility

* Ensure color contrast meets WCAG guidelines.
* Maintain semantic HTML and ARIA attributes.

### 4.8. Theming Consistency

* Regularly review the theme for consistency.
* Document any deviations from the theme.

---

## 5. Backend vs. Frontend Logic

* **Default to Frontend:** Use client components when possible (especially for real-time API SDKs).
* **Backend Only When Needed:** Move logic server-side for security, sensitive data, or heavy processing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bladnman)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bladnman)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
