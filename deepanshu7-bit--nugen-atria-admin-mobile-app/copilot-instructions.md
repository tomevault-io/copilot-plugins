## nugen-atria-admin-mobile-app

> This is a React Native/Expo admin mobile application for hotel management with order and booking management features. Built with TypeScript, React Navigation, and a bottom-tab navigation pattern.

# Nugen Atria Admin Mobile App - Cursor Rules

## Project Overview
This is a React Native/Expo admin mobile application for hotel management with order and booking management features. Built with TypeScript, React Navigation, and a bottom-tab navigation pattern.

---

## Tech Stack

### Core Framework
- **React Native**: 0.79.3
- **Expo**: ~53.0.11 (managed workflow)
- **React**: 19.0.0
- **TypeScript**: ~5.8.3 (strict mode enabled)

### Navigation
- **@react-navigation/native**: ^6.1.18
- **@react-navigation/native-stack**: ^6.11.0 (Stack navigation)
- **@react-navigation/bottom-tabs**: ^6.6.1 (Tab navigation)

### Development Tools
- **ESLint**: ^9.22.0 with `eslint-config-expo` and TypeScript plugin
- **Babel**: @babel/core ^7.25.2

### UI Libraries
- **@expo/vector-icons**: ^14.1.0 (MaterialIcons)
- **react-native-safe-area-context**: ~5.4.0
- **react-native-screens**: ~4.11.1

---

## Project Structure

### `/src/data`
- Mock data and fixtures for development
- Contains: `mockData.ts` with sample orders and bookings
- Use named exports for all data

### `/src/features`
- Feature-driven, domain-based organization
- Each feature directory (dashboard, orders, bookings, settings) contains:
  - `screens/` - Main screen components
  - `components/` - Feature-specific reusable components
- Features are independent and self-contained

### `/src/navigation`
- `RootNavigator.tsx` - Root stack navigator (tabs + detail screens)
- `MainTabsNavigator.tsx` - Bottom tab navigator
- `types.ts` - TypeScript param lists for type-safe navigation

### `/src/shared`
- `components/` - Shared components used across features
- `theme/` - Design tokens (colors, typography, spacing)

### `/src/types`
- `domain.ts` - Shared domain types and interfaces
- Contains: Order, Booking, OrderStatus types

---

## TypeScript Conventions

### Configuration
- Strict mode enabled in `tsconfig.json`
- Path alias: `@/*` maps to `src/*`
- Use for clean cross-feature imports

### Type Definitions
- Use `interface` for component props and domain objects
- Use `type` for unions, intersections, and param lists
- Export types with `export type` or `export interface`

### Component Props
```typescript
interface ComponentNameProps {
  item: DomainType;
  onPress: () => void;
}

export function ComponentName({ item, onPress }: ComponentNameProps) {
  // Implementation
}
```

### Navigation Types
```typescript
// Define in src/navigation/types.ts
export type RootStackParamList = {
  ScreenName: { param1: string };
  ScreenWithoutParams: undefined;
};

// Use in components
type NavigationProp = NativeStackNavigationProp<RootStackParamList>;
const navigation = useNavigation<NavigationProp>();
```

---

## Component Patterns

### Functional Components
- Always use named function declarations
- Never use default exports for components
- Use arrow functions for callbacks and handlers

```typescript
// ✅ Correct
export function MyComponent() { ... }

// ❌ Incorrect
export default function MyComponent() { ... }
const MyComponent = () => { ... }
```

### Screen Components
- Name with `Screen` suffix (e.g., `OrdersScreen.tsx`)
- Wrap content in `<ScreenContainer>` for consistent layout
- Structure: imports → component → styles

```typescript
import React from 'react';
import { StyleSheet, Text } from 'react-native';
import { ScreenContainer } from '@/shared/components/ScreenContainer';
import { colors } from '@/shared/theme/colors';

export function ExampleScreen() {
  return (
    <ScreenContainer>
      <Text style={styles.title}>Title</Text>
    </ScreenContainer>
  );
}

const styles = StyleSheet.create({
  title: {
    fontSize: 24,
    fontWeight: '700',
    color: colors.text
  }
});
```

### Reusable Components
- Define props interface above component
- Export component with named export
- Colocate styles at the bottom

```typescript
interface CardProps {
  item: DataType;
  onPress: () => void;
}

export function Card({ item, onPress }: CardProps) {
  return (/* JSX */);
}

const styles = StyleSheet.create({
  // Styles
});
```

