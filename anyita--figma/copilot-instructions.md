## figma

> This is an Expo-based React Native application with TypeScript support, utilizing:

# AGENTS.md - React Native Development Specification

## Project Overview

This is an Expo-based React Native application with TypeScript support, utilizing:

- **React Native**: 0.79.6
- **React**: 19.0.0
- **Expo**: ~53.0.22
- **TypeScript**: ~5.8.3
- **Expo Router**: File-based routing
- **Theme System**: Light/Dark mode support

## Directory Structure

```
/
├── app/                    # File-based routing (Expo Router)
│   ├── (tabs)/            # Tab-based screens
│   ├── _layout.tsx        # Root layout
│   └── +not-found.tsx     # 404 page
├── components/            # Reusable components
│   ├── ui/               # Low-level UI primitives
│   └── [ComponentName].tsx
├── constants/            # App constants and configuration
├── hooks/               # Custom React hooks
├── assets/             # Static assets (images, fonts)
└── scripts/           # Build and utility scripts
```

## Component Architecture

### 1. Component Classification

**Theme Components** (Themed*)
- Components that support light/dark themes
- Example: `ThemedText`, `ThemedView`
- Must extend base component props
- Must support `lightColor` and `darkColor` props

**UI Primitives** (components/ui/)
- Platform-specific implementations allowed
- File naming: `ComponentName.tsx` + `ComponentName.ios.tsx`
- Low-level, reusable building blocks

**Feature Components**
- Business logic components
- Should compose UI primitives and theme components

### 2. Component Structure Template

```typescript
import React from 'react';
import { StyleSheet, ViewStyle, TextStyle } from 'react-native';
import { useThemeColor } from '@/hooks/useThemeColor';

// Props interface with clear typing
export interface ComponentNameProps {
  // Required props first
  title: string;

  // Optional props with defaults
  variant?: 'primary' | 'secondary';
  disabled?: boolean;

  // Style props
  style?: ViewStyle;

  // Theme support for themed components
  lightColor?: string;
  darkColor?: string;

  // Event handlers
  onPress?: () => void;

  // Children if applicable
  children?: React.ReactNode;
}

export function ComponentName({
  title,
  variant = 'primary',
  disabled = false,
  style,
  lightColor,
  darkColor,
  onPress,
  children,
}: ComponentNameProps) {
  const backgroundColor = useThemeColor(
    { light: lightColor, dark: darkColor },
    'background'
  );

  return (
    // Component JSX
  );
}

const styles = StyleSheet.create({
  container: {
    // Styles here
  },
});
```

## Coding Standards

### 1. TypeScript Guidelines

**Required Configurations:**
- Strict mode enabled
- Explicit return types for complex functions
- Interface over type for object definitions
- Props interfaces exported

```typescript
// ✅ Good
export interface UserProfileProps {
  user: User;
  onUpdate: (user: User) => Promise<void>;
}

// ❌ Avoid
export type UserProfileProps = {
  user: any;
  onUpdate: Function;
}
```

**Generic Typing:**
```typescript
// ✅ Generic components
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string;
}

export function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  // Implementation
}
```

### 2. File Naming Conventions

- **Components**: PascalCase (`UserProfile.tsx`)
- **Hooks**: camelCase with 'use' prefix (`useUserData.ts`)
- **Utils**: camelCase (`dateHelpers.ts`)
- **Constants**: PascalCase (`Colors.ts`, `Config.ts`)
- **Types**: PascalCase (`UserTypes.ts`)

### 3. Import Organization

```typescript
// 1. React imports
import React from 'react';
import { useState, useEffect } from 'react';

// 2. React Native imports
import { View, Text, StyleSheet } from 'react-native';

// 3. Third-party library imports
import { useRouter } from 'expo-router';

// 4. Local imports (using path aliases)
import { useThemeColor } from '@/hooks/useThemeColor';
import { Button } from '@/components/ui/Button';
import { Colors } from '@/constants/Colors';

// 5. Type-only imports last
import type { User } from '@/types/UserTypes';
```

### 4. Function and Variable Naming

- **Functions**: camelCase (`getUserProfile`)
- **Components**: PascalCase (`UserProfile`)
- **Constants**: SCREAMING_SNAKE_CASE (`API_ENDPOINTS`)
- **Boolean variables**: Prefixed with `is`, `has`, `can`, `should` (`isVisible`, `hasPermission`)

