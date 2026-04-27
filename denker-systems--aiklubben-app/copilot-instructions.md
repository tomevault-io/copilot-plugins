## 10-performance-rules

> - **Mode**: Always On

# Performance Rules - React Native iOS Optimization

## Activation

- **Mode**: Always On
- **Description**: Performance optimization patterns for smooth 60fps iOS apps

---

## Rendering Performance

### Component Memoization

```typescript
// Memoize components that receive stable props
// CORRECT: Prevent unnecessary re-renders
export const ExpensiveList = React.memo(({ items, onItemPress }: Props) => {
  return (
    <FlatList
      data={items}
      renderItem={renderItem}
      keyExtractor={item => item.id}
    />
  );
});

// CORRECT: Custom comparison for complex props
export const UserCard = React.memo(
  ({ user, onPress }: Props) => { /* render */ },
  (prevProps, nextProps) => {
    return (
      prevProps.user.id === nextProps.user.id &&
      prevProps.user.updatedAt === nextProps.user.updatedAt
    );
  }
);
```

### useCallback & useMemo

```typescript
// useCallback: Memoize functions passed to child components
const ParentComponent = () => {
  const [count, setCount] = useState(0);

  // CORRECT: Memoized callback
  const handlePress = useCallback(() => {
    setCount(c => c + 1);
  }, []);

  // WRONG: New function every render
  const handlePressBad = () => {
    setCount(c => c + 1);
  };

  return <ChildComponent onPress={handlePress} />;
};

// useMemo: Memoize expensive calculations
const Component = ({ items }: Props) => {
  // CORRECT: Only recalculate when items change
  const sortedItems = useMemo(() => {
    return [...items].sort((a, b) => a.name.localeCompare(b.name));
  }, [items]);

  // CORRECT: Expensive filtering
  const filteredItems = useMemo(() => {
    return items.filter(item => complexFilter(item));
  }, [items]);

  // WRONG: Don't memoize simple operations
  const itemCount = useMemo(() => items.length, [items]); // Overkill
};
```

### When NOT to Memoize

```typescript
// DON'T memoize:
// 1. Primitive props (strings, numbers, booleans)
// 2. Components that always re-render anyway
// 3. Simple calculations
// 4. Root-level components

// WRONG: Unnecessary memoization
const title = useMemo(() => `Hello ${name}`, [name]); // Simple string
const doubled = useMemo(() => count * 2, [count]); // Simple math
```

---

## List Performance

### FlatList Optimization

```typescript
// Optimized FlatList configuration
<FlatList
  data={items}
  renderItem={renderItem}
  keyExtractor={item => item.id}

  // CRITICAL: Define item layout for virtualization
  getItemLayout={(data, index) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index,
  })}

  // Performance settings
  removeClippedSubviews={true}      // Remove off-screen items
  maxToRenderPerBatch={10}          // Items per batch
  updateCellsBatchingPeriod={50}    // Batch update interval
  windowSize={5}                     // Viewport multiplier
  initialNumToRender={10}           // Initial render count

  // Avoid inline functions
  // WRONG: Creates new function each render
  // renderItem={({ item }) => <Item item={item} />}
/>
```

### Optimized renderItem

```typescript
// Extract renderItem outside component or memoize
const renderItem = useCallback(({ item, index }: ListRenderItemInfo<Item>) => (
  <ItemComponent item={item} index={index} />
), []);

// OR define outside component
const renderItem = ({ item }: ListRenderItemInfo<Item>) => (
  <ItemComponent item={item} />
);

// Item component should be memoized
const ItemComponent = React.memo(({ item }: { item: Item }) => (
  <View style={styles.item}>
    <Text>{item.title}</Text>
  </View>
));
```

### Avoid Nested VirtualizedLists

```typescript
// WRONG: Nested FlatLists cause performance issues
<FlatList
  data={sections}
  renderItem={({ item: section }) => (
    <FlatList // NESTED - BAD!
      data={section.items}
      renderItem={renderItem}
    />
  )}
/>

// CORRECT: Use SectionList for grouped data
<SectionList
  sections={sections}
  renderItem={renderItem}
  renderSectionHeader={renderHeader}
  stickySectionHeadersEnabled={false}
/>

// CORRECT: Flatten data for single FlatList
const flattenedData = sections.flatMap(section => [
  { type: 'header', data: section },
  ...section.items.map(item => ({ type: 'item', data: item })),
]);
```

---

## Image Optimization

### Image Loading

```typescript
// Use expo-image for better performance
import { Image } from 'expo-image';

<Image
  source={{ uri: imageUrl }}
  style={styles.image}
  contentFit="cover"
  transition={200}
  // Caching
  cachePolicy="memory-disk"
  // Placeholder
  placeholder={blurhash}
  placeholderContentFit="cover"
/>

// Preload critical images
import { Image } from 'expo-image';

useEffect(() => {
  Image.prefetch([
    'https://example.com/image1.jpg',
    'https://example.com/image2.jpg',
  ]);
}, []);
```

### Image Dimensions

```typescript
// ALWAYS specify image dimensions
// CORRECT: Explicit dimensions prevent layout shift
<Image
  source={{ uri: imageUrl }}
  style={{ width: 100, height: 100 }}
/>

// WRONG: Missing dimensions
<Image
  source={{ uri: imageUrl }}
  style={{ flex: 1 }} // Layout may jump
/>

// Use aspect ratio with one dimension
<Image
  source={{ uri: imageUrl }}
  style={{ width: '100%', aspectRatio: 16/9 }}
/>
```