---

## Styling Guidelines

### Theme Colors
Always use the centralized theme colors from `@/shared/theme/colors`:
```typescript
import { colors } from '@/shared/theme/colors';

// Available colors:
colors.primary      // #1E40AF - Primary brand blue
colors.background   // #F8FAFC - Light background
colors.card         // #FFFFFF - Card/surface background
colors.text         // #0F172A - Primary text
colors.muted        // #64748B - Secondary/muted text
colors.success      // #15803D - Success indicators
colors.border       // #E2E8F0 - Borders
```

### StyleSheet Pattern
- Use `StyleSheet.create()` for all styles
- Colocate styles at the bottom of the component file
- No external stylesheet files

### Standard Spacing
- Screen padding: `16`
- Card padding: `14`
- Gap between elements: `12`
- Border radius for cards: `14`
- Border width: `1`

### Font Weights
- Bold: `'700'`
- Semi-bold: `'600'`
- Regular: default (no specification needed)

### Card Pattern
```typescript
const styles = StyleSheet.create({
  card: {
    backgroundColor: colors.card,
    borderRadius: 14,
    padding: 14,
    borderWidth: 1,
    borderColor: colors.border
  }
});
```

---

## Import Organization

Follow this order for imports:

1. React core
2. React Native components
3. Third-party libraries (navigation, expo, etc.)
4. Local imports using `@/` alias
5. Feature-relative imports

```typescript
// 1. React core
import React from 'react';

// 2. React Native
import { StyleSheet, Text, View } from 'react-native';

// 3. Third-party
import { useNavigation } from '@react-navigation/native';
import { NativeStackNavigationProp } from '@react-navigation/native-stack';

// 4. Local @ imports
import { orders } from '@/data/mockData';
import { ScreenContainer } from '@/shared/components/ScreenContainer';
import { colors } from '@/shared/theme/colors';
import { Order } from '@/types/domain';

// 5. Feature-relative imports
import { OrderCard } from '../components/OrderCard';
```

---

## Navigation Guidelines

### Screen Registration
- Register main tab screens in `MainTabsNavigator.tsx`
- Register detail/modal screens in `RootNavigator.tsx`
- Define params in `src/navigation/types.ts`

### Navigation Options
- Bottom tab screens: `headerShown: false`
- Stack screens: Show header with consistent styling
- Use MaterialIcons for tab bar icons

### Navigating Between Screens
```typescript
import { useNavigation } from '@react-navigation/native';
import { NativeStackNavigationProp } from '@react-navigation/native-stack';
import { RootStackParamList } from '@/navigation/types';

type NavigationProp = NativeStackNavigationProp<RootStackParamList>;

export function MyScreen() {
  const navigation = useNavigation<NavigationProp>();
  
  const handleNavigate = () => {
    navigation.navigate('DetailScreen', { id: '123' });
  };
  
  // ...
}
```

---

## Development Commands

### Running the App
```bash
npm start          # Start Expo dev server
npm run android    # Run on Android emulator/device
npm run ios        # Run on iOS simulator/device
```

### Code Quality
```bash
npm run lint       # Run ESLint
npm run typecheck  # Run TypeScript type checking
```

### Pre-commit Checklist
1. Run `npm run typecheck` - Must pass
2. Run `npm run lint` - Must pass
3. Test on device/simulator

---

## Feature Development Workflow

### Adding a New Feature
1. Create `src/features/{feature-name}/` directory
2. Add subdirectories: `screens/`, `components/`
3. Create screen: `{Feature}Screen.tsx`
4. Add navigation route in appropriate navigator
5. Update `types.ts` with navigation params
6. Add domain types in `src/types/domain.ts` if needed

### Adding Mock Data
1. Add to `src/data/mockData.ts`
2. Export with named export
3. Import using `@/data/mockData` in features

### Creating Shared Components
1. Add to `src/shared/components/`
2. Use PascalCase filename
3. Export with named export
4. Import using `@/shared/components/{ComponentName}`

---

## Dos and Don'ts

### ✅ DO
- Use TypeScript strict mode
- Use `@/` path alias for imports from src/
- Use theme colors from `@/shared/theme/colors`
- Wrap all screens with `<ScreenContainer>`
- Use named exports for all components
- Colocate styles with components using `StyleSheet.create()`
- Define component props with interfaces
- Use type-safe navigation with param lists
- Use `useMemo` for expensive computations
- Use MaterialIcons from `@expo/vector-icons`
- Follow PascalCase for component files and names
- Use feature-driven directory structure