## Styling Guidelines

### 1. StyleSheet Usage

**Always use StyleSheet.create:**
```typescript
// ✅ Good
const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#ffffff',
  },
});

// ❌ Avoid inline styles for static styles
<View style={{ flex: 1, backgroundColor: '#ffffff' }} />
```

### 2. Theme Integration

**Use theme hooks:**
```typescript
const backgroundColor = useThemeColor(
  { light: Colors.light.background, dark: Colors.dark.background },
  'background'
);
```

**Conditional styling:**
```typescript
const dynamicStyles = StyleSheet.create({
  button: {
    backgroundColor: disabled ? Colors.light.disabled : Colors.light.primary,
    opacity: disabled ? 0.6 : 1,
  },
});
```

### 3. Responsive Design

```typescript
import { Dimensions } from 'react-native';

const { width, height } = Dimensions.get('window');

const styles = StyleSheet.create({
  container: {
    width: width * 0.9,
    maxWidth: 400, // Max width for tablets
  },
});
```

## Component Reusability Guidelines

### 1. Composition over Props Drilling

```typescript
// ✅ Good - Composable
<Card>
  <CardHeader title="User Profile" />
  <CardContent>
    <UserAvatar src={user.avatar} />
    <UserInfo name={user.name} email={user.email} />
  </CardContent>
  <CardActions>
    <Button title="Edit" onPress={handleEdit} />
  </CardActions>
</Card>

// ❌ Avoid - Props drilling
<UserCard
  title="User Profile"
  avatar={user.avatar}
  name={user.name}
  email={user.email}
  showEditButton={true}
  onEdit={handleEdit}
/>
```

### 2. Higher-Order Components (HOCs)

```typescript
// Theme HOC
export function withTheme<P extends object>(
  Component: React.ComponentType<P>
) {
  return function ThemedComponent(props: P) {
    const theme = useThemeColor();
    return <Component {...props} theme={theme} />;
  };
}
```

### 3. Custom Hooks for Logic Reuse

```typescript
// ✅ Extract complex logic into hooks
export function useUserProfile(userId: string) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    // Fetch user logic
  }, [userId]);

  const updateUser = useCallback(async (updates: Partial<User>) => {
    // Update logic
  }, []);

  return { user, loading, error, updateUser };
}
```

## Performance Optimization

### 1. React.memo for Pure Components

```typescript
export const ExpensiveComponent = React.memo<ExpensiveComponentProps>(
  ({ data, onSelect }) => {
    return (
      // Component implementation
    );
  }
);
```

### 2. useCallback for Event Handlers

```typescript
const handlePress = useCallback((item: Item) => {
  navigation.navigate('Details', { itemId: item.id });
}, [navigation]);
```

### 3. useMemo for Expensive Calculations

```typescript
const filteredItems = useMemo(() => {
  return items.filter(item => item.category === selectedCategory);
}, [items, selectedCategory]);
```

### 4. Image Optimization

```typescript
import { Image } from 'expo-image';

// ✅ Use Expo Image with optimization
<Image
  source={{ uri: imageUrl }}
  style={styles.image}
  contentFit="cover"
  transition={200}
  cachePolicy="memory-disk"
/>
```

## Platform-Specific Development

### 1. Platform-Specific Files

```
components/ui/
├── TabBarBackground.tsx      # Default implementation
├── TabBarBackground.ios.tsx  # iOS-specific
└── TabBarBackground.android.tsx # Android-specific
```

### 2. Platform Checks

```typescript
import { Platform } from 'react-native';

const styles = StyleSheet.create({
  container: {
    paddingTop: Platform.OS === 'ios' ? 44 : 24,
    ...Platform.select({
      ios: {
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.25,
        shadowRadius: 3.84,
      },
      android: {
        elevation: 5,
      },
    }),
  },
});
```

## Navigation Guidelines

### 1. Expo Router File Structure

```typescript
// app/_layout.tsx - Root layout
export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
      <Stack.Screen name="modal" options={{ presentation: 'modal' }} />
    </Stack>
  );
}

// app/(tabs)/_layout.tsx - Tab layout
export default function TabLayout() {
  return (
    <Tabs>
      <Tabs.Screen
        name="index"
        options={{
          title: 'Home',
          tabBarIcon: ({ color }) => <TabBarIcon name="home" color={color} />,
        }}
      />
    </Tabs>
  );
}
```

