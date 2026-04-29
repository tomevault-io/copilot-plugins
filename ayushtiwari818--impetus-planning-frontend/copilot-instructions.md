## impetus-planning-frontend

> :::tip[Accessibility First]

# Accessibility Requirements Rules

:::tip[Accessibility First]
Ensure all components and interfaces are accessible to users with disabilities
:::

## 1. GENDS Accessibility Features

:::info[GENDS Foundation]

- Use GENDS components which have built-in accessibility features
- Trust GENDS accessibility implementations
- Test GENDS component combinations for accessibility
- Report accessibility issues with GENDS components to the design system team
  :::

:::code-group

```jsx [GENDS Accessibility]
// ✅ GENDS components with built-in accessibility
import { Button, Input, Modal, DataTable } from "gends";

// These components include:
// - Proper ARIA labels
// - Keyboard navigation
// - Screen reader support
// - Focus management
```

:::

## 2. Semantic HTML Structure

:::warning[Semantic Requirements]

- Use semantic HTML elements with GENDS components
- Maintain proper heading hierarchy (h1, h2, h3...)
- Use landmark elements (nav, main, aside, footer)
- Ensure logical tab order and focus flow
  :::

:::code-group

```jsx [Semantic Structure]
// ✅ Semantic structure with GENDS
import { Text, Button, TitleBar } from "gends";

const PageLayout = () => (
  <main>
    <TitleBar title="Page Title" /> {/* Proper heading structure */}
    <section>
      <Text as="h2">Section Title</Text>
      <Button>Accessible Action</Button>
    </section>
  </main>
);
```

:::

## 3. Keyboard Navigation

:::danger[Keyboard Requirements]

- Ensure all interactive elements are keyboard accessible
- Test tab navigation through all components
- Implement proper focus indicators
- Use GENDS components that handle keyboard events
  :::

:::code-group

```jsx [Keyboard Accessibility]
// ✅ Keyboard accessible custom component
const AccessibleDropdown = ({ options, onSelect }) => {
  const handleKeyDown = (event) => {
    if (event.key === "Enter" || event.key === " ") {
      // Handle activation
    }
    if (event.key === "Escape") {
      // Close dropdown
    }
  };

  return (
    <div
      role="combobox"
      tabIndex={0}
      onKeyDown={handleKeyDown}
      aria-expanded={isOpen}
    >
      {/* Dropdown content */}
    </div>
  );
};
```

:::

## 4. ARIA Labels and Descriptions

:::tip[ARIA Guidelines]

- Use descriptive ARIA labels for complex interactions
- Provide ARIA descriptions for detailed information
- Use GENDS component ARIA props when available
- Ensure screen reader announcements are meaningful
  :::

:::code-group

```jsx [ARIA Usage]
// ✅ Proper ARIA usage with GENDS
import { Button, Input, Modal } from "gends";

const AccessibleForm = () => (
  <form>
    <Input label="Email Address" aria-describedby="email-help" required />
    <div id="email-help">We'll never share your email address</div>

    <Button aria-label="Submit form to create account" type="submit">
      Create Account
    </Button>
  </form>
);
```

:::

## 5. Color and Contrast

:::warning[Visual Accessibility]

- Ensure WCAG AA contrast ratios (4.5:1 for normal text)
- Don't rely solely on color to convey information
- Use GENDS color tokens that meet accessibility standards
- Test with color blindness simulators
  :::

:::code-group

```jsx [Color Accessibility]
// ✅ Accessible status indication
import { Badge, IcSuccess, IcError, Text } from "gends";

const StatusIndicator = ({ status, message }) => (
  <div>
    {status === "success" ? (
      <Badge variant="success" icon={<IcSuccess />}>
        <Text>Success: {message}</Text>
      </Badge>
    ) : (
      <Badge variant="error" icon={<IcError />}>
        <Text>Error: {message}</Text>
      </Badge>
    )}
  </div>
);
```

:::

## 6. Screen Reader Support

:::info[Screen Reader Compatibility]

- Test with screen readers (NVDA, JAWS, VoiceOver)
- Ensure proper reading order and context
- Use GENDS components that announce changes
- Implement live regions for dynamic content
  :::

