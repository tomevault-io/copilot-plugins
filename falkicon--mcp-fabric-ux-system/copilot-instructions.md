## mcp-fabric-ux-system

> Core accessibility (a11y) standards based on WCAG 2.2 AA for Fabric UX.

# Core Accessibility Specification

**Status:** Draft (as per source)

**Goal:** Define the fundamental accessibility requirements applicable to all Fabric UX components, ensuring alignment with established web standards.

**Compliance Target:** All components **must** comply with [**WCAG 2.2 Level AA**](mdc:Projects - MS/Projects - MS/Projects - MS/Projects - MS/https:/www.w3.org/TR/WCAG22) guidelines.

## 1. Alignment with WCAG Principles

Our approach is grounded in the four core principles of WCAG:

*   **Perceivable:** Information and UI components must be presentable to users in ways they can perceive (e.g., text alternatives, sufficient contrast, adaptability).
*   **Operable:** UI components and navigation must be operable via keyboard and other input methods, without requiring interactions the user cannot perform (e.g., no functionality solely relies on specific gestures like dragging or hover, sufficient time, navigable).
*   **Understandable:** Information and the operation of the UI must be understandable (e.g., readable text, predictable behavior, input assistance).
*   **Robust:** Content must be robust enough that it can be interpreted reliably by a wide variety of user agents, including assistive technologies (e.g., correct use of semantic HTML and ARIA roles, states, properties).

## 2. General Requirements

These apply to all components unless specifically exempted with justification.

*   **Keyboard Accessibility:** All interactive features must be fully operable via keyboard alone, following standard conventions (Tab, Shift+Tab, Arrows, Space, Enter, Esc, Home, End, PageUp, PageDown where applicable).
*   **Focus Management:** Logical focus order (**WCAG 2.4.3**) must be maintained. Focus must be visibly indicated (**WCAG 2.4.7**, **WCAG 2.4.11**) and managed predictably during interactions, especially with dynamic content, modals, or context menus.
*   **Semantic Structure & ARIA:** Use semantic HTML elements where possible. Apply appropriate ARIA roles (e.g., `grid`, `row`, `columnheader`, `gridcell`) and state/property attributes (`aria-sort`, `aria-selected`, `aria-expanded`, etc.) correctly and keep them synchronized with the component's state.
    *   **Implementation Guidance:** Leverage the standard [`ElementInternals`](mdc:Projects - MS/Projects - MS/Projects - MS/Projects - MS/https:/developer.mozilla.org/en-US/docs/Web/API/ElementInternals) API for setting ARIA roles and properties programmatically (e.g., `this.elementInternals.role = 'gridcell'`, `this.elementInternals.ariaSelected = 'true'`). This aligns with modern Web Component best practices.
*   **Accessible Names & Descriptions:** All interactive elements, regions, and the grid itself must have accessible names (e.g., via `aria-label`, `aria-labelledby`). Use `aria-describedby` for additional instructions or context where needed.
*   **Contrast:** Meet **WCAG 1.4.3 (AA)** for text and **WCAG 1.4.11 (AA)** for non-text elements (UI components, graphical objects, state indicators, focus outlines) in all themes and interaction states.
*   **Target Size:** All interactive targets **must** meet **[WCAG 2.5.8 (Target Size Minimum)](mdc:Projects - MS/Projects - MS/Projects - MS/Projects - MS/https:/www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html)** (e.g., 24x24 CSS pixels or sufficient spacing).
*   **Non-Dragging Alternatives:** Functionality achievable via path-based gestures (like dragging) **must** have a single-pointer activation alternative (**[WCAG 2.5.1](mdc:Projects - MS/Projects - MS/Projects - MS/Projects - MS/https:/www.w3.org/WAI/WCAG22/Understanding/pointer-gestures.html)**) and a keyboard alternative (**WCAG 2.1.1**).
*   **Text Alternatives:** All non-text content (images, icons conveying information, charts) **must** have appropriate text alternatives (**[WCAG 1.1.1](mdc:Projects - MS/Projects - MS/Projects - MS/Projects - MS/https:/www.w3.org/WAI/WCAG22/Understanding/non-text-content.html)**).
*   **Labels and Instructions:** Provide clear labels or instructions when user input is required (**[WCAG 3.3.2](mdc:Projects - MS/Projects - MS/Projects - MS/Projects - MS/https:/www.w3.org/WAI/WCAG22/Understanding/labels-or-instructions.html)**).
*   **Error Handling:** Identify and describe input errors clearly (**[WCAG 3.3.1](mdc:Projects - MS/Projects - MS/Projects - MS/Projects - MS/https:/www.w3.org/WAI/WCAG22/Understanding/error-identification.html)**) and suggest corrections where possible (**[WCAG 3.3.3](mdc:Projects - MS/Projects - MS/Projects - MS/Projects - MS/https:/www.w3.org/WAI/WCAG22/Understanding/error-suggestion.html)**).
*   **Consistency:** Maintain consistent navigation (**[WCAG 3.2.3](mdc:Projects - MS/Projects - MS/Projects - MS/Projects - MS/https:/www.w3.org/WAI/WCAG22/Understanding/consistent-navigation.html)**) and identification (**[WCAG 3.2.4](mdc:Projects - MS/Projects - MS/Projects - MS/Projects - MS/https:/www.w3.org/WAI/WCAG22/Understanding/consistent-identification.html)**) across component sets.