### 2. Type-Safe Navigation

```typescript
// types/navigation.ts
export type RootStackParamList = {
  Home: undefined;
  Profile: { userId: string };
  Settings: undefined;
};

// Usage
import { useRouter } from 'expo-router';

const router = useRouter();
router.push({
  pathname: '/profile/[id]',
  params: { id: userId }
});
```

## State Management

### 1. Local State with useState

```typescript
const [formData, setFormData] = useState<FormData>({
  name: '',
  email: '',
});

const updateFormField = useCallback((field: keyof FormData, value: string) => {
  setFormData(prev => ({ ...prev, [field]: value }));
}, []);
```

### 2. Complex State with useReducer

```typescript
interface State {
  loading: boolean;
  data: User[];
  error: string | null;
}

type Action =
  | { type: 'FETCH_START' }
  | { type: 'FETCH_SUCCESS'; payload: User[] }
  | { type: 'FETCH_ERROR'; payload: string };

const reducer = (state: State, action: Action): State => {
  switch (action.type) {
    case 'FETCH_START':
      return { ...state, loading: true, error: null };
    // ... other cases
    default:
      return state;
  }
};
```

## Testing Guidelines

### 1. Component Testing Structure

```typescript
// __tests__/UserProfile.test.tsx
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import { UserProfile } from '../UserProfile';

describe('UserProfile', () => {
  const mockUser = {
    id: '1',
    name: 'John Doe',
    email: 'john@example.com',
  };

  it('renders user information correctly', () => {
    const { getByText } = render(<UserProfile user={mockUser} />);

    expect(getByText('John Doe')).toBeTruthy();
    expect(getByText('john@example.com')).toBeTruthy();
  });

  it('calls onEdit when edit button is pressed', () => {
    const mockOnEdit = jest.fn();
    const { getByText } = render(
      <UserProfile user={mockUser} onEdit={mockOnEdit} />
    );

    fireEvent.press(getByText('Edit'));
    expect(mockOnEdit).toHaveBeenCalledWith(mockUser);
  });
});
```

### 2. Hook Testing

```typescript
import { renderHook, act } from '@testing-library/react-hooks';
import { useCounter } from '../useCounter';

describe('useCounter', () => {
  it('should increment counter', () => {
    const { result } = renderHook(() => useCounter(0));

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });
});
```

## Error Handling

### 1. Error Boundaries

```typescript
interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends React.Component<
  React.PropsWithChildren<{}>,
  ErrorBoundaryState
> {
  constructor(props: React.PropsWithChildren<{}>) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <ErrorFallback error={this.state.error} />;
    }

    return this.props.children;
  }
}
```

### 2. Async Error Handling

```typescript
export function useAsyncOperation<T>() {
  const [state, setState] = useState<{
    data: T | null;
    loading: boolean;
    error: string | null;
  }>({
    data: null,
    loading: false,
    error: null,
  });

  const execute = useCallback(async (operation: () => Promise<T>) => {
    setState(prev => ({ ...prev, loading: true, error: null }));

    try {
      const result = await operation();
      setState({ data: result, loading: false, error: null });
      return result;
    } catch (error) {
      const message = error instanceof Error ? error.message : 'Unknown error';
      setState({ data: null, loading: false, error: message });
      throw error;
    }
  }, []);

  return { ...state, execute };
}
```

## Development Workflow

### 1. Pre-commit Checklist

- [ ] TypeScript compilation passes
- [ ] ESLint rules pass (`npm run lint`)
- [ ] Components are properly typed
- [ ] Theme compatibility verified
- [ ] Platform-specific implementations tested
- [ ] Performance implications considered

### 2. Code Review Guidelines

**Required Reviews For:**
- New components in `/components`
- Changes to theme system
- Navigation structure modifications
- Performance-critical code
- Platform-specific implementations

**Review Checklist:**
- [ ] Proper TypeScript typing
- [ ] Theme system integration
- [ ] Component reusability
- [ ] Performance considerations
- [ ] Accessibility compliance
- [ ] Error handling implementation

### 3. Component Documentation

