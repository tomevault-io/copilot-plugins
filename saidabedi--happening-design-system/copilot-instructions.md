## happening-design-system

> A neumorphic design system for React Native with Mediterranean warmth (Terracotta + Teal palette).

# Design System - @saidabedi/design-system

A neumorphic design system for React Native with Mediterranean warmth (Terracotta + Teal palette).

## Installation

```bash
npm install @saidabedi/design-system

# For icons (required for Icon component)
npx expo install react-native-svg
# or
npm install react-native-svg
```

## Usage (React Native)

```tsx
// Import components and tokens from /native
import {
  NeumorphicView,
  Card,
  Button,
  Icon,
  colors,
  colorsDark,
  spacing,
  radii,
} from '@saidabedi/design-system/native'

// Theme-aware colors
const c = isDark ? colorsDark : colors
```

---

## Components

### NeumorphicView

A View with neumorphic styling. Supports raised, cut, and flat variants.

```tsx
import { NeumorphicView } from '@saidabedi/design-system/native'

// Raised (default) - floating above surface
<NeumorphicView variant="raised">
  <Text>Floating content</Text>
</NeumorphicView>

// Cut - pressed into surface (selected state)
<NeumorphicView variant="cut">
  <Text>Selected item</Text>
</NeumorphicView>

// Interactive with press handling
<NeumorphicView
  variant={isSelected ? "cut" : "raised"}
  onPress={() => setSelected(!isSelected)}
  showPressedState
>
  <Text>Tap me</Text>
</NeumorphicView>
```

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `variant` | `'raised'` \| `'cut'` \| `'flat'` | `'raised'` | Neumorphic style |
| `mode` | `'light'` \| `'dark'` | `'light'` | Color mode |
| `radius` | `keyof radii` \| `number` | `'2xl'` | Border radius |
| `showPressedState` | `boolean` | `false` | Show pressed feedback |
| `onPress` | `() => void` | - | Press handler |
| `accentBorder` | `boolean` | `false` | Add gradient border (terracotta → teal) |
| `LinearGradient` | `Component` | - | Required for accentBorder |

### Card

A NeumorphicView with padding presets.

```tsx
import { Card } from '@saidabedi/design-system/native'
import { LinearGradient } from 'expo-linear-gradient'

<Card variant="raised" size="md">
  <Text>Card content</Text>
</Card>

// With gradient accent border (terracotta → teal)
<Card variant="raised" accentBorder LinearGradient={LinearGradient}>
  <Text>Card with accent border</Text>
</Card>

// Interactive card with accent border
<Card
  variant={isSelected ? "cut" : "raised"}
  interactive
  accentBorder
  LinearGradient={LinearGradient}
  onPress={() => selectItem()}
>
  <Text>Selectable card</Text>
</Card>
```

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `variant` | `'raised'` \| `'cut'` \| `'flat'` \| `'bordered'` | `'raised'` | Visual style |
| `size` | `'sm'` \| `'md'` \| `'lg'` | `'md'` | Padding (24/32/48) |
| `interactive` | `boolean` | `false` | Enable press states |
| `mode` | `'light'` \| `'dark'` | `'light'` | Color mode |
| `accentBorder` | `boolean` | `false` | Add gradient border (terracotta → teal) |
| `LinearGradient` | `Component` | - | Required for accentBorder (expo-linear-gradient or react-native-linear-gradient) |

### Button

A neumorphic button component.

```tsx
import { Button } from '@saidabedi/design-system/native'

<Button variant="primary" onPress={handlePress}>
  Click me
</Button>

<Button variant="secondary" size="lg">
  Large secondary
</Button>

<Button variant="outline" mode="dark">
  Outline button
</Button>
```

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `variant` | `'primary'` \| `'secondary'` \| `'ghost'` \| `'outline'` | `'primary'` | Visual style |
| `size` | `'sm'` \| `'md'` \| `'lg'` | `'md'` | Button size |
| `mode` | `'light'` \| `'dark'` | `'light'` | Color mode |
| `disabled` | `boolean` | `false` | Disabled state |

### Icon

Custom SVG icons with gradient support using react-native-svg.

```tsx
import { Icon } from '@saidabedi/design-system/native'

// Basic usage
<Icon name="calendar" size="lg" />

// With gradient fill
<Icon name="heart" gradient="brand" />

// With gradient stroke only (border)
<Icon name="heart-outline" gradient="brand" gradientMode="stroke" />

// Custom color
<Icon name="search" color="#C4756A" />

// Dark mode
<Icon name="moon" mode="dark" />
```

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `name` | `IconName` | required | Icon name |
| `size` | `'sm'` \| `'md'` \| `'lg'` \| `'xl'` \| `number` | `'md'` | Size (16/20/24/32) |
| `color` | `string` | text.primary | Icon color |
| `gradient` | `'brand'` \| `'sunset'` \| `'ocean'` \| `IconGradient` | - | Gradient fill |
| `gradientMode` | `'fill'` \| `'stroke'` \| `'both'` | `'both'` | How gradient applies |
| `mode` | `'light'` \| `'dark'` | `'light'` | Color mode |

**Available Icons:**
`calendar`, `search`, `heart`, `heart-outline`, `close-circle`, `sunny`, `moon`, `phone-portrait`, `options`

---

## Common Patterns

