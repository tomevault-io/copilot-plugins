## callers

> *   **Naming Convention (e.g., BEM, SUIT CSS, or similar):**

# CSS Coding Guidelines

*   **Naming Convention (e.g., BEM, SUIT CSS, or similar):**
    *   **Principle:** Use a consistent and predictable naming convention for classes to avoid conflicts, improve readability, and make CSS more maintainable.
    *   **Application:** Choose a methodology like BEM (Block, Element, Modifier) (e.g., `.block__element--modifier`) or another systematic approach. Be consistent throughout the project. Avoid overly generic names and ID selectors for styling.

*   **Modularity and Scalability (e.g., ITCSS, SMACSS):**
    *   **Principle:** Structure CSS in a way that is modular, scalable, and easy to manage, especially in larger projects.
    *   **Application:** Consider architectural approaches like ITCSS (Inverted Triangle CSS) or SMACSS (Scalable and Modular Architecture for CSS). Break CSS into logical components/modules. Use preprocessors (Sass, Less) to enhance organization with variables, mixins, and nesting (used judiciously).

*   **Readability and Formatting:**
    *   **Principle:** Write CSS that is easy to read and understand.
    *   **Application:** Be consistent with formatting (e.g., indentation, spacing, one property per line or multi-line). Logically order properties (e.g., positioning, box model, typography, visual). Comment non-obvious sections or complex selectors.

*   **Performance and Selectors:**
    *   **Principle:** Write efficient CSS that doesn't negatively impact page rendering performance.
    *   **Application:** Avoid overly complex or qualified selectors. Minimize the use of the universal selector (`*`) and descendant selectors where class-based selectors are more efficient. Avoid using `!important` unless absolutely necessary for overriding external styles or utility classes.

*   **Responsiveness and Units:**
    *   **Principle:** Design CSS to be responsive and adaptable to different screen sizes and devices.
    *   **Application:** Use relative units (e.g., `em`, `rem`, `%`, `vw`, `vh`) for flexible layouts and typography where appropriate. Employ media queries for responsive adjustments. Consider a mobile-first or desktop-first approach consistently.

*   **Separation of Concerns:**
    *   **Principle:** Keep CSS focused on presentation, separate from HTML structure and JavaScript behavior.
    *   **Application:** Avoid inline styles in HTML. JavaScript should primarily toggle classes to change styling, rather than directly manipulating CSS properties.

*   **Conciseness (Shorthand Properties):**
    *   **Principle:** Use shorthand properties where appropriate to reduce code verbosity, but ensure readability is maintained.
    *   **Application:** Use properties like `margin`, `padding`, `font`, `background` shorthand when setting multiple related values. For example, `margin: 10px 15px;` instead of `margin-top: 10px; margin-right: 15px; margin-bottom: 10px; margin-left: 15px;` (if all sides are symmetrical or paired).

*   **Accessibility:**
    *   **Principle:** Ensure CSS choices support accessibility.
    *   **Application:** Ensure sufficient color contrast. Avoid relying solely on color to convey information. Make sure focus states (`:focus`) are clearly visible. Use `prefers-reduced-motion` media query for users sensitive to animations. 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andreasalomone) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