```typescript
/**
 * UserCard displays user profile information with customizable actions
 *
 * @param user - User object containing profile data
 * @param variant - Visual style variant ('compact' | 'expanded')
 * @param showActions - Whether to display action buttons
 * @param onEdit - Callback fired when edit button is pressed
 *
 * @example
 * <UserCard
 *   user={currentUser}
 *   variant="expanded"
 *   showActions={true}
 *   onEdit={handleEditUser}
 * />
 */
export function UserCard({
  user,
  variant = 'compact',
  showActions = false,
  onEdit,
}: UserCardProps) {
  // Implementation
}
```

## Code Documentation and Comments Guidelines

### 1. Component Documentation Standards

**File Header Comments:**
```typescript
/**
 * UserProfile.tsx
 *
 * A comprehensive user profile component that displays user information
 * with theme support and interactive editing capabilities.
 *
 * Features:
 * - Theme-aware styling (light/dark mode)
 * - Platform-adaptive layout
 * - Accessibility compliance
 * - Loading states and error handling
 *
 * @author Team Name
 * @created 2024-09-07
 * @lastModified 2024-09-07
 */
```

**Component Function Comments:**
```typescript
/**
 * UserProfile - Main user profile display component
 *
 * Renders user avatar, personal information, and action buttons.
 * Automatically adapts to current theme and handles loading/error states.
 *
 * @param user - User object containing profile data
 * @param editable - Whether profile can be edited (default: false)
 * @param onEdit - Callback fired when edit button is pressed
 * @param onDelete - Optional callback for delete action
 * @param style - Additional styling to apply to container
 *
 * @returns JSX.Element - The rendered user profile component
 *
 * @example
 * ```tsx
 * <UserProfile
 *   user={currentUser}
 *   editable={true}
 *   onEdit={handleEditProfile}
 *   style={{ marginTop: 20 }}
 * />
 * ```
 */
export function UserProfile({
  user,
  editable = false,
  onEdit,
  onDelete,
  style,
}: UserProfileProps) {
  // Get theme colors for consistent styling across light/dark modes
  const backgroundColor = useThemeColor(
    { light: Colors.light.background, dark: Colors.dark.background },
    'background'
  );

  // Loading state management for async operations
  const [isLoading, setIsLoading] = useState(false);

  /**
   * Handles the edit button press event
   * Validates user permissions and triggers edit callback
   */
  const handleEditPress = useCallback(() => {
    // Guard clause: Check if editing is allowed
    if (!editable || !onEdit) {
      console.warn('Edit functionality not available');
      return;
    }

    // Set loading state to prevent multiple rapid clicks
    setIsLoading(true);

    try {
      // Execute edit callback with current user data
      onEdit(user);
    } catch (error) {
      console.error('Error during edit:', error);
    } finally {
      // Always reset loading state
      setIsLoading(false);
    }
  }, [editable, onEdit, user]);

  return (
    <ThemedView style={[styles.container, { backgroundColor }, style]}>
      {/* User Avatar Section */}
      <View style={styles.avatarContainer}>
        {/* Display user avatar with fallback to default */}
        <Image
          source={{ uri: user.avatar || DEFAULT_AVATAR_URL }}
          style={styles.avatar}
          // Accessibility: Describe the image for screen readers
          accessibilityLabel={`Profile photo of ${user.name}`}
        />
      </View>

      {/* User Information Section */}
      <View style={styles.infoContainer}>
        {/* User's display name */}
        <ThemedText
          type="title"
          style={styles.userName}
          // Accessibility: Mark as heading for better navigation
          accessibilityRole="header"
        >
          {user.name}
        </ThemedText>

        {/* User's email address */}
        <ThemedText
          type="subtitle"
          style={styles.userEmail}
          // Allow text selection for copying email
          selectable={true}
        >
          {user.email}
        </ThemedText>
      </View>

      {/* Action Buttons Section - Only show if editable */}
      {editable && (
        <View style={styles.actionsContainer}>
          <TouchableOpacity
            style={[
              styles.editButton,
              // Disable button styling when loading
              isLoading && styles.disabledButton
            ]}
            onPress={handleEditPress}
            // Prevent interaction during loading
            disabled={isLoading}
            // Accessibility attributes for screen readers
            accessible={true}
            accessibilityRole="button"
            accessibilityLabel="Edit user profile"
            accessibilityHint="Opens the profile editing screen"
            // Show loading state in accessibility label
            accessibilityState={{ disabled: isLoading }}
          >
            <ThemedText style={styles.buttonText}>
              {/* Show different text based on loading state */}
              {isLoading ? 'Loading...' : 'Edit Profile'}
            </ThemedText>
          </TouchableOpacity>
        </View>
      )}
    </ThemedView>
  );
}
```

