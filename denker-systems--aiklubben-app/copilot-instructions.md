## 13-forms-input-rules

> handleSubmit,

# Forms & Input Rules - React Native iOS

## Activation

- **Mode**: Always On
- **Description**: Form handling patterns for iOS-compliant user input

---

## Input Component Standards

### TextInput iOS Configuration

```typescript
// Standard TextInput with iOS optimizations
<TextInput
  // Required props
  value={value}
  onChangeText={setValue}

  // iOS-specific props
  clearButtonMode="while-editing"    // Show clear button
  keyboardType="default"             // Appropriate keyboard
  returnKeyType="done"               // Return key label
  autoCapitalize="none"              // For email/username
  autoCorrect={false}                // Disable for sensitive fields
  autoComplete="email"               // iOS autofill
  textContentType="emailAddress"     // iOS password manager

  // Accessibility
  accessible={true}
  accessibilityLabel="Email address"
  accessibilityHint="Enter your email"

  // Styling
  style={styles.input}
  placeholderTextColor="#9CA3AF"
/>
```

### Keyboard Types by Field

```typescript
const KEYBOARD_CONFIGS = {
  email: {
    keyboardType: 'email-address' as const,
    autoCapitalize: 'none' as const,
    autoComplete: 'email' as const,
    textContentType: 'emailAddress' as const,
  },
  password: {
    secureTextEntry: true,
    autoCapitalize: 'none' as const,
    autoComplete: 'password' as const,
    textContentType: 'password' as const,
  },
  phone: {
    keyboardType: 'phone-pad' as const,
    autoComplete: 'tel' as const,
    textContentType: 'telephoneNumber' as const,
  },
  number: {
    keyboardType: 'numeric' as const,
  },
  decimal: {
    keyboardType: 'decimal-pad' as const,
  },
  url: {
    keyboardType: 'url' as const,
    autoCapitalize: 'none' as const,
    autoCorrect: false,
  },
  name: {
    autoCapitalize: 'words' as const,
    autoComplete: 'name' as const,
    textContentType: 'name' as const,
  },
};
```

---

## Keyboard Avoiding

### KeyboardAvoidingView Setup

```typescript
import {
  KeyboardAvoidingView,
  Platform,
  ScrollView
} from 'react-native';
import { useSafeAreaInsets } from 'react-native-safe-area-context';

const FormScreen = () => {
  const insets = useSafeAreaInsets();

  return (
    <KeyboardAvoidingView
      style={{ flex: 1 }}
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
      keyboardVerticalOffset={Platform.OS === 'ios' ? 0 : 20}
    >
      <ScrollView
        contentContainerStyle={{
          flexGrow: 1,
          paddingBottom: insets.bottom + 20,
        }}
        keyboardShouldPersistTaps="handled"
        showsVerticalScrollIndicator={false}
      >
        <FormContent />
      </ScrollView>
    </KeyboardAvoidingView>
  );
};
```

### Keyboard Dismiss Patterns

```typescript
import { Keyboard, TouchableWithoutFeedback } from 'react-native';

// Dismiss on tap outside
const DismissKeyboardView = ({ children }: { children: ReactNode }) => (
  <TouchableWithoutFeedback onPress={Keyboard.dismiss}>
    <View style={{ flex: 1 }}>
      {children}
    </View>
  </TouchableWithoutFeedback>
);

// Dismiss on scroll
<ScrollView
  keyboardDismissMode="on-drag"
  keyboardShouldPersistTaps="handled"
>
```

---

## Form State Management

### Form Hook Pattern

