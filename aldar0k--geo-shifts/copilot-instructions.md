## react-native

> React Native CLI projects have native folders (/android, /ios) that you should rarely touch. Your main work area is the TypeScript/JavaScript layer.


# React Native with FSD

## Project Structure

React Native CLI projects have native folders (/android, /ios) that you should rarely touch. Your main work area is the TypeScript/JavaScript layer.

### Working Directory Structure

Create /src folder in the root and organize all application code there following FSD:

```
/src
  /app
    providers/         ← MobX Provider, NavigationContainer
    index.tsx          ← root App component
  /processes
    navigation/        ← navigation config, navigators
    geo/               ← geolocation & permissions handling
  /pages
    shifts-list/
      ui/ShiftsListScreen.tsx
    shift-details/
      ui/ShiftDetailsScreen.tsx
  /widgets
    shifts-list/
      ui/ShiftsList.tsx
  /features
    refresh-shifts/
      model/
      ui/
  /entities
    shift/
      api/shifts.ts
      model/shifts.store.ts
      ui/ShiftCard.tsx
  /shared
    api/              ← axios/fetch config, base API
    config/           ← constants, env
    ui/               ← Button, Input, Card, etc.
    lib/              ← utils (date, geo, formatting)
    hooks/            ← custom hooks
```

## React Native Specific Rules

### Components

- Use React Native components: View, Text, Image, ScrollView, FlatList, TouchableOpacity, etc.
- NEVER use HTML elements (div, span, img, button) - they don't exist in RN
- For styling use StyleSheet.create() or inline styles (no CSS files, no Tailwind)
- Use React Native's flex layout (flexbox is default, no need for display: flex)

### Navigation

- Place navigation configuration in /processes/navigation or /app
- Pages must be registered in navigator (Stack.Navigator, Tab.Navigator, Drawer.Navigator)
- Use React Navigation (@react-navigation/native) for navigation
- Navigation types should be defined and exported from navigation config

### State Management (MobX)

- Place stores in entities/_/model/_.store.ts for domain entities
- Place stores in features/_/model/_.store.ts for feature-specific state
- Global stores go to app/stores/
- Use mobx-react-lite hooks (observer, useLocalObservableStore)
- Configure MobX provider in app/providers/

### Processes Layer (React Native specific)

Use processes/ for cross-cutting native integrations:

- processes/geo/ - geolocation, location permissions
- processes/push/ - push notifications
- processes/auth/ - authentication flow
- processes/navigation/ - navigation configuration

### Native Layer (/android, /ios)

Touch native code ONLY for:

- Adding permissions (AndroidManifest.xml, Info.plist)
- Changing app icon/splash screen
- Adding native dependencies that require linking
- Changing package name/bundle identifier

### Permissions

- Request permissions using react-native-permissions or native modules
- Permission logic goes to processes/ (e.g., processes/geo/model/permissions.ts)
- Always check permission status before requesting
- Handle all permission states (granted, denied, blocked, unavailable)

### API Layer

- Place API configuration in shared/api/
- Entity-specific API calls go to entities/\*/api/
- Use axios or fetch
- Define TypeScript interfaces for API responses in entities/\*/model/types.ts

### TypeScript

- Define types for navigation params
- Define types for API responses
- Define types for store models
- Use strict mode in tsconfig.json
- Avoid 'any' type

### Performance

- Use React.memo for list items
- Use FlatList for long lists (not ScrollView with .map())
- Optimize images (use appropriate sizes, formats)
- Use lazy loading where appropriate

### File Naming

- Screens: \*Screen.tsx (e.g., ShiftsListScreen.tsx)
- Components: PascalCase (e.g., ShiftCard.tsx)
- Stores: \*.store.ts (e.g., shifts.store.ts)
- API files: \*.api.ts or just api.ts inside entity folder
- Types: \*.types.ts or types.ts

## Entry Point

- index.js (root) registers the app: AppRegistry.registerComponent()
- App.tsx imports from src/app/index.tsx
- src/app/index.tsx wraps app with providers and navigation

---
> Source: [Aldar0K/geo-shifts](https://github.com/Aldar0K/geo-shifts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-12 -->