### 2. Inline Comments for Complex Logic

**State Management Comments:**
```typescript
export function useUserData(userId: string) {
  // State to track user data with proper typing
  const [userData, setUserData] = useState<User | null>(null);

  // Loading state to show spinners/skeletons
  const [isLoading, setIsLoading] = useState(true);

  // Error state to display error messages to user
  const [error, setError] = useState<string | null>(null);

  /**
   * Fetches user data from API
   * Includes error handling and loading state management
   */
  const fetchUserData = useCallback(async () => {
    // Reset error state before new request
    setError(null);
    setIsLoading(true);

    try {
      // Call API service to get user data
      const response = await userService.getUserById(userId);

      // Validate response structure before setting state
      if (!response || !response.data) {
        throw new Error('Invalid user data received');
      }

      // Update state with fetched user data
      setUserData(response.data);
    } catch (err) {
      // Handle different types of errors appropriately
      const errorMessage = err instanceof Error
        ? err.message
        : 'Failed to load user data';

      console.error('User data fetch failed:', err);
      setError(errorMessage);

      // Clear user data on error
      setUserData(null);
    } finally {
      // Always set loading to false, regardless of success/failure
      setIsLoading(false);
    }
  }, [userId]);

  // Effect to fetch data when userId changes
  useEffect(() => {
    // Only fetch if userId is provided
    if (userId) {
      fetchUserData();
    } else {
      // Clear data and loading state if no userId
      setUserData(null);
      setIsLoading(false);
    }
  }, [userId, fetchUserData]);

  // Return all state values and refetch function
  return {
    userData,
    isLoading,
    error,
    refetch: fetchUserData, // Allow manual refresh
  };
}
```

### 3. Algorithm and Business Logic Comments

**Complex Calculations:**
```typescript
/**
 * Calculates the total price including taxes and discounts
 *
 * Algorithm:
 * 1. Calculate subtotal from all items
 * 2. Apply item-level discounts
 * 3. Apply order-level discounts
 * 4. Calculate tax on discounted amount
 * 5. Add shipping costs
 *
 * @param items - Array of cart items with prices
 * @param discountCode - Optional discount code to apply
 * @param shippingMethod - Selected shipping option
 * @param taxRate - Tax rate as decimal (e.g., 0.08 for 8%)
 * @returns Detailed price breakdown object
 */
export function calculateOrderTotal(
  items: CartItem[],
  discountCode?: string,
  shippingMethod?: ShippingMethod,
  taxRate: number = 0.08
): OrderTotal {
  // Step 1: Calculate base subtotal from all items
  const subtotal = items.reduce((total, item) => {
    // Multiply quantity by unit price for each item
    return total + (item.quantity * item.price);
  }, 0);

  // Step 2: Apply item-level discounts (member pricing, bulk discounts)
  const itemDiscounts = items.reduce((totalDiscount, item) => {
    // Check if item has bulk pricing (e.g., buy 3 get 1 free)
    if (item.quantity >= item.bulkThreshold) {
      // Calculate discount amount based on bulk rules
      const freeItems = Math.floor(item.quantity / item.bulkThreshold);
      const discountAmount = freeItems * item.price;
      return totalDiscount + discountAmount;
    }
    return totalDiscount;
  }, 0);

  // Step 3: Apply order-level discount code
  let orderDiscount = 0;
  if (discountCode) {
    const discount = validateDiscountCode(discountCode);
    if (discount) {
      // Apply percentage or fixed amount discount
      orderDiscount = discount.type === 'percentage'
        ? (subtotal - itemDiscounts) * (discount.value / 100)
        : discount.value;

      // Ensure discount doesn't exceed order value
      orderDiscount = Math.min(orderDiscount, subtotal - itemDiscounts);
    }
  }

  // Step 4: Calculate tax on the discounted amount
  const taxableAmount = subtotal - itemDiscounts - orderDiscount;
  const taxAmount = Math.max(0, taxableAmount * taxRate);

  // Step 5: Calculate shipping costs based on method
  const shippingCost = shippingMethod ?
    calculateShippingCost(shippingMethod, taxableAmount) : 0;

  // Step 6: Calculate final total
  const finalTotal = taxableAmount + taxAmount + shippingCost;

  // Return detailed breakdown for transparency
  return {
    subtotal,
    itemDiscounts,
    orderDiscount,
    taxAmount,
    shippingCost,
    total: finalTotal,
    // Additional metadata for UI display
    breakdown: {
      beforeTax: taxableAmount,
      afterTax: finalTotal,
      savings: itemDiscounts + orderDiscount,
    }
  };
}
```

