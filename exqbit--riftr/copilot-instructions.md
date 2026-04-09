## riftr

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Riftr** is a React Native mobile application (iOS/Android) built with Expo for managing Riftbound TCG collections. The app simulates booster pack opening, collection tracking, deck building, and card database browsing.

**Tech Stack**: React Native, Expo SDK 54, TypeScript, Zustand (state management), React Native Paper (UI)
**Platform**: iOS & Android (via Expo Go for development)
**Current Phase**: MVP Phase 1 Complete

## Development Commands

### Setup and Development
```bash
# Install dependencies
npm install

# Start development server
npm start
# or
expo start

# Run on specific platform (during development)
npm run ios      # iOS Simulator (press 'i' in CLI)
npm run android  # Android Emulator (press 'a' in CLI)
npm run web      # Web browser (press 'w' - limited functionality)

# Clear cache if needed
expo start --clear
```

### Production Builds
```bash
# Build for production using EAS
npm install -g eas-cli
eas build --platform ios
eas build --platform android
eas build --platform all
```

### Testing
- No automated tests in MVP phase
- Manual testing via Expo Go app on physical devices
- Use iOS Simulator or Android Emulator for testing

## Architecture Overview

### State Management Architecture

The app uses **Zustand with AsyncStorage persistence**. There are 5 independent stores:

1. **CollectionStore** (`useCollectionStore`)
   - Manages `Map<string, CollectionEntry>` for card ownership
   - Actions: `addToCollection`, `removeFromCollection`, `toggleOwned`, `updateQuantity`
   - Import/Export: `importCollection`, `exportCollection`
   - Stats: `getCollectionStats` (totalCards, uniqueCards, completionRate)

2. **DeckStore** (`useDeckStore`)
   - Manages array of `Deck` objects
   - Actions: `createDeck`, `updateDeck`, `deleteDeck`, `duplicateDeck`
   - Each deck tracks: name, cards with quantities, format, champion, notes

3. **PackStore** (`usePackStore`)
   - Tracks pack opening history (last 100 packs)
   - Virtual currency system (starts at 500💎)
   - Actions: `addPackOpening`, `spendCurrency`, `updateCurrency`

4. **SettingsStore** (`useSettingsStore`)
   - User preferences: theme (dark/light), animations, sound, haptics
   - UI preferences: grid columns (3/4), show owned/unowned filters

5. **StatsStore** (`useStatsStore`)
   - User statistics: total packs opened, collection value, completion rate
   - Actions: `incrementPacksOpened`, `calculateCollectionValue`

### Navigation Structure

```
Root (Stack Navigator)
├── Main (Tab Navigator)
│   ├── Home Tab → HomeScreen (Pack Selection)
│   ├── Collection Tab → CollectionScreen (Owned Cards)
│   ├── Database Tab → DatabaseScreen (Card Search)
│   └── Decks Tab → DecksScreen (Deck List)
└── Modals (Stack Screens)
    ├── CardDetail → CardDetailScreen
    ├── PackOpening → PackOpeningScreen
    ├── DeckBuilder → DeckBuilderScreen
    └── Settings → SettingsScreen
```

### Data Flow Pattern

```
User Action → Screen Component → Zustand Store Action → AsyncStorage Persistence
     ↓              ↓                     ↓                        ↓
  Haptic       Update UI              State Update         Auto-persist
```

**Example: Opening a Booster Pack**
1. User taps pack on HomeScreen
2. `spendCurrency()` called on PackStore
3. Navigate to PackOpeningScreen
4. `generatePackContents()` creates random cards
5. For each card: `addToCollection()` on CollectionStore
6. `incrementPacksOpened()` on StatsStore
7. `addPackOpening()` logs pack to history
8. All stores auto-persist to AsyncStorage

## File Structure

```
riftbound-companion/
├── App.tsx                          # Root component (theme provider, navigation)
├── index.ts                         # Expo entry point
├── app.json                         # Expo configuration
├── package.json                     # Dependencies & scripts
│
├── src/
│   ├── navigation/
│   │   └── index.tsx               # Navigation setup (Tab + Stack)
│   │
│   ├── screens/                    # 8 Main Screens
│   │   ├── HomeScreen.tsx          # Pack selection + currency display
│   │   ├── PackOpeningScreen.tsx  # Pack opening animation + card reveal
│   │   ├── CollectionScreen.tsx   # Grid/List view of owned cards
│   │   ├── DatabaseScreen.tsx     # Searchable card database with filters
│   │   ├── CardDetailScreen.tsx   # Individual card details
│   │   ├── DecksScreen.tsx        # Deck list management
│   │   ├── DeckBuilderScreen.tsx  # Deck editor with mana curve
│   │   └── SettingsScreen.tsx     # App settings & theme toggle
│   │
│   ├── store/
│   │   └── index.ts               # All 5 Zustand stores + persistence
│   │
│   ├── types/
│   │   └── index.ts               # TypeScript interfaces (Card, Deck, etc.)
│   │
│   ├── data/
│   │   └── mockCards.ts           # 50 sample cards + utility functions
│   │
│   └── constants/
│       └── index.ts               # Colors, themes, layouts, pack configs
│
└── assets/                        # App icons, splash screens
```

