## 06-navigation-rules

> - **Mode**: Always On

# Navigation Rules - React Navigation for iOS

## Activation

- **Mode**: Always On
- **Description**: Navigation patterns and structure for React Native iOS apps

---

## Navigation Structure

### Navigator Hierarchy

```typescript
// App.tsx - Root navigation structure
<NavigationContainer>
  <Stack.Navigator screenOptions={{ headerShown: false }}>
    {isAuthenticated ? (
      <Stack.Screen name="Main" component={MainNavigator} />
    ) : (
      <Stack.Screen name="Auth" component={AuthNavigator} />
    )}
  </Stack.Navigator>
</NavigationContainer>
```

### Tab Navigator Setup

```typescript
// MainNavigator.tsx
const Tab = createBottomTabNavigator();

<Tab.Navigator
  screenOptions={{
    headerShown: false,
    tabBarStyle: {
      backgroundColor: '#0C0A17',
      borderTopColor: '#1F2937',
      height: 49 + insets.bottom, // iOS tab bar height + safe area
      paddingBottom: insets.bottom,
    },
    tabBarActiveTintColor: '#8B5CF6',
    tabBarInactiveTintColor: '#6B7280',
  }}
>
  <Tab.Screen
    name="Home"
    component={HomeScreen}
    options={{
      tabBarIcon: ({ color, size }) => (
        <Home size={size} color={color} />
      ),
    }}
  />
  {/* More tabs */}
</Tab.Navigator>
```

---

## Screen Types

### Root Screens (Tab Bar Visible)

```typescript
// Screens accessible from bottom tab bar
type RootScreens = {
  Home: undefined;
  Courses: undefined;
  Profile: undefined;
  Settings: undefined;
};

// Root screens use isRoot={true} for header
<ScreenLayout isRoot={true} title="Home">
  <HomeContent />
</ScreenLayout>
```

### Sub Screens (Back Button)

```typescript
// Screens navigated to FROM root screens
type SubScreens = {
  CourseDetail: { courseId: string };
  Lesson: { lessonId: string; courseId: string };
  EditProfile: undefined;
};

// Sub screens show back button
<ScreenLayout title="Course Details">
  <CourseDetailContent />
</ScreenLayout>
```

---

## Navigation Patterns

### Type-Safe Navigation

```typescript
// types/navigation.ts
import { NativeStackNavigationProp } from '@react-navigation/native-stack';
import { RouteProp } from '@react-navigation/native';

// Define all routes and their params
export type RootStackParamList = {
  // Auth Stack
  Login: undefined;
  Register: undefined;

  // Main Stack
  Main: undefined;

  // Course Stack
  Courses: undefined;
  CourseDetail: { courseId: string };
  Lesson: { lessonId: string; courseId: string };
};

// Navigation prop type
export type NavigationProp<T extends keyof RootStackParamList> = NativeStackNavigationProp<
  RootStackParamList,
  T
>;

// Route prop type
export type RouteProps<T extends keyof RootStackParamList> = RouteProp<RootStackParamList, T>;
```

### Using Navigation Hook

```typescript
import { useNavigation, useRoute } from '@react-navigation/native';
import type { NavigationProp, RouteProps } from '@/types/navigation';

const CourseDetailScreen = () => {
  const navigation = useNavigation<NavigationProp<'CourseDetail'>>();
  const route = useRoute<RouteProps<'CourseDetail'>>();

  const { courseId } = route.params;

  const handleLessonPress = (lessonId: string) => {
    navigation.navigate('Lesson', { lessonId, courseId });
  };

  const handleBack = () => {
    navigation.goBack();
  };

  return (/* Screen content */);
};
```

---

## Header Configuration

### Custom Header Component