### 4. API Integration Comments

```typescript
/**
 * UserService - Handles all user-related API operations
 *
 * Base URL: /api/v1/users
 * Authentication: Bearer token required
 * Rate Limits: 100 requests per minute
 */
export class UserService {
  private baseURL = process.env.EXPO_PUBLIC_API_URL + '/api/v1/users';

  /**
   * Fetches user profile by ID
   *
   * @param userId - Unique user identifier
   * @returns Promise<User> - User profile data
   * @throws {ApiError} - When user not found or server error
   */
  async getUserById(userId: string): Promise<ApiResponse<User>> {
    try {
      // Build request URL with user ID parameter
      const url = `${this.baseURL}/${encodeURIComponent(userId)}`;

      // Get authentication token from secure storage
      const token = await SecureStore.getItemAsync('auth_token');
      if (!token) {
        throw new Error('Authentication required');
      }

      // Make API request with proper headers
      const response = await fetch(url, {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`,
          // Include app version for API compatibility
          'X-App-Version': Constants.expoConfig?.version || '1.0.0',
        },
      });

      // Check if response is successful
      if (!response.ok) {
        // Handle different HTTP status codes appropriately
        switch (response.status) {
          case 401:
            throw new Error('Authentication failed');
          case 404:
            throw new Error('User not found');
          case 429:
            throw new Error('Too many requests');
          default:
            throw new Error(`Server error: ${response.status}`);
        }
      }

      // Parse and validate response data
      const data = await response.json();

      // Ensure response has expected structure
      if (!data || typeof data !== 'object') {
        throw new Error('Invalid response format');
      }

      return data;
    } catch (error) {
      // Log error for debugging (remove in production)
      console.error('getUserById failed:', {
        userId,
        error: error.message,
        timestamp: new Date().toISOString(),
      });

      // Re-throw error for caller to handle
      throw error;
    }
  }
}
```

### 5. Performance Critical Code Comments

```typescript
/**
 * VirtualizedList - High-performance list component for large datasets
 *
 * Performance optimizations applied:
 * - React.memo to prevent unnecessary re-renders
 * - useCallback for stable event handlers
 * - useMemo for expensive calculations
 * - Virtualization for memory efficiency
 */