## Key File Locations

### Core System Files
- Entry Point: `/App.tsx`
- Navigation: `/src/navigation/index.tsx`
- State Management: `/src/store/index.ts`
- Type Definitions: `/src/types/index.ts`
- Constants & Config: `/src/constants/index.ts`
- Mock Data: `/src/data/mockCards.ts`

### Screen Components
- Pack Selection: `/src/screens/HomeScreen.tsx`
- Pack Opening: `/src/screens/PackOpeningScreen.tsx`
- Collection View: `/src/screens/CollectionScreen.tsx`
- Card Database: `/src/screens/DatabaseScreen.tsx`
- Card Details: `/src/screens/CardDetailScreen.tsx`
- Deck List: `/src/screens/DecksScreen.tsx`
- Deck Builder: `/src/screens/DeckBuilderScreen.tsx`
- Settings: `/src/screens/SettingsScreen.tsx`

## Development Patterns

### Screen Component Pattern
```typescript
import { useTheme } from 'react-native-paper';
import * as Haptics from 'expo-haptics';
import { useNavigation } from '@react-navigation/native';

export default function ScreenName() {
  const theme = useTheme();
  const navigation = useNavigation();
  const { state, actions } = useSomeStore();

  const [localState, setLocalState] = useState();

  const handleAction = () => {
    // Haptic feedback for user interactions
    Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium);

    // Business logic
    actions.doSomething();
  };

  return (
    <ScrollView style={{ backgroundColor: theme.colors.background }}>
      {/* UI Components */}
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  // Styles here
});
```

### Zustand Store Pattern
```typescript
export const useSomeStore = create<SomeStore>()(
  persist(
    (set, get) => ({
      // Initial state
      data: [],

      // Actions
      addItem: (item) => {
        set((state) => ({
          data: [...state.data, item],
        }));
      },
    }),
    {
      name: STORAGE_KEYS.something,
      storage: createJSONStorage(() => AsyncStorage),
    }
  )
);
```

### Haptic Feedback Patterns
```typescript
import * as Haptics from 'expo-haptics';

// Light tap - button press
Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light);

// Medium - card selection
Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium);

// Heavy - pack opening
Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Heavy);

// Success/Warning/Error notifications
Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success);
Haptics.notificationAsync(Haptics.NotificationFeedbackType.Warning);
Haptics.notificationAsync(Haptics.NotificationFeedbackType.Error);
```

### Animation Pattern (React Native Animated)
```typescript
import { Animated } from 'react-native';

const [animValue] = useState(() => new Animated.Value(0));

// Spring animation
Animated.spring(animValue, {
  toValue: 1,
  tension: 50,
  friction: 7,
  useNativeDriver: true,
}).start();

// Interpolation
const animatedStyle = {
  transform: [{
    scale: animValue.interpolate({
      inputRange: [0, 1],
      outputRange: [0, 1],
    }),
  }],
  opacity: animValue,
};
```

## Important Data Structures

### Card Interface
```typescript
interface Card {
  id: string;                    // "RB-001"
  name: string;                  // "Viktor, Machine Herald"
  set: string;                   // "Foundations"
  rarity: 'common' | 'uncommon' | 'rare' | 'legendary';
  type: 'champion' | 'unit' | 'spell' | 'relic';
  domain: Domain[];              // ['Piltover & Zaun']
  energy: number;                // Mana cost
  power?: number;                // Attack (units/champions)
  health?: number;               // Health (units/champions)
  abilities: string[];           // ['Hexcore', 'Evolve']
  text: string;                  // Card effect description
  flavorText?: string;           // Lore text
  imageUrl: string;              // Placeholder URL
  artist: string;
  cardNumber: string;
  legality: {
    standard: boolean;
    limited: boolean;
  };
}
```

### CollectionEntry Interface
```typescript
interface CollectionEntry {
  cardId: string;
  owned: boolean;
  quantity: number;
  foil: boolean;
  dateAdded: string;             // ISO string
  variants: ('normal' | 'foil')[];
}
```

### Deck Interface
```typescript
interface Deck {
  id: string;
  name: string;
  champion?: string;             // Champion card ID
  cards: DeckCard[];             // { cardId, quantity }[]
  format: 'standard' | 'limited' | 'unlimited';
  dateCreated: string;           // ISO string
  dateModified: string;          // ISO string
  notes?: string;
}
```

## Theme & Color System

### Rarity Colors (Consistent Throughout App)
```typescript
common: '#808080'       // Gray
uncommon: '#32cd32'     // Green
rare: '#4169e1'         // Blue
legendary: '#9400d3'    // Purple
```

### Dark Theme (Default)
```typescript
primary: '#0596AA'      // Piltover Blue
secondary: '#C89B3C'    // Demacia Gold
accent: '#6b46c1'       // Viktor Purple
background: '#0A0E1A'   // Dark Navy
surface: '#0F1B2F'      // Slightly lighter
card: '#162136'         // Card background
```

### Light Theme (Optional)
- Available via Settings toggle
- Uses standard light colors with same primary/secondary

