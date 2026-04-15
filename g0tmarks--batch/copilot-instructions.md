## batch

> Standards for implementing accessible design patterns based on learnings from mobile navbar implementation.

# Accessibility Standards

Standards for implementing accessible design patterns based on learnings from mobile navbar implementation.

<rule>
name: accessibility_standards
description: Standards for accessibility implementation
filters:
  - type: file_extension
    pattern: "\\.(html|css|js)$"
  - type: content
    pattern: "(button|nav|menu|form|input|label)"

actions:
  - type: suggest
    message: |
      ## Accessibility Standards

      ### 1. ARIA Labels and Roles
      - Always include `aria-label` for interactive elements without visible text
      - Use semantic HTML elements (`<nav>`, `<button>`, `<main>`)
      - Provide descriptive labels for screen readers

      ### 2. Keyboard Navigation
      - Ensure all interactive elements are keyboard accessible
      - Implement proper focus management for dynamic content
      - Maintain logical tab order throughout the interface

      ### 3. Focus Management
      - Handle focus when opening/closing modals or dropdowns
      - Provide visible focus indicators
      - Don't remove focus from the page without user action

      ### 4. Screen Reader Support
      - Use descriptive text for navigation items
      - Provide context for dynamic content changes
      - Ensure error messages are announced to screen readers

      ### 5. Color and Contrast
      - Maintain sufficient color contrast (WCAG AA standards)
      - Don't rely solely on color to convey information
      - Provide alternative indicators for color-coded information

examples:
  - input: |
      <!-- Bad: Missing ARIA label -->
      <button class="hamburger-menu">
        <span></span>
        <span></span>
        <span></span>
      </button>

      <!-- Good: Proper ARIA label -->
      <button class="hamburger-menu" aria-label="Toggle navigation menu">
        <span></span>
        <span></span>
        <span></span>
      </button>
    output: "Include descriptive ARIA labels for interactive elements"

  - input: |
      /* Bad: No focus indicator */
      .nav-item:focus { outline: none; }

      /* Good: Visible focus indicator */
      .nav-item:focus { 
        outline: 2px solid #007bff;
        outline-offset: 2px;
      }
    output: "Provide visible focus indicators for keyboard navigation"

  - input: |
      // Bad: No focus management
      function openMenu() {
        menu.classList.add('active');
      }

      // Good: Proper focus management
      function openMenu() {
        menu.classList.add('active');
        menu.querySelector('a').focus();
      }
    output: "Implement proper focus management for dynamic content"

metadata:
  priority: high
  version: 1.0
  tags: ["accessibility", "aria", "keyboard", "screen-reader"]
</rule>
description:
globs:
alwaysApply: false
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/g0tMarks)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/g0tMarks)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
