## medapp

> 1. NEVER, ever, ever, ever, clean the DB without conscent. In fact, never run any code that touches the database unless explicitely told.

# General rules

1. NEVER, ever, ever, ever, clean the DB without conscent. In fact, never run any code that touches the database unless explicitely told.
2. When interacting with the API in a client-side component use the hooks defined in `~/components/provider.tsx`. Always prefer Remix loaders to grab initial data and actions to submit forms.
3. Always define event handlers (like for button clicks) using something like:
  ```js
  const handleClick = () => {};

  <Component onClick={handleClick} />
  ```

  Always memoize functions to avoid them being re-created.
4. Don't use `as any` to get out of a TS compilation error. Always have proper types.
5. When defining feathers hooks, these must go in their own file. If the hook is meant to work in multiple services, then create it in `apps/api/src/hooks`. If the hook is server-specific, then `apps/api/src/services/:service-name/hooks`
6. When running tests, these must use the `athelas_api_test` database instead of the actual DB used by the app (`athelas_api`) (this is behind a `NODE_ENV=test` flag)
7. When writing feathers services, **ALWAYS** prefer using feathers service calls instead of raw SQL queries, unless told otherwise.
8. When writing API code, make sure to add or update tests for the related functionality

# Code Style

1. When using conditional rendering of components, prefer explicit true/false code over ternary operators, like:

```jsx
{isDesktop && (
  <SomeDesktopComponent />
)}

{!isDesktop && (
  <SomeMobileComponent />
)}
```

# Medical Forms Styling Rules

Adhere to these styling rules when creating or modifying medical forms in the `apps/ui/app/components/forms/` directory.

## Core Principles

1.  **Shared Styles**: ALWAYS import styling components from `~/components/forms/styles`. DO NOT define local styled components for common form elements.
2.  **Consistency**: All forms must share the same visual language: borderless inputs, specific label widths, and card-based layouts.
3.  **Read-Only Support**: Every form component MUST accept a `readOnly` prop and handle it by disabling inputs or rendering alternative text.
4.  **Auto-Saving (onChange)**: Forms should NOT have an `onSubmit` callback or a submit button. Instead, they must accept an `onChange` prop. This callback should be invoked whenever any field value changes by triggering the `onChange` in a `useEffect` watching `form.values`. General form debouncing is discouraged to keep the UI responsive; only use `useDebouncedValue` for specific fields that perform heavy database queries (e.g., ICD-10 or medication searches).
5. **Mobile first**: Every component must work on mobile and adapt to larger screens. There's a `media` object exported from `~/stitches` to help with that.
6. **Re-use**: Don't reinvent the wheel, when possible rely on `lodash`, `mantine`, etc. If you have to roll your own components, use PandaCSS for styling.
7. **Icons**: If icons are needed, always use `@phosphor-icons/react` and never attempt to install another icon library.
8. **Encounter Tree**: In the `encounter-tree.tsx`, clicking a date should do nothing. Navigation should only happen when clicking specific form items within an encounter.
9. **Linting**: React hooks can't be called conditionally. The early exit pattern can't be used earlier than hook calls.

After creating a new form, add it to `components/forms/encounter-form.tsx` and `components/new-encounter-sidebar.tsx` and make sure it's usable.

## Styling Components

Use these exported components from `styles.tsx`:

-   `FormContainer`: The main wrapper for the form (flex column, 2rem gap).
-   `FormCard`: A white container with a light gray border and rounded corners.
-   `FieldRow`: A flex row for each field (1rem padding, bottom border).
-   `Label`: A text label (25% width, right-aligned, gray-6 color).
-   `StyledTextInput`, `StyledTextarea`, `StyledDateInput`: Borderless Mantine inputs with specific line heights.
-   `StyledTitle`: A blue-4 title with 400 font weight. Don't add the `order` prop, this usually sends titles to the bottom of the form and gives me extra work to fix them.
-   `FormHeader`, `ItemHeader`: Flex containers for form/item titles and action buttons.

Add new components as needed.

## Implementation Example

```tsx
import { useEffect } from 'react';
import { useForm } from '@mantine/form';
import { FormContainer, FormCard, FieldRow, Label, StyledTextarea, StyledTitle } from '~/components/forms/styles';

export function MyNewForm({ initialData, onChange, readOnly }: Props) {
  const form = useForm({
    initialValues: parseInitialValues(initialData),
  });

  useEffect(() => {
    if (!readOnly) {
      onChange(transformToLegacyFormat(form.values));
    }
  }, [form.values, onChange, readOnly]);

  return (
    <FormContainer>
      <StyledTitle order={1}>Form Title</StyledTitle>
      <FormCard>
        <FieldRow>
          <Label>Field Label:</Label>
          <StyledTextarea
            readOnly={readOnly}
            autosize
            {...form.getInputProps('field')}
          />
        </FieldRow>
      </FormCard>
    </FormContainer>
  );
}
```

## Data Format

Most forms follow a specific legacy structure where values are indexed strings (e.g., `field_0`, `field_1`) and include a `count` field. Always verify the requested JSON structure before implementing the `onChange` and `parseInitialValues` logic.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/silvestreh)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/silvestreh)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