export const VirtualizedList = React.memo<VirtualizedListProps>(({
  data,
  renderItem,
  onEndReached,
  estimatedItemSize = 50,
}) => {
  // Memoize expensive filtering/sorting operations
  const processedData = useMemo(() => {
    // Apply filters and sorting only when data changes
    return data
      .filter(item => item.visible) // Filter visible items
      .sort((a, b) => a.order - b.order); // Sort by display order
  }, [data]);

  // Stable callback to prevent child re-renders
  const handleItemPress = useCallback((item: ListItem) => {
    // Haptic feedback for better UX
    Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light);

    // Navigate to item detail screen
    navigation.navigate('ItemDetail', { itemId: item.id });
  }, [navigation]);

  // Optimized render function with minimal logic
  const renderListItem = useCallback(({ item, index }: ListRenderItemInfo<ListItem>) => {
    return (
      <ListItemComponent
        key={item.id} // Stable key for React optimization
        item={item}
        index={index}
        onPress={handleItemPress}
        // Pass minimal props to reduce render cost
        isLast={index === processedData.length - 1}
      />
    );
  }, [handleItemPress, processedData.length]);

  return (
    <FlatList
      data={processedData}
      renderItem={renderListItem}
      // Performance optimizations
      removeClippedSubviews={true} // Remove off-screen views from memory
      maxToRenderPerBatch={10} // Limit initial render batch size
      windowSize={10} // Control number of screens to render
      initialNumToRender={20} // Items to render on first load
      getItemLayout={(data, index) => ({
        // Provide item dimensions for better scrolling performance
        length: estimatedItemSize,
        offset: estimatedItemSize * index,
        index,
      })}
      // Stable key extractor
      keyExtractor={(item) => item.id}
      onEndReached={onEndReached}
      onEndReachedThreshold={0.1} // Trigger load more at 10% from end
    />
  );
});
```

### 6. Style Comments

```typescript
const styles = StyleSheet.create({
  // Main container - provides base layout structure
  container: {
    flex: 1, // Fill available space
    backgroundColor: '#ffffff', // Default background (overridden by theme)
    paddingHorizontal: 16, // Standard app margin
  },

  // Header section styling
  header: {
    height: 60, // Fixed header height for consistency
    flexDirection: 'row', // Horizontal layout for title and buttons
    alignItems: 'center', // Vertically center content
    justifyContent: 'space-between', // Space between title and actions
    paddingTop: Platform.OS === 'ios' ? 44 : 24, // Account for status bar
    ...Platform.select({
      // Platform-specific shadow styles
      ios: {
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 1 },
        shadowOpacity: 0.1,
        shadowRadius: 4,
      },
      android: {
        elevation: 4, // Android shadow
      },
    }),
  },

  // Button styling with interactive states
  primaryButton: {
    backgroundColor: '#007AFF', // iOS blue
    paddingHorizontal: 24,
    paddingVertical: 12,
    borderRadius: 8,
    minWidth: 120, // Ensure consistent button size
    alignItems: 'center',
    justifyContent: 'center',
    // Add touch feedback visual cue
    opacity: 1, // Normal state, modified by TouchableOpacity
  },

  // Disabled button state
  disabledButton: {
    backgroundColor: '#E5E5E7', // Gray background
    opacity: 0.6, // Reduced opacity to indicate disabled state
  },

  // Responsive font sizing based on device size
  titleText: {
    fontSize: width > 400 ? 18 : 16, // Larger text on bigger screens
    fontWeight: '600',
    color: '#000000', // Overridden by theme
    lineHeight: 24, // Consistent line spacing
  },
});
```

## Accessibility Guidelines

### 1. Accessibility Props

```typescript
import { AccessibilityRole } from 'react-native';

<TouchableOpacity
  accessible={true}
  accessibilityRole="button"
  accessibilityLabel="Edit user profile"
  accessibilityHint="Opens the user profile editing screen"
  onPress={handleEdit}
>
  <Text>Edit</Text>
</TouchableOpacity>
```

### 2. Screen Reader Support

```typescript
<View accessibilityRole="header">
  <Text accessibilityLevel={1}>Profile Settings</Text>
</View>

<FlatList
  data={items}
  accessibilityLabel="Settings list"
  renderItem={({ item }) => (
    <TouchableOpacity
      accessibilityRole="button"
      accessibilityLabel={`${item.title}, ${item.description}`}
    >
      <SettingsItem {...item} />
    </TouchableOpacity>
  )}
/>
```

## Performance Monitoring

### 1. Bundle Analysis

```bash
# Analyze bundle size
npx expo export --platform web
npx webpack-bundle-analyzer web-build/static/js/*.js
```

### 2. Performance Metrics

```typescript
import { performance } from 'perf_hooks';

// Measure component render time
const ComponentWithMetrics = () => {
  useEffect(() => {
    const startTime = performance.now();

    return () => {
      const endTime = performance.now();
      console.log(`Component rendered in ${endTime - startTime}ms`);
    };
  }, []);

  return <YourComponent />;
};
```

## Conclusion

This specification ensures consistent, maintainable, and scalable React Native development. All team members should:

1. Follow TypeScript strict guidelines
2. Implement proper component composition
3. Maintain theme system compatibility
4. Consider performance implications
5. Write comprehensive tests
6. Document complex components
7. Ensure accessibility compliance

Regular reviews of this specification ensure it stays current with project evolution and React Native ecosystem changes.

---
> Source: [Anyita/figma](https://github.com/Anyita/figma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