:::code-group

```jsx [Screen Reader Support]
// ✅ Screen reader announcements
import { Text, notify } from "gends";

const DataLoader = ({ isLoading, data }) => {
  useEffect(() => {
    if (!isLoading && data) {
      notify.success("Data loaded successfully", {
        "aria-live": "polite",
      });
    }
  }, [isLoading, data]);

  return (
    <div>
      {isLoading && <Text aria-live="polite">Loading data...</Text>}
      {data && <DataDisplay data={data} />}
    </div>
  );
};
```

:::

## 7. Focus Management

:::tip[Focus Control]

- Manage focus in modals and dynamic content
- Return focus to trigger element when closing modals
- Use GENDS Modal component focus management
- Implement visible focus indicators
  :::

:::code-group

```jsx [Focus Management]
// ✅ Focus management with GENDS Modal
import { Modal, Button } from "gends";

const AccessibleModal = ({ isOpen, onClose, triggerRef }) => {
  return (
    <Modal
      btnTitle="Open Modal"
      open={isOpen}
      setOpen={onClose}
      customWidth=""
      description="Leaving this page will delete all unsaved changes."
      headerTitle="Leave page with unsaved changes?"
      onBack={function Xs() {}}
      primaryButtonProps={{
        onClick: function Xs() {},
        title: "Confirm",
      }}
      secondaryButtonProps={{
        onClick: function Xs() {},
        title: "Cancel",
      }}
      secondaryClose
      size="m"
    />
  );
};
```

:::

## 8. Form Accessibility

:::warning[Form Requirements]

- Associate labels with form controls
- Provide clear error messages
- Use GENDS form components with built-in validation
- Group related form fields appropriately
  :::

:::code-group

```jsx [Accessible Forms]
// ✅ Accessible form with GENDS
import { Input, Checkbox, Button, Text } from "gends";

const AccessibleForm = () => {
  const [errors, setErrors] = useState({});

  return (
    <form>
      <fieldset>
        <legend>Personal Information</legend>

        <Input
          label="First Name"
          required
          error={errors.firstName}
          aria-invalid={!!errors.firstName}
        />

        <Checkbox
          label="I agree to terms and conditions"
          required
          aria-describedby="terms-help"
        />
        <Text id="terms-help" as="p" variant="desktop-display-s">
          Please read our terms before proceeding
        </Text>
      </fieldset>

      <Button type="submit">Submit Form</Button>
    </form>
  );
};
```

:::

## 9. Mobile Accessibility

:::info[Mobile Support]

- Ensure touch targets are at least 44px
- Test with mobile screen readers
- Use GENDS responsive components
- Implement proper zoom support
  :::

## 10. Testing Requirements

:::danger[Testing Mandatory]

- Test with keyboard-only navigation
- Test with screen readers
- Use automated accessibility testing tools
- Verify WCAG 2.1 AA compliance
- Test with users who have disabilities
  :::

:::code-group

```javascript [Accessibility Testing]
// ✅ Automated accessibility testing
import { axe, toHaveNoViolations } from "jest-axe";

expect.extend(toHaveNoViolations);

test("component should be accessible", async () => {
  const { container } = render(<MyComponent />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

:::

## 11. Documentation Requirements

:::tip[Documentation Standards]

- Document accessibility features of custom components
- Include accessibility examples in component README
- Document keyboard shortcuts and interactions
- Provide accessibility testing guidelines
  :::

## 12. Common Accessibility Patterns

:::info[Best Practices]

- Use GENDS DataTable for accessible data presentation
- Use GENDS navigation components for accessible routing
- Implement proper loading states with GENDS
- Use GENDS notification system for accessible feedback
  :::

:::code-group

```jsx [Accessible Patterns]
// ✅ Accessible data table
import { DataTable, useDataTable } from "gends";

const AccessibleTable = ({ data, columns }) => {
  const tableProps = useDataTable({
    data,
    columns,
    accessibleOptions: {
      caption: "List of purchase orders",
      summary: "Table showing order details with status and actions",
    },
  });

  return <DataTable {...tableProps} />;
};
```

:::

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ayushtiwari818) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