---

## Animation Performance

### Use Native Driver

```typescript
// Animated API with native driver
import { Animated } from 'react-native';

const fadeAnim = useRef(new Animated.Value(0)).current;

Animated.timing(fadeAnim, {
  toValue: 1,
  duration: 300,
  useNativeDriver: true, // CRITICAL for performance
}).start();

// Native driver supports:
// - opacity
// - transform (translateX, translateY, scale, rotate)

// Native driver does NOT support:
// - width, height, padding, margin
// - backgroundColor
// - borderRadius
```

### Reanimated 2 for Complex Animations

```typescript
// Use Reanimated for gesture-driven animations
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
} from 'react-native-reanimated';

const Component = () => {
  const offset = useSharedValue(0);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: offset.value }],
  }));

  // Runs on UI thread - 60fps guaranteed
  return <Animated.View style={animatedStyle} />;
};
```

---

## JavaScript Thread Optimization

### Avoid Heavy Operations on JS Thread

```typescript
// WRONG: Heavy computation blocks JS thread
const processLargeArray = (items: Item[]) => {
  return items.map((item) => expensiveTransform(item));
};

// CORRECT: Use InteractionManager
import { InteractionManager } from 'react-native';

const loadData = async () => {
  // Wait for animations to complete
  await InteractionManager.runAfterInteractions();

  // Then do heavy work
  const result = processLargeArray(items);
  setData(result);
};

// CORRECT: Batch updates
import { unstable_batchedUpdates } from 'react-native';

unstable_batchedUpdates(() => {
  setItems(newItems);
  setCount(newCount);
  setLoading(false);
});
```

### Debounce & Throttle

```typescript
// Debounce for search input
import { useMemo } from 'react';
import debounce from 'lodash/debounce';

const SearchInput = () => {
  const [query, setQuery] = useState('');

  const debouncedSearch = useMemo(
    () => debounce((text: string) => {
      performSearch(text);
    }, 300),
    []
  );

  const handleChange = (text: string) => {
    setQuery(text);
    debouncedSearch(text);
  };

  // Cleanup
  useEffect(() => {
    return () => debouncedSearch.cancel();
  }, [debouncedSearch]);

  return <TextInput value={query} onChangeText={handleChange} />;
};
```

---

## Memory Management

### Cleanup Effects

```typescript
// ALWAYS cleanup subscriptions and timers
useEffect(() => {
  const subscription = eventEmitter.addListener('event', handler);

  return () => {
    subscription.remove(); // Cleanup
  };
}, []);

useEffect(() => {
  const timer = setTimeout(() => {
    // Do something
  }, 1000);

  return () => {
    clearTimeout(timer); // Cleanup
  };
}, []);
```

### Avoid Memory Leaks

```typescript
// WRONG: Setting state after unmount
useEffect(() => {
  fetchData().then((data) => {
    setData(data); // May leak if component unmounted
  });
}, []);

// CORRECT: Check if mounted
useEffect(() => {
  let isMounted = true;

  fetchData().then((data) => {
    if (isMounted) {
      setData(data);
    }
  });

  return () => {
    isMounted = false;
  };
}, []);

// CORRECT: Use AbortController
useEffect(() => {
  const controller = new AbortController();

  fetch(url, { signal: controller.signal })
    .then((res) => res.json())
    .then(setData)
    .catch((err) => {
      if (err.name !== 'AbortError') {
        setError(err);
      }
    });

  return () => controller.abort();
}, [url]);
```

---

## Bundle Size Optimization

### Tree Shaking

```typescript
// CORRECT: Named imports for tree shaking
import { map, filter } from 'lodash-es';
import { format } from 'date-fns';

// WRONG: Full library imports
import _ from 'lodash';
import * as dateFns from 'date-fns';
```

### Lazy Loading

```typescript
// Lazy load heavy screens
const HeavyScreen = React.lazy(() => import('./HeavyScreen'));

// With Suspense
<Suspense fallback={<LoadingScreen />}>
  <HeavyScreen />
</Suspense>
```

---

## Profiling Tools

### React DevTools Profiler

```typescript
// Wrap components to profile
import { Profiler } from 'react';

const onRenderCallback = (
  id: string,
  phase: 'mount' | 'update',
  actualDuration: number,
) => {
  console.log(`${id} ${phase}: ${actualDuration}ms`);
};

<Profiler id="MyComponent" onRender={onRenderCallback}>
  <MyComponent />
</Profiler>
```

---

## Forbidden Performance Practices

1. **NEVER** create inline functions in render for callbacks
2. **NEVER** use index as key for dynamic lists
3. **NEVER** nest VirtualizedLists (FlatList inside FlatList)
4. **NEVER** omit useNativeDriver for supported animations
5. **NEVER** forget to cleanup effects (subscriptions, timers)
6. **NEVER** import full libraries when tree-shaking is available
7. **NEVER** set state without checking mount status in async operations
8. **NEVER** skip getItemLayout for FlatLists with fixed item heights

---
> Source: [denker-systems/aiklubben-app](https://github.com/denker-systems/aiklubben-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