## 3. Component-Specific Considerations

Individual component specifications **must** reference this core document and **must** detail any specific accessibility requirements, ARIA patterns, or high-risk areas relevant to that component's unique functionality (e.g., specific keyboard interactions for a grid, ARIA roles for a tree view).

## 4. Fabric UX Accessibility Resources

In addition to the general standards above, refer to the specific Fabric UX accessibility guidance:

*   **[Developer Guidelines](mdc:Projects - MS/Projects - MS/Projects - MS/Projects - MS/https:/ambitious-cliff-0c8148010.2.azurestaticapps.net/?path=/docs/guides-accessibility-developer-guidelines--docs):** Practical implementation guidance for developers.
*   **[Common Pitfalls and Solutions](mdc:Projects - MS/Projects - MS/Projects - MS/Projects - MS/https:/ambitious-cliff-0c8148010.2.azurestaticapps.net/?path=/docs/guides-accessibility-common-pitfalls-and-solutions--docs):** Learn from common mistakes.
*   **[Design Principles](mdc:Projects - MS/Projects - MS/Projects - MS/Projects - MS/https:/ambitious-cliff-0c8148010.2.azurestaticapps.net/?path=/docs/guides-accessibility-design-principles--docs):** Understand the underlying design philosophy.
*   **[WCAG 2.2 Guidance](mdc:Projects - MS/Projects - MS/Projects - MS/Projects - MS/https:/ambitious-cliff-0c8148010.2.azurestaticapps.net/?path=/docs/guides-accessibility-wcag2-2--docs):** Fabric-specific perspectives on WCAG 2.2.
*   **[Accessibility Overview](mdc:Projects - MS/Projects - MS/Projects - MS/Projects - MS/https:/ambitious-cliff-0c8148010.2.azurestaticapps.net/?path=/docs/guides-accessibility-overview--docs):** General introduction to accessibility within Fabric UX.

## 5. Testing Requirements

All components must undergo rigorous accessibility testing:

*   **Manual Keyboard Testing:** Verify all interactions using only the keyboard.
*   **Screen Reader Testing:** Test thoroughly with supported screen readers (e.g., NVDA, JAWS, VoiceOver) covering navigation, state announcements, and interaction flows.
*   **Magnification Testing:** Verify usability with screen magnification tools (e.g., ZoomText, Windows Magnifier).
*   **Contrast Checking:** Use tools (e.g., browser dev tools, Colour Contrast Analyser) to verify color contrast compliance in all themes and states.
*   **Automated Testing:** Implement automated accessibility checks using tools like axe-core during development and CI/CD.
*   **Code Review:** Include accessibility checks as part of the standard code review process.
*   **Dedicated Reviews:** Conduct specific accessibility reviews at key project milestones. 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Falkicon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Falkicon)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