```typescript
// components/layout/Header.tsx
interface HeaderProps {
  title?: string;
  showBack?: boolean;
  showMenu?: boolean;
  rightElement?: React.ReactNode;
}

export const Header: React.FC<HeaderProps> = ({
  title,
  showBack = true,
  showMenu = false,
  rightElement,
}) => {
  const navigation = useNavigation();
  const insets = useSafeAreaInsets();

  return (
    <View style={[styles.header, { paddingTop: insets.top }]}>
      {/* Left: Back or placeholder */}
      <View style={styles.headerLeft}>
        {showBack && (
          <Pressable
            onPress={() => navigation.goBack()}
            hitSlop={{ top: 10, bottom: 10, left: 10, right: 10 }}
            style={styles.backButton}
          >
            <ArrowLeft size={24} color="#F9FAFB" />
          </Pressable>
        )}
      </View>

      {/* Center: Title */}
      <View style={styles.headerCenter}>
        {title && (
          <Text style={styles.headerTitle} numberOfLines={1}>
            {title}
          </Text>
        )}
      </View>

      {/* Right: Menu or custom */}
      <View style={styles.headerRight}>
        {showMenu && (
          <Pressable style={styles.menuButton}>
            <Menu size={24} color="#F9FAFB" />
          </Pressable>
        )}
        {rightElement}
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  header: {
    flexDirection: 'row',
    alignItems: 'center',
    paddingHorizontal: 16,
    paddingBottom: 12,
    backgroundColor: '#0C0A17',
  },
  headerLeft: {
    width: 44,
    alignItems: 'flex-start',
  },
  headerCenter: {
    flex: 1,
    alignItems: 'center',
  },
  headerRight: {
    width: 44,
    alignItems: 'flex-end',
  },
  headerTitle: {
    fontSize: 17,
    fontWeight: '600',
    color: '#F9FAFB',
  },
  backButton: {
    width: 44,
    height: 44,
    justifyContent: 'center',
    alignItems: 'center',
  },
  menuButton: {
    width: 44,
    height: 44,
    justifyContent: 'center',
    alignItems: 'center',
  },
});
```

---

## Screen Transitions

### iOS Standard Transitions

```typescript
// Stack Navigator with iOS transitions
<Stack.Navigator
  screenOptions={{
    headerShown: false,
    // iOS native transitions
    animation: 'slide_from_right',
    gestureEnabled: true,
    gestureDirection: 'horizontal',
  }}
>
```

### Modal Presentations

```typescript
// Present screen as modal
<Stack.Screen
  name="Settings"
  component={SettingsScreen}
  options={{
    presentation: 'modal',
    animation: 'slide_from_bottom',
  }}
/>

// Full-screen modal
<Stack.Screen
  name="Lesson"
  component={LessonScreen}
  options={{
    presentation: 'fullScreenModal',
    animation: 'fade',
  }}
/>
```

---

## Deep Linking

### Configure Deep Links

```typescript
// App.tsx
const linking = {
  prefixes: ['aiklubben://', 'https://aiklubben.se'],
  config: {
    screens: {
      Main: {
        screens: {
          Home: 'home',
          Courses: 'courses',
        },
      },
      CourseDetail: 'course/:courseId',
      Lesson: 'lesson/:lessonId',
    },
  },
};

<NavigationContainer linking={linking}>
  {/* Navigators */}
</NavigationContainer>
```

---

## Navigation State Persistence

### Persist Navigation State

```typescript
import AsyncStorage from '@react-native-async-storage/async-storage';

const PERSISTENCE_KEY = 'NAVIGATION_STATE';

const App = () => {
  const [isReady, setIsReady] = useState(false);
  const [initialState, setInitialState] = useState();

  useEffect(() => {
    const restoreState = async () => {
      try {
        const savedState = await AsyncStorage.getItem(PERSISTENCE_KEY);
        if (savedState) {
          setInitialState(JSON.parse(savedState));
        }
      } finally {
        setIsReady(true);
      }
    };

    restoreState();
  }, []);

  if (!isReady) return <SplashScreen />;

  return (
    <NavigationContainer
      initialState={initialState}
      onStateChange={(state) =>
        AsyncStorage.setItem(PERSISTENCE_KEY, JSON.stringify(state))
      }
    >
      {/* Navigators */}
    </NavigationContainer>
  );
};
```

---

## Navigation Guards

### Auth Guard Pattern

```typescript
// Use in navigation container
const Navigation = () => {
  const { isAuthenticated, isLoading } = useAuth();

  if (isLoading) {
    return <SplashScreen />;
  }

  return (
    <Stack.Navigator screenOptions={{ headerShown: false }}>
      {isAuthenticated ? (
        <>
          <Stack.Screen name="Main" component={MainNavigator} />
          <Stack.Screen name="CourseDetail" component={CourseDetailScreen} />
        </>
      ) : (
        <>
          <Stack.Screen name="Login" component={LoginScreen} />
          <Stack.Screen name="Register" component={RegisterScreen} />
        </>
      )}
    </Stack.Navigator>
  );
};
```

---

## Forbidden Navigation Practices

1. **NEVER** use string-based navigation without type checking
2. **NEVER** navigate from outside React components
3. **NEVER** deep nest navigators more than 3 levels
4. **NEVER** use headerShown: true with custom headers
5. **NEVER** forget to handle hardware back button on Android
6. **NEVER** use navigation.reset() without clear UX reason
7. **NEVER** pass complex objects as route params (use IDs)
8. **NEVER** ignore safe areas in custom headers

---
> Source: [denker-systems/aiklubben-app](https://github.com/denker-systems/aiklubben-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