```typescript
// hooks/useForm.ts
interface FormConfig<T> {
  initialValues: T;
  validationSchema?: z.ZodSchema<T>;
  onSubmit: (values: T) => Promise<void> | void;
}

export const useForm = <T extends Record<string, unknown>>({
  initialValues,
  validationSchema,
  onSubmit,
}: FormConfig<T>) => {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  const [touched, setTouched] = useState<Partial<Record<keyof T, boolean>>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [submitError, setSubmitError] = useState<string | null>(null);

  const setValue = useCallback(<K extends keyof T>(field: K, value: T[K]) => {
    setValues((prev) => ({ ...prev, [field]: value }));
    setErrors((prev) => ({ ...prev, [field]: undefined }));
    setSubmitError(null);
  }, []);

  const setFieldTouched = useCallback((field: keyof T) => {
    setTouched((prev) => ({ ...prev, [field]: true }));
  }, []);

  const validate = useCallback((): boolean => {
    if (!validationSchema) return true;

    const result = validationSchema.safeParse(values);

    if (result.success) {
      setErrors({});
      return true;
    }

    const newErrors: Partial<Record<keyof T, string>> = {};
    result.error.errors.forEach((err) => {
      const field = err.path[0] as keyof T;
      if (!newErrors[field]) {
        newErrors[field] = err.message;
      }
    });
    setErrors(newErrors);
    return false;
  }, [values, validationSchema]);

  const handleSubmit = useCallback(async () => {
    // Mark all fields as touched
    const allTouched = Object.keys(initialValues).reduce(
      (acc, key) => ({ ...acc, [key]: true }),
      {} as Record<keyof T, boolean>,
    );
    setTouched(allTouched);

    if (!validate()) return;

    setIsSubmitting(true);
    setSubmitError(null);

    try {
      await onSubmit(values);
    } catch (error) {
      setSubmitError(error instanceof Error ? error.message : 'Ett fel uppstod');
    } finally {
      setIsSubmitting(false);
    }
  }, [values, validate, onSubmit, initialValues]);

  const reset = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    setSubmitError(null);
  }, [initialValues]);

  return {
    values,
    errors,
    touched,
    isSubmitting,
    submitError,
    setValue,
    setFieldTouched,
    handleSubmit,
    reset,
    isValid: Object.keys(errors).length === 0,
  };
};
```

---

## Input Validation

### Real-time Validation

```typescript
// Validate on blur
const EmailInput = () => {
  const { values, errors, touched, setValue, setFieldTouched } = useForm();

  return (
    <FormField
      label="E-post"
      value={values.email}
      onChangeText={(text) => setValue('email', text)}
      onBlur={() => setFieldTouched('email')}
      error={touched.email ? errors.email : undefined}
      keyboardType="email-address"
      autoCapitalize="none"
    />
  );
};
```

### Validation Rules

```typescript
// lib/validation/rules.ts
import { z } from 'zod';

export const emailRule = z.string().min(1, 'E-post krävs').email('Ogiltig e-postadress');

export const passwordRule = z
  .string()
  .min(1, 'Lösenord krävs')
  .min(8, 'Minst 8 tecken')
  .regex(/[A-Z]/, 'Minst en stor bokstav')
  .regex(/[0-9]/, 'Minst en siffra');

export const requiredString = (fieldName: string) => z.string().min(1, `${fieldName} krävs`);

export const optionalString = z.string().optional();

// Composite schemas
export const loginSchema = z.object({
  email: emailRule,
  password: z.string().min(1, 'Lösenord krävs'),
});

export const registerSchema = z
  .object({
    name: requiredString('Namn'),
    email: emailRule,
    password: passwordRule,
    confirmPassword: z.string(),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: 'Lösenorden matchar inte',
    path: ['confirmPassword'],
  });
```

---

## Form Field Component

### Reusable FormField

