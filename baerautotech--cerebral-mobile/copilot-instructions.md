## cerebral-mobile

> React Native specific TypeScript patterns for components, hooks, and context.

# ⚛️ React Native TypeScript Patterns

**Extends the general TypeScript strictness rules with React Native-specific requirements.**

## **Component Typing (MANDATORY)**

### **1. Functional Component Pattern**
```typescript
// ✅ PREFERRED - Explicit Props interface
import { View, Text, ViewStyle, StyleProp } from 'react-native';
import { User } from '../types'; // Assuming a User type exists

interface UserCardProps {
  user: User;
  onPress?: (user: User) => void;
  style?: StyleProp<ViewStyle>;
  children?: React.ReactNode;
}

// ✅ Arrow function with explicit props typing
const UserCard = ({ user, onPress, style, children }: UserCardProps) => {
  return (
    <View style={style}>
      <Text>{user.name}</Text>
      {children}
    </View>
  );
};
```

### **2. Event Handler Typing (MANDATORY)**
```typescript
import { GestureResponderEvent, Pressable } from 'react-native';

// ✅ CORRECT - Use GestureResponderEvent for press events
interface ButtonProps {
  onPress: (event: GestureResponderEvent) => void;
}

const MyButton = ({ onPress }: ButtonProps) => {
  return <Pressable onPress={onPress}><Text>Submit</Text></Pressable>;
};
```

## **React Hooks Typing (MANDATORY)**

### **1. useState Patterns**
```typescript
// ✅ CORRECT - Explicit typing for complex states
const [user, setUser] = useState<User | null>(null);
const [users, setUsers] = useState<User[]>([]);
const [loading, setLoading] = useState<boolean>(false);
```

### **2. useRef Patterns**
```typescript
import { TextInput } from 'react-native';

// ✅ CORRECT - React Native element refs
const inputRef = useRef<TextInput>(null);

// ✅ USAGE with null checking
const focusInput = (): void => {
  inputRef.current?.focus();
};
```

## **Custom Hooks (MANDATORY)**

### **1. Custom Hook Typing**
```typescript
// ✅ CORRECT - Explicit parameter and return types for a custom hook
interface UseApiReturn<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

function useApi<T>(url: string): UseApiReturn<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch(url);
        const result = await response.json();
        setData(result);
      } catch (e) {
        setError('Failed to fetch data');
      } finally {
        setLoading(false);
      }
    };
    fetchData();
  }, [url]);

  return { data, loading, error };
}
```

## **Context API Typing (MANDATORY)**

### **1. Context Value Typing**
```typescript
// ✅ CORRECT - Context interface
interface AuthContextValue {
  user: User | null;
  login: () => Promise<void>;
  logout: () => void;
  isLoading: boolean;
}

// ✅ CORRECT - Context creation with a default value
const AuthContext = createContext<AuthContextValue | undefined>(undefined);

// ✅ CORRECT - Custom hook for context usage
export const useAuth = (): AuthContextValue => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

## **Styling (MANDATORY)**

### **1. Style Props and StyleSheet**
```typescript
import { View, Text, StyleSheet, StyleProp, ViewStyle, TextStyle } from 'react-native';

interface StyledBoxProps {
  style?: StyleProp<ViewStyle>;
  textStyle?: StyleProp<TextStyle>;
  children: React.ReactNode;
}

const StyledBox = ({ style, textStyle, children }: StyledBoxProps) => {
  return (
    <View style={[styles.defaultBox, style]}>
      <Text style={[styles.defaultText, textStyle]}>{children}</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  defaultBox: {
    padding: 16,
    backgroundColor: 'lightblue',
  },
  defaultText: {
    fontSize: 16,
    color: 'black',
  },
});
```

## **🚫 ANTI-PATTERNS TO AVOID**

```typescript
// ❌ FORBIDDEN - Untyped props
const BadComponent = (props) => {
  return <View><Text>{props.title}</Text></View>;
};

// ❌ FORBIDDEN - Using web-specific types
// import { CSSProperties } from 'react';
// const style: CSSProperties = { backgroundColor: 'red' };

// ❌ FORBIDDEN - Untyped useState
const [data, setData] = useState(null);

// ❌ FORBIDDEN - Missing dependencies in useEffect
useEffect(() => {
  // some logic using userId
}, []); // Missing userId dependency
```
---

**This rule works alongside the general TypeScript strictness rules to ensure React Native components are type-safe, maintainable, and follow enterprise-grade patterns.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/baerautotech) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