### ❌ DON'T
- Don't hardcode colors (always use theme)
- Don't use default exports for components
- Don't create external stylesheet files
- Don't use inline styles for complex components
- Don't prop drill beyond 2 levels
- Don't import React Native components with wildcards
- Don't skip TypeScript types for component props
- Don't use anonymous arrow function components
- Don't mix styling approaches
- Don't create untypeable navigation

---

## Code Quality Standards

### TypeScript
- Enable and maintain strict mode
- No implicit `any` types
- Define interfaces for all component props
- Type all navigation parameters
- Use proper return types for functions

### ESLint
- Follow `eslint-config-expo` rules
- Follow `@typescript-eslint/recommended` rules
- Fix all linting errors before committing

### Code Organization
- Keep related code together (screens, components, types)
- Use feature-based organization
- Maintain clear separation between shared and feature-specific code
- One component per file

---

## Example Templates

### Full Screen Example
```typescript
import React from 'react';
import { StyleSheet, Text, View, FlatList } from 'react-native';
import { useNavigation } from '@react-navigation/native';
import { NativeStackNavigationProp } from '@react-navigation/native-stack';

import { data } from '@/data/mockData';
import { ScreenContainer } from '@/shared/components/ScreenContainer';
import { colors } from '@/shared/theme/colors';
import { RootStackParamList } from '@/navigation/types';
import { ItemType } from '@/types/domain';
import { ItemCard } from '../components/ItemCard';

type ScreenNavigationProp = NativeStackNavigationProp<RootStackParamList>;

export function ListScreen() {
  const navigation = useNavigation<ScreenNavigationProp>();

  const handleItemPress = (id: string) => {
    navigation.navigate('DetailScreen', { id });
  };

  return (
    <ScreenContainer>
      <Text style={styles.title}>Screen Title</Text>
      <FlatList
        data={data}
        renderItem={({ item }) => (
          <ItemCard 
            item={item} 
            onPress={() => handleItemPress(item.id)} 
          />
        )}
        keyExtractor={(item) => item.id}
        contentContainerStyle={styles.list}
      />
    </ScreenContainer>
  );
}

const styles = StyleSheet.create({
  title: {
    fontSize: 24,
    fontWeight: '700',
    color: colors.text,
    marginBottom: 16
  },
  list: {
    gap: 12
  }
});
```

### Card Component Example
```typescript
import React from 'react';
import { StyleSheet, Text, View, TouchableOpacity } from 'react-native';
import { ItemType } from '@/types/domain';
import { colors } from '@/shared/theme/colors';

interface ItemCardProps {
  item: ItemType;
  onPress: () => void;
}

export function ItemCard({ item, onPress }: ItemCardProps) {
  return (
    <TouchableOpacity onPress={onPress} style={styles.card}>
      <Text style={styles.title}>{item.title}</Text>
      <Text style={styles.description}>{item.description}</Text>
    </TouchableOpacity>
  );
}

const styles = StyleSheet.create({
  card: {
    backgroundColor: colors.card,
    borderRadius: 14,
    padding: 14,
    borderWidth: 1,
    borderColor: colors.border
  },
  title: {
    fontSize: 16,
    fontWeight: '700',
    color: colors.text,
    marginBottom: 4
  },
  description: {
    fontSize: 14,
    color: colors.muted
  }
});
```

---

## Quick Reference

### Common Imports
```typescript
// Screens
import { ScreenContainer } from '@/shared/components/ScreenContainer';
import { colors } from '@/shared/theme/colors';

// Navigation
import { useNavigation } from '@react-navigation/native';
import { NativeStackNavigationProp } from '@react-navigation/native-stack';
import { RootStackParamList } from '@/navigation/types';

// Types
import { Order, Booking } from '@/types/domain';

// Data
import { orders, bookings } from '@/data/mockData';
```

### File Naming Patterns
- Screens: `{Feature}Screen.tsx` (e.g., `OrdersScreen.tsx`)
- Components: `{ComponentName}.tsx` (e.g., `OrderCard.tsx`)
- Types: `domain.ts`, `types.ts`
- Navigation: `RootNavigator.tsx`, `MainTabsNavigator.tsx`