## Pack Configuration

Located in `src/constants/index.ts`:

```typescript
PACK_CONFIG = {
  starter: {
    cards: 12,
    cost: 0,  // Free
    guaranteed: { champion: 1, rare: 2, uncommon: 4, common: 5 }
  },
  foundations_booster: {
    cards: 10,
    cost: 100,
    distribution: { common: 7, uncommon: 2, rare: 1 },
    legendaryChance: 0.1
  },
  expansion_booster: {
    cards: 15,
    cost: 150,
    distribution: { common: 9, uncommon: 4, rare: 2 },
    legendaryChance: 0.15
  }
}
```

## Common Development Tasks

### Adding a New Screen
1. Create `/src/screens/NewScreen.tsx`
2. Add route type to `RootStackParamList` in `/src/navigation/index.tsx`
3. Register screen in Stack or Tab navigator
4. Import and use `useNavigation<NavigationProp<RootStackParamList>>()`

### Adding a New Store
1. Define interface in `/src/store/index.ts`
2. Create store with `create<Interface>()(persist(...))`
3. Add storage key to `STORAGE_KEYS` in `/src/constants/index.ts`
4. Export store hook (e.g., `useNewStore`)

### Modifying Card Data
- Edit `/src/data/mockCards.ts`
- Follow existing `Card` interface structure
- Maintain 50 cards for proper completion percentage calculations

### Changing Theme Colors
1. Edit `COLORS` object in `/src/constants/index.ts`
2. Update `darkTheme` or `lightTheme` configurations
3. Restart app (`expo start --clear`) to see changes

### Adding Navigation Routes
1. Update `RootStackParamList` type with new route params
2. Add screen to Stack navigator in `/src/navigation/index.tsx`
3. Use `navigation.navigate('ScreenName', params)` to navigate

## Known Limitations (MVP Phase)

- **Card Images**: Using placeholder.com URLs (need real card art)
- **Import/Export**: Logs to console only (no file system access yet)
- **Cloud Sync**: Not implemented (local AsyncStorage only)
- **API Integration**: No Riot Games API connection (using mock data)
- **Testing**: Manual testing only (no unit/integration tests)
- **Pack History**: Limited to last 100 packs to prevent storage bloat

## Planned Features (Phase 2-3)

1. **Lottie Animations**: Enhanced pack opening experience
2. **Cloud Sync**: Firebase or Supabase for cross-device sync
3. **Riot API**: Official card data integration (requires API key)
4. **Price Tracking**: TCGPlayer/Cardmarket API for card values
5. **Trading System**: Prepare for user-to-user trading
6. **Extended Stats**: Win rates, deck performance tracking
7. **Push Notifications**: New card releases, event reminders

## Debugging Tips

### Common Issues

**AsyncStorage not persisting:**
- Check `STORAGE_KEYS` are unique and correctly prefixed with `@`
- Verify `createJSONStorage(() => AsyncStorage)` is properly configured
- Use Expo DevTools to inspect AsyncStorage: `npx expo start --dev-client`

**Navigation types not working:**
- Ensure `RootStackParamList` is properly exported from `/src/navigation/index.tsx`
- Use `NavigationProp<RootStackParamList>` for `useNavigation()` typing

**Theme not applying:**
- Verify `PaperProvider theme={theme}` wraps entire app in `App.tsx`
- Use `useTheme()` hook from `react-native-paper` in components
- Check that colors are accessed via `theme.colors.propertyName`

**Haptics not working:**
- iOS: Works on physical devices only (not simulator)
- Android: May require permissions in `app.json`
- Check settings: `settings.hapticEnabled` must be `true`

### Performance Optimization Notes

- **Pack History**: Auto-slices to 100 entries to prevent storage bloat
- **Collection Map**: Using `Map<string, CollectionEntry>` for O(1) lookups
- **Memoization**: Not heavily implemented in MVP (add in Phase 2)
- **Image Loading**: Consider adding image caching library for production

## Legal & Attribution

**Bundle IDs:**
- iOS: `com.yourcompany.riftboundtcg`
- Android: `com.yourcompany.riftboundtcg`

**Disclaimer:**
"This app is not affiliated with, endorsed, sponsored, or approved by Riot Games. Riftbound and all associated properties are trademarks or registered trademarks of Riot Games, Inc."

## Quick Reference

### TypeScript Imports
```typescript
// Navigation
import { useNavigation, NavigationProp } from '@react-navigation/native';

// Stores
import { useCollectionStore, useDeckStore, usePackStore } from '../store';

// Types
import { Card, Deck, CollectionEntry } from '../types';

// Constants
import { COLORS, PACK_CONFIG, LAYOUT } from '../constants';

// Haptics
import * as Haptics from 'expo-haptics';

// Theme
import { useTheme } from 'react-native-paper';
```

### Useful Hooks
```typescript
const theme = useTheme();
const navigation = useNavigation<NavigationProp<RootStackParamList>>();
const { collection, addToCollection } = useCollectionStore();
const { settings, updateSettings } = useSettingsStore();
const { decks, createDeck } = useDeckStore();
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ExQbit)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ExQbit)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