```typescript
// components/ui/FormField.tsx
interface FormFieldProps {
  label: string;
  value: string;
  onChangeText: (text: string) => void;
  onBlur?: () => void;
  error?: string;
  placeholder?: string;
  secureTextEntry?: boolean;
  keyboardType?: KeyboardTypeOptions;
  autoCapitalize?: 'none' | 'sentences' | 'words' | 'characters';
  multiline?: boolean;
  numberOfLines?: number;
  maxLength?: number;
  disabled?: boolean;
  rightElement?: ReactNode;
}

export const FormField: React.FC<FormFieldProps> = ({
  label,
  value,
  onChangeText,
  onBlur,
  error,
  placeholder,
  secureTextEntry,
  keyboardType,
  autoCapitalize,
  multiline,
  numberOfLines,
  maxLength,
  disabled,
  rightElement,
}) => {
  const [isFocused, setIsFocused] = useState(false);
  const hasError = !!error;

  return (
    <View style={styles.container}>
      <Text style={styles.label}>{label}</Text>

      <View style={[
        styles.inputContainer,
        isFocused && styles.inputFocused,
        hasError && styles.inputError,
        disabled && styles.inputDisabled,
      ]}>
        <TextInput
          style={[styles.input, multiline && styles.multilineInput]}
          value={value}
          onChangeText={onChangeText}
          onFocus={() => setIsFocused(true)}
          onBlur={() => {
            setIsFocused(false);
            onBlur?.();
          }}
          placeholder={placeholder}
          placeholderTextColor="#6B7280"
          secureTextEntry={secureTextEntry}
          keyboardType={keyboardType}
          autoCapitalize={autoCapitalize}
          multiline={multiline}
          numberOfLines={numberOfLines}
          maxLength={maxLength}
          editable={!disabled}

          // Accessibility
          accessible={true}
          accessibilityLabel={label}
          accessibilityState={{ disabled }}
          accessibilityHint={error || placeholder}
        />

        {rightElement && (
          <View style={styles.rightElement}>
            {rightElement}
          </View>
        )}
      </View>

      {hasError && (
        <View style={styles.errorContainer}>
          <AlertCircle size={14} color="#EF4444" />
          <Text style={styles.errorText}>{error}</Text>
        </View>
      )}

      {maxLength && (
        <Text style={styles.charCount}>
          {value.length}/{maxLength}
        </Text>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    marginBottom: 16,
  },
  label: {
    fontSize: 14,
    fontWeight: '600',
    color: '#F9FAFB',
    marginBottom: 8,
  },
  inputContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    backgroundColor: '#1A1625',
    borderRadius: 12,
    borderWidth: 1,
    borderColor: '#374151',
  },
  inputFocused: {
    borderColor: '#8B5CF6',
  },
  inputError: {
    borderColor: '#EF4444',
  },
  inputDisabled: {
    opacity: 0.5,
  },
  input: {
    flex: 1,
    height: 48,
    paddingHorizontal: 16,
    fontSize: 16,
    color: '#F9FAFB',
  },
  multilineInput: {
    height: 100,
    paddingTop: 12,
    textAlignVertical: 'top',
  },
  rightElement: {
    paddingRight: 12,
  },
  errorContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    marginTop: 6,
    gap: 4,
  },
  errorText: {
    fontSize: 13,
    color: '#EF4444',
  },
  charCount: {
    fontSize: 12,
    color: '#6B7280',
    textAlign: 'right',
    marginTop: 4,
  },
});
```

---

## Submit Button State

### Submit Button Pattern

```typescript
// Form submit button with proper states
interface SubmitButtonProps {
  onPress: () => void;
  isSubmitting: boolean;
  isDisabled: boolean;
  label: string;
}

export const SubmitButton: React.FC<SubmitButtonProps> = ({
  onPress,
  isSubmitting,
  isDisabled,
  label,
}) => (
  <Pressable
    onPress={onPress}
    disabled={isSubmitting || isDisabled}
    style={({ pressed }) => [
      styles.button,
      (isSubmitting || isDisabled) && styles.buttonDisabled,
      pressed && !isSubmitting && !isDisabled && styles.buttonPressed,
    ]}
    accessibilityRole="button"
    accessibilityLabel={isSubmitting ? 'Skickar...' : label}
    accessibilityState={{ disabled: isSubmitting || isDisabled }}
  >
    {isSubmitting ? (
      <ActivityIndicator color="#FFFFFF" size="small" />
    ) : (
      <Text style={styles.buttonText}>{label}</Text>
    )}
  </Pressable>
);
```

---

## Forbidden Form Practices

1. **NEVER** use uncontrolled inputs in React Native
2. **NEVER** validate only on submit (use real-time + submit)
3. **NEVER** show validation errors before user interaction
4. **NEVER** forget keyboard avoiding behavior
5. **NEVER** use keyboardShouldPersistTaps="never" for forms
6. **NEVER** omit accessibility labels on form fields
7. **NEVER** use default keyboard type for specialized inputs
8. **NEVER** forget to handle keyboard dismiss gestures

---
> Source: [denker-systems/aiklubben-app](https://github.com/denker-systems/aiklubben-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