### Standard Spacing Values
- Screen padding: `16`
- Card padding: `14`
- Element gaps: `12`
- Border radius: `14`
- Border width: `1`

---

## Best Practices

### Performance
- Use `useMemo` for expensive computations
- Use `useCallback` for memoized callbacks
- Use `React.memo` sparingly (only when needed)
- Use `FlatList` for long scrollable lists

### Code Quality
- All code must pass TypeScript strict checks
- All code must pass ESLint checks
- Use meaningful variable and function names
- Keep components focused and single-purpose

### Accessibility
- Provide accessible labels where appropriate
- Use semantic component structure
- Consider touch target sizes (minimum 44x44)

---

## Adding New Features

### Step-by-Step Process
1. **Create feature directory**: `src/features/{feature-name}/`
2. **Add subdirectories**: `screens/`, `components/`
3. **Create screen**: `{Feature}Screen.tsx` in screens/
4. **Add types**: Update `src/types/domain.ts` if needed
5. **Add mock data**: Update `src/data/mockData.ts` if needed
6. **Register navigation**: Add to appropriate navigator
7. **Update types**: Add to `src/navigation/types.ts`
8. **Test**: Run typecheck and lint

### Navigation Setup
1. For tab screens: Add to `MainTabsNavigator.tsx`
2. For detail screens: Add to `RootNavigator.tsx`
3. Define params in `src/navigation/types.ts`
4. Use typed navigation hook in component

---

## Common Patterns

### List Screen Pattern
```typescript
export function ListScreen() {
  const navigation = useNavigation<NavigationProp>();

  return (
    <ScreenContainer>
      <Text style={styles.title}>Title</Text>
      <FlatList
        data={items}
        renderItem={({ item }) => (
          <ItemCard 
            item={item}
            onPress={() => navigation.navigate('Detail', { id: item.id })}
          />
        )}
        keyExtractor={(item) => item.id}
        contentContainerStyle={styles.list}
      />
    </ScreenContainer>
  );
}
```

### Dashboard KPI Pattern
```typescript
export function DashboardScreen() {
  const totalValue = useMemo(() => {
    return data.reduce((sum, item) => sum + item.value, 0);
  }, [data]);

  return (
    <ScreenContainer>
      <View style={styles.kpiContainer}>
        <KpiCard label="Total" value={totalValue} />
      </View>
    </ScreenContainer>
  );
}
```

### Card Component Pattern
```typescript
interface CardProps {
  item: ItemType;
  onPress: () => void;
}

export function Card({ item, onPress }: CardProps) {
  return (
    <TouchableOpacity onPress={onPress} style={styles.card}>
      {/* Card content */}
    </TouchableOpacity>
  );
}

const styles = StyleSheet.create({
  card: {
    backgroundColor: colors.card,
    borderRadius: 14,
    padding: 14,
    borderWidth: 1,
    borderColor: colors.border
  }
});
```

---

## Entry Point

### App.tsx (Root Component)
- Located at project root (not in src/)
- Wraps app with required providers:
  - `SafeAreaProvider`
  - `NavigationContainer`
  - `StatusBar`
- Renders `RootNavigator`

```typescript
export default function App() {
  return (
    <SafeAreaProvider>
      <NavigationContainer>
        <StatusBar style="dark" />
        <RootNavigator />
      </NavigationContainer>
    </SafeAreaProvider>
  );
}
```

---

## Testing & Validation

### Before Committing
1. Run `npm run typecheck` - Must pass with no errors
2. Run `npm run lint` - Must pass with no errors
3. Test on device/simulator
4. Verify hot reload works

### Code Review Checklist
- [ ] Uses theme colors (no hardcoded colors)
- [ ] Props interface defined
- [ ] Named exports used
- [ ] Styles colocated with component
- [ ] TypeScript types defined
- [ ] Import organization follows convention
- [ ] ScreenContainer used for screens
- [ ] Navigation properly typed

---

## Notes

- This is an admin-focused mobile app for hotel staff
- Currently uses mock data (no API integration yet)
- Bottom tab navigation is the primary navigation pattern
- Status bar style is set to "dark"
- No state management library (local state only)
- No testing framework configured yet

---
> Source: [Deepanshu7-bit/nugen-atria-admin-mobile-app](https://github.com/Deepanshu7-bit/nugen-atria-admin-mobile-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