### Selected vs Unselected States

```tsx
// Selected: cut surface + gradient filled icon
<NeumorphicView variant="cut">
  <Icon name="calendar" size="lg" gradient="brand" />
</NeumorphicView>

// Unselected: raised surface + outline icon
<NeumorphicView variant="raised">
  <Icon name="calendar" size="lg" gradient="brand" gradientMode="stroke" />
</NeumorphicView>
```

### Event Card Example

```tsx
import { LinearGradient } from 'expo-linear-gradient'

<Card
  variant={isSelected ? "cut" : "raised"}
  interactive
  accentBorder
  LinearGradient={LinearGradient}
  mode={isDark ? 'dark' : 'light'}
  onPress={() => selectEvent(event.id)}
>
  <Icon
    name="calendar"
    size="lg"
    gradient="brand"
    gradientMode={isSelected ? 'fill' : 'stroke'}
    mode={isDark ? 'dark' : 'light'}
  />
  <Text style={{ color: c.text.primary }}>{event.title}</Text>
</Card>
```

---

## Design Tokens

### Colors

**Light mode:** `colors` | **Dark mode:** `colorsDark`

```tsx
// Brand
c.brand.primary          // #C4756A (terracotta)
c.brand.secondary        // #5A9A9A (teal)

// Surfaces (for neumorphic cards)
c.surface.raised         // #FFFFFF (light) / #292524 (dark) - floating
c.surface.base           // #FDF8F4 (light) / #1C1917 (dark) - default
c.surface.cut            // #F5EDE5 (light) / #1A1816 (dark) - pressed/inset

// Backgrounds
c.background.primary     // #FDF8F4 (light) / #1C1917 (dark)

// Text
c.text.primary           // #2D2420 (light) / #FAF5F0 (dark)
c.text.secondary         // #5C4D45 (light) / #C4B8B0 (dark)
c.text.muted             // #8A7A70 (light) / #9A8A80 (dark)

// Icons
c.icon.primary           // Main icon color
c.icon.secondary         // Muted icon color
c.icon.brand             // Terracotta icon

// Feedback
c.feedback.success       // #5A9A9A (teal)
c.feedback.error         // #C4756A (terracotta)
c.feedback.warning       // #D4A574 (amber)

// Interactive
c.interactive.primary       // #C4756A
c.interactive.primaryHover  // #A65A50
c.interactive.primaryActive // #8A4A42
```

### Spacing

```tsx
spacing.xs     // 4
spacing.sm     // 8
spacing.md     // 12
spacing.lg     // 16
spacing.xl     // 20
spacing['2xl'] // 24
spacing['3xl'] // 32
spacing['4xl'] // 40
spacing['5xl'] // 48
```

### Border Radius

```tsx
radii.sm      // 4
radii.md      // 6
radii.lg      // 8
radii.xl      // 12
radii['2xl']  // 16
radii['3xl']  // 24
radii.full    // 9999
```

### Typography

```tsx
// Font families (load these fonts in your app)
fontFamilies.sans   // "DM Sans"
fontFamilies.serif  // "DM Serif Display"
fontFamilies.mono   // "JetBrains Mono"

// Pre-composed text styles
textStyles.display.hero     // { fontSize: 96, fontFamily: serif }
textStyles.heading.h1       // { fontSize: 36, fontFamily: serif }
textStyles.heading.h4       // { fontSize: 20, fontFamily: sans, fontWeight: 500 }
textStyles.body.medium      // { fontSize: 16, fontFamily: sans }
textStyles.label.medium     // { fontSize: 14, fontFamily: sans, fontWeight: 500 }
textStyles.caption.medium   // { fontSize: 12, fontFamily: sans }
```

### RN Shadows

```tsx
import { rnShadowsLight, rnShadowsDark, getShadow } from '@saidabedi/design-system/native'

const shadows = isDark ? rnShadowsDark : rnShadowsLight

// Apply to a View
<View style={getShadow(shadows['md'])}>
  ...
</View>
```

### Motion

```tsx
durations.fast    // 150 (micro-interactions)
durations.normal  // 200 (standard)
durations.slow    // 300 (emphasis)

easings.default   // Easing.bezier(0.22, 1, 0.36, 1)
easings.out       // Easing.bezier(0, 0, 0.2, 1)
easings.bounce    // Easing.bezier(0.34, 1.56, 0.64, 1)
```

---

## Dark Mode

Always switch between light/dark tokens based on theme:

```tsx
const { colorScheme } = useColorScheme() // or your theme hook
const isDark = colorScheme === 'dark'

const c = isDark ? colorsDark : colors
const bgColor = isDark ? '#1C1917' : '#FDF8F4'
```

---

## Gradients

Built-in gradient presets for icons and components:

```tsx
import { iconGradients } from '@saidabedi/design-system/native'

// Preset gradients
iconGradients.brand     // { start: '#C4756A', end: '#5A9A9A' }  - Terracotta to Teal
iconGradients.sunset    // { start: '#C4756A', end: '#D4A574' }  - Terracotta to Amber
iconGradients.ocean     // { start: '#5A9A9A', end: '#4A8A8A' }  - Teal shades

// Custom gradient
<Icon name="heart" gradient={{ start: '#FF6B6B', end: '#4ECDC4' }} />
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SaidAbedi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SaidAbedi)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
