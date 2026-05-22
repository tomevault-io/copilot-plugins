## slds-rules

> 1. [Foundational Principles for Salesforce UI Development](#foundational-principles-for-salesforce-ui-development)

## Table of Contents
1. [Foundational Principles for Salesforce UI Development](#foundational-principles-for-salesforce-ui-development)
2. [UX Principles](#ux-principles)
3. [Visual Design](#visual-design)
4. [Component Usage](#component-usage)
5. [Interaction Patterns](#interaction-patterns)
6. [Accessibility](#accessibility)
7. [Form Design](#form-design)
8. [Feedback and Validation](#feedback-and-validation)
9. [Layout and Responsiveness](#layout-and-responsiveness)
10. [Consistency and Reusability](#consistency-and-reusability)
11. [Theming and Customization](#theming-and-customization)
12. [Testing and QA](#testing-and-qa)
13. [Documentation and Onboarding](#documentation-and-onboarding)
14. [Feedback and Updates](#feedback-and-updates)

---

# Foundational Principles for Salesforce UI Development

Whenever developing Salesforce experiences—especially using LWC and SLDS—follow these foundational guidelines to ensure scalable, maintainable, and consistent UI code:

## Making Customizations: Decision Hierarchy

When writing UI code, always prioritize the following order for styling and layout decisions:

1. **Lightning Base Component/Variant**
   - Before writing any HTML, CSS, or JavaScript, determine if an existing Lightning Base Component (e.g., `lightning-button`, `lightning-card`) can achieve the desired appearance or behavior. For the full set of components and documentation, see the [Lightning Base Components Index](mdc:.cursor/rules/lightning-base-components/index.mdc).
   - Use a base component's out-of-the-box configuration options before adding any classes or writing custom CSS.

2. **Utility Class**
   - If the base component alone is insufficient, check if an SLDS utility class is available. Utility classes are especially important for composing "molecular" components—those made up of several smaller components that form part of a page or an entire page.
   - For the full set of utility classes, see [utilities](mdc:.cursor/rules/utilities/index.mdc).

3. **Custom CSS with Styling Token**
   - If neither a base component nor a utility class suffices, use the correct global styling hook (e.g., `--slds-g-color-surface-1`, `--slds-g-color-accent-1`) and assign it to the CSS property with a CSS variable. See [global-styling-hooks-color-usage.mdc](mdc:.cursor/rules/global-styling-hooks/global-styling-hooks-color-usage.mdc) for guidance on color usage, and only use real global styling hooks as specified in [slds-global-styling-hooks.mdc](mdc:.cursor/rules/global-styling-hooks/slds-global-styling-hooks.mdc).
   - Always use styling hooks in the correct semantic manner (never use a styling hook for an unintended purpose, such as `width: var(--slds-g-radius-border-circle)`).

> **Avoid all hard-coded values** except as a last resort (e.g., `height: 100%` if no utility class or styling hook exists).

---

## UX Principles
- The design should be easy to learn, understand, and navigate without explicit instruction.
- Respect progressive disclosure: don't overwhelm users.
- Use space, layouts, and data-density strategically to drive clarity.
- Use consistent interactions across flows, apps, and pages.
- Always test with assistive technologies, Salesforce customers, and internal users.

---

## Visual Design
> See [Foundational Principles for Salesforce UI Development](mdc:#foundational-principles-for-salesforce-ui-development) for the required order of styling and layout decisions.
- Follow SLDS spacing scale for margins and padding.
- Maintain brand-appropriate themes using semantic styling hooks.
- Establish information hierarchy using SLDS typography heading levels and card components.
- All text and interactive elements must meet WCAG 2.1 color contrast standards.
- All content must be in card containers.
- Text, icons, and other content elements must use appropriate spacing within cards. If no specific spacing is specified, use `slds-p-around_small`.
- Only use SLDS icons for any iconography requirements. See [Icons Index](mdc:.cursor/rules/icons/index.mdc) for the authoritative list of available icons and usage guidelines.
- Always use `<lightning-icon>` for SLDS icons.
- Standard Object icons should be used adjacent to record titles.
- Utility icons should be used for most other icon needs.
- **Never use Action icons.** (Action icons are reserved for platform use only and should not be used in custom apps or components.)

**Example: Card Container with Proper Spacing**
```html
<lightning-card class="slds-p-around_small">
  <div class="slds-text-heading_medium">Title</div>
</lightning-card>
```

**Example: Using a Styling Hook**
```css
.my-card {
  --slds-c-card-color-background: var(--slds-g-color-surface-1);
}
```

---

## Component Usage
> As outlined in the [Foundational Principles](#foundational-principles-for-salesforce-ui-development), always start with Lightning Base Components, then Utility Classes, then Styling Hooks.
- Use Lightning Base Components (e.g., `<lightning-card>`, `<lightning-datatable>`, `<lightning-button>`) as your first choice. Only write new LWC markup if a Lightning Base Component does not exist for the use case.
- If a Lightning Web Component needs customization, use styling hooks and slots.
- Only override SLDS CSS with styling hooks, nothing else.
- Follow structural markup patterns for each component (e.g., `slds-modal`, `slds-button`, `slds-form-element`).
- Apply utility classes rather than custom CSS to reduce override complexity.
- Use only documented SLDS variants for buttons, forms, inputs, etc.
- Always run the SLDS Linter tool and address issues it reports.
- Compose larger components by using a combination of smaller Lightning Base Components.
- Break down large UIs into smaller, reusable LWC components. Each component should have its own folder using kebab-case, and the main class should use PascalCase.
- All components that display data must handle empty, loading, and error states gracefully, using SLDS illustrations or messages where appropriate.

**Example: Accessible Button**
```html
<lightning-button label="Save" variant="brand" onclick={handleSave} title="Save record"></lightning-button>
```

---

## Interaction Patterns
- Use modal dialogs (`slds-modal`) for focused, interruptive tasks.
- Use toast notifications (`slds-notify_toast`) for transient system feedback.
- Apply spinners and skeletons to communicate loading states.
- Interactive elements must have default, hover, focus, active, pressed, and visited states.
- Always provide a visible focus state for keyboard users.
- Use appropriate `aria-` attributes and roles to support interactivity.
- Avoid nesting interactive elements inside links or buttons or other non-card containers.

---

## Accessibility
- Every component must be accessible per WCAG 2.1 guidelines.
- Use SLDS color utility classes and tokens for all color styling.
- Associate labels with inputs via `for` and `id` or ARIA attributes.
- Modals must use `role="dialog"` and trap focus.
- Toasts must be `aria-live` regions to announce updates.
- Maintain appropriate color contrast and keyboard navigability.
- Maintain an appropriate tab order for keyboard users to efficiently navigate through the UI.
- All tables must include `role="grid"` and use `scope="col"` for column headers. Add ARIA attributes (e.g., `aria-label`, `aria-labelledby`, `aria-describedby`) to all interactive elements and tables for screen reader support.

---

## Form Design
- Use Lightning Base Components (e.g., `<lightning-input>`, `<lightning-combobox>`, `<lightning-radio-group>`, `<lightning-textarea>`) for all form elements. Do not use raw `<input>`, `<select>`, or `<textarea>` unless a Lightning Base Component does not exist for the use case.
- Use `slds-form-element` for grouping input + label + help text.
- Label every input field with `<label>` or `aria-label`.
- Show validation errors inline with helper text and an error icon.
- Group related inputs using `fieldset` + `legend` when needed.
- Use correct input types (e.g., `type="email"`) and patterns.
- Utilize semantic styling hook colors for error and warning states.
- Use edit and read-only modes for data.

---

## Feedback and Validation
- Use error and warning classes to indicate form validation issues.
- Add `role="alert"` for inline error messages.
- Use `aria-describedby` to associate help or error text.
- Provide confirmation and success feedback with toasts or banners.

---

## Layout and Responsiveness
- Ensure all layouts are responsive using SLDS grid and utility classes. Test components at multiple breakpoints (`slds-size_1-of-1`, `slds-medium-size_1-of-2`, etc.) and avoid fixed pixel widths.
- Use breakpoints to resize and reflow with `slds-grid`.
- All designs must be responsive and adaptable to zooming.
- Leverage responsive utility classes (`slds-medium-size_...`, etc.).
- Avoid fixed pixel widths; use percentages or percentage-related units.

---

## Consistency and Reusability
- Reuse SLDS components across products and clouds.
- Follow documented slot and modifier patterns for extending components.
- Align styles with styling hooks.
- Minimize overrides and avoid one-off custom styles.

---

## Theming and Customization
> See [Foundational Principles for Salesforce UI Development](#foundational-principles-for-salesforce-ui-development) for the correct order of customization and overrides.
- Theme only via styling hooks and CSS custom properties, as described in the Foundational Principles.
- Use the Theming and Branding feature in Setup to create consistent visual identity.
- Never use `!important` or manual overrides of SLDS styles.
- Verify customizations meet SLDS accessibility requirements.
- Do not use inline style attributes in any HTML or JSX. All layout, spacing, and visual styling must be achieved using SLDS utility classes or component CSS files.

---

## Testing and QA
- Test with screen readers like NVDA, JAWS, and VoiceOver.
- Confirm keyboard-only navigation works across all components.
- Test components across breakpoints defined in SLDS.
- Run automated a11y tools (e.g., axe-core, Lighthouse).
- Validate CSS classes match SLDS docs (no rogue classes).
- Use Storybook or similar to validate visual and interactive consistency.
- Every LWC must include a Jest test file named `[component].test.js` in a `__tests__` subfolder. All exported classes and public methods must have JSDoc comments describing their purpose and usage.
- All public methods and UI states must be covered by Jest unit tests. Aim for at least 85% code coverage.
- Every component must include a usage example in its README or JSDoc, showing how to use the component in a parent LWC or Lightning App Builder.

---

## Documentation and Onboarding
- Link to component developer docs in component documentation.
- Document when, how, and why components diverge from Lightning Base Components.
- Include accessibility notes with every new pattern.

---

## Feedback and Updates
- For questions, feedback, or to propose updates to these rules, please contact the design system team at design.system@salesforce.com or submit a pull request to the rules repository.
- Stay updated with the latest Lightning Design System releases and Salesforce accessibility standards. Always refer to https://lightningdesignsystem.com and https://developer.salesforce.com/docs/component-library/overview/components for source-of-truth documentation. 
---

---
> Source: [austin-guevara/lwc-slds-lbc-starter](https://github.com/austin-guevara/lwc-slds-lbc-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
