## 07-accessibility-rules

> - **Mode**: Always On

# Accessibility Rules - iOS VoiceOver & WCAG 2.1

## Activation

- **Mode**: Always On
- **Description**: Accessibility requirements for iOS apps per Apple guidelines

---

## Core Accessibility Properties

### Required Properties for Interactive Elements

```typescript
// EVERY interactive element MUST have accessibility properties
<Pressable
  accessible={true}
  accessibilityRole="button"
  accessibilityLabel="Start lesson"
  accessibilityHint="Opens the lesson screen"
  accessibilityState={{
    disabled: isDisabled,
    selected: isSelected,
  }}
  onPress={handlePress}
>
  <Text>Start</Text>
</Pressable>
```

### Accessibility Roles (iOS Standard)

```typescript
type AccessibilityRole =
  | 'none' // No role
  | 'button' // Tappable elements
  | 'link' // Navigation links
  | 'search' // Search fields
  | 'image' // Images
  | 'imagebutton' // Tappable images
  | 'text' // Static text
  | 'header' // Section headers
  | 'adjustable' // Sliders, steppers
  | 'alert' // Alert dialogs
  | 'checkbox' // Checkboxes
  | 'combobox' // Dropdown selects
  | 'menu' // Menu containers
  | 'menubar' // Menu bars
  | 'menuitem' // Menu items
  | 'progressbar' // Progress indicators
  | 'radio' // Radio buttons
  | 'radiogroup' // Radio button groups
  | 'scrollbar' // Scroll indicators
  | 'spinbutton' // Numeric steppers
  | 'switch' // Toggle switches
  | 'tab' // Tab buttons
  | 'tablist' // Tab containers
  | 'timer' // Countdown timers
  | 'toolbar'; // Toolbars
```

---

## Accessibility Labels

### Label Writing Rules

```typescript
// GOOD: Descriptive, action-oriented
accessibilityLabel="Play audio"
accessibilityLabel="Delete message"
accessibilityLabel="Mark as complete"

// BAD: Vague, non-descriptive
accessibilityLabel="Button"
accessibilityLabel="Icon"
accessibilityLabel="Press here"

// GOOD: Include state information
accessibilityLabel={`Lesson ${index + 1}, ${isCompleted ? 'completed' : 'not completed'}`}

// BAD: Redundant information
accessibilityLabel="Button: Submit button" // Don't repeat "button"
```

### Label Patterns by Element Type

```typescript
// Buttons
accessibilityLabel="Submit form"
accessibilityHint="Saves your changes and closes the screen"

// Icons
accessibilityLabel="Settings"
// Note: Decorative icons should be hidden
accessibilityElementsHidden={true}

// Images
accessibilityLabel="Profile photo of John Doe"
// OR for decorative images:
accessible={false}

// Progress indicators
accessibilityLabel={`Loading, ${progress}% complete`}
accessibilityRole="progressbar"
accessibilityValue={{
  min: 0,
  max: 100,
  now: progress,
}}
```

---

## Accessibility Hints

### Hint Guidelines

```typescript
// Hints describe the RESULT of the action
// GOOD: Describes outcome
accessibilityHint = 'Opens your profile settings';
accessibilityHint = 'Removes this item from your cart';
accessibilityHint = 'Navigates to the previous screen';

// BAD: Describes how to interact
accessibilityHint = 'Tap to open'; // VoiceOver already says this
accessibilityHint = 'Double tap to activate'; // Redundant
```

### When to Use Hints

```typescript
// USE hints for:
// - Non-obvious actions
// - Actions with consequences
// - Navigation actions

// DON'T use hints for:
// - Obvious actions (e.g., "Save" button saves)
// - Simple toggles (state is announced)
// - Standard navigation patterns
```

---

## Accessibility State

### State Properties

```typescript
<Pressable
  accessibilityState={{
    disabled: boolean,    // Element cannot be interacted with
    selected: boolean,    // Element is selected (tabs, options)
    checked: boolean,     // Checkbox/radio is checked
    busy: boolean,        // Element is loading/processing
    expanded: boolean,    // Expandable element is open
  }}
/>
```

### Dynamic State Updates

```typescript
// Update state for live feedback
const [isLoading, setIsLoading] = useState(false);

<Pressable
  accessibilityState={{
    busy: isLoading,
    disabled: isLoading,
  }}
  accessibilityLabel={isLoading ? "Submitting..." : "Submit"}
>
  {isLoading ? <ActivityIndicator /> : <Text>Submit</Text>}
</Pressable>
```

---

## Accessibility Value

### For Adjustable Elements

```typescript
// Sliders, steppers, progress bars
<View
  accessibilityRole="adjustable"
  accessibilityValue={{
    min: 0,
    max: 100,
    now: currentValue,
    text: `${currentValue} percent`,
  }}
  accessibilityActions={[
    { name: 'increment', label: 'Increase' },
    { name: 'decrement', label: 'Decrease' },
  ]}
  onAccessibilityAction={(event) => {
    switch (event.nativeEvent.actionName) {
      case 'increment':
        setValue(Math.min(max, currentValue + step));
        break;
      case 'decrement':
        setValue(Math.max(min, currentValue - step));
        break;
    }
  }}
/>
```

---

## Color & Contrast

### Minimum Contrast Requirements (WCAG 2.1 AA)

```typescript
// Text contrast ratios
const CONTRAST_RATIOS = {
  normalText: 4.5, // Text < 18pt
  largeText: 3.0, // Text >= 18pt or bold >= 14pt
  uiComponents: 3.0, // Icons, borders, controls
  graphicalObjects: 3.0, // Charts, diagrams
};

// Example compliant color pairs
const accessibleColors = {
  // Dark background
  darkBg: '#0C0A17',
  textOnDark: '#F9FAFB', // Contrast: 15.8:1 ✓
  secondaryOnDark: '#9CA3AF', // Contrast: 5.4:1 ✓

  // Light background
  lightBg: '#FFFFFF',
  textOnLight: '#1F2937', // Contrast: 14.7:1 ✓
};
```

### Never Use Color Alone

```typescript
// WRONG: Color only indicates status
<View style={{ backgroundColor: isError ? 'red' : 'green' }} />

// CORRECT: Color + icon + text
<View style={{ backgroundColor: isError ? colors.error : colors.success }}>
  {isError ? <XIcon /> : <CheckIcon />}
  <Text>{isError ? 'Error occurred' : 'Success'}</Text>
</View>
```

---

## Focus Management

### Accessibility Focus

```typescript
import { AccessibilityInfo, findNodeHandle } from 'react-native';

// Move focus to element
const ref = useRef<View>(null);

const focusOnElement = () => {
  const node = findNodeHandle(ref.current);
  if (node) {
    AccessibilityInfo.setAccessibilityFocus(node);
  }
};

// Use after navigation or content changes
useEffect(() => {
  if (showNewContent) {
    focusOnElement();
  }
}, [showNewContent]);
```

### Focus Order

```typescript
// Use accessibilityElementsHidden to exclude decorative elements
<View>
  <DecorationView accessibilityElementsHidden={true} />
  <MainContent accessible={true} />
</View>

// Group related elements
<View
  accessible={true}
  accessibilityLabel="User profile: John Doe, Premium member"
>
  <Avatar />
  <Text>John Doe</Text>
  <Badge>Premium</Badge>
</View>
```

---

## Live Regions

### Announce Dynamic Content

```typescript
import { AccessibilityInfo } from 'react-native';

// Announce important changes
const announceToVoiceOver = (message: string) => {
  AccessibilityInfo.announceForAccessibility(message);
};

// Example: Toast notification
const showToast = (message: string) => {
  setToastMessage(message);
  announceToVoiceOver(message);
};

// Example: Form validation
const handleSubmit = () => {
  if (hasErrors) {
    announceToVoiceOver(`Form has ${errors.length} errors. First error: ${errors[0]}`);
  }
};
```

### Live Region Properties

```typescript
// For elements that update dynamically
<Text
  accessibilityLiveRegion="polite" // Announces when idle
  // or "assertive" for critical updates
>
  {statusMessage}
</Text>
```

---

## Reduced Motion

### Respect User Preferences

```typescript
import { useReducedMotion } from 'moti';
// OR
import { AccessibilityInfo } from 'react-native';

// Using Moti hook
const Component = () => {
  const reducedMotion = useReducedMotion();

  return (
    <MotiView
      animate={{ opacity: 1 }}
      transition={reducedMotion
        ? { type: 'timing', duration: 0 }
        : { type: 'spring' }
      }
    />
  );
};

// Using native API
const [reduceMotion, setReduceMotion] = useState(false);

useEffect(() => {
  AccessibilityInfo.isReduceMotionEnabled().then(setReduceMotion);

  const subscription = AccessibilityInfo.addEventListener(
    'reduceMotionChanged',
    setReduceMotion
  );

  return () => subscription.remove();
}, []);
```

---

## Testing Accessibility

### Manual Testing Checklist

```
□ VoiceOver: Enable and navigate entire app
□ Dynamic Type: Test with largest text size
□ Reduce Motion: Verify animations respect preference
□ Color Filters: Test with grayscale enabled
□ Switch Control: Verify all interactive elements reachable
□ Bold Text: Verify text remains legible
```

### Automated Testing

```typescript
// Use react-native-testing-library
import { render, screen } from '@testing-library/react-native';

test('button has accessibility label', () => {
  render(<MyButton label="Submit" />);

  expect(screen.getByRole('button', { name: 'Submit' })).toBeTruthy();
});
```

---

## Forbidden Accessibility Practices

1. **NEVER** omit accessibility labels on interactive elements
2. **NEVER** use color alone to convey information
3. **NEVER** use contrast ratios below WCAG AA standards
4. **NEVER** disable accessibility on form inputs
5. **NEVER** ignore reduced motion preferences
6. **NEVER** use time-based interactions without alternatives
7. **NEVER** trap keyboard/switch control focus
8. **NEVER** hide critical content from screen readers

---
> Source: [denker-systems/aiklubben-app](https://github.com/denker-systems/aiklubben-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
