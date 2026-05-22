## rawbank-signup

> This is a React TypeScript application with Material-UI integration for a banking signup form. The app uses Supabase for authentication and follows modern React patterns.

# Cursor Rules for Rawbank Signup App

## Project Overview

This is a React TypeScript application with Material-UI integration for a banking signup form. The app uses Supabase for authentication and follows modern React patterns.

## Architecture Principles

### 1. Self-Contained Components

- **PRIORITY**: Always create self-contained, reusable components
- Each component should have its own state management and styling
- Components should be composable and not tightly coupled
- Use TypeScript interfaces for component props
- Export components as default exports with proper naming conventions

```typescript
// Example: Self-contained component structure
interface ComponentProps {
  title: string;
  onSubmit: (data: FormData) => void;
  loading?: boolean;
}

const ComponentName: React.FC<ComponentProps> = ({ title, onSubmit, loading = false }) => {
  // Component logic here
  return (
    // JSX here
  );
};

export default ComponentName;
```

### 2. Custom Hooks for Data Access

- **PRIORITY**: Extract all data fetching and state management into custom hooks
- Create hooks for Supabase operations, form state, and API calls
- Use React Query or SWR for server state management when applicable
- Hooks should be pure functions that return state and actions

```typescript
// Example: Custom hook structure
const useCustomHook = (param: string) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const fetchData = useCallback(async () => {
    // Data fetching logic
  }, [param]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return { data, loading, error, refetch: fetchData };
};
```

### 3. Material-UI Skeleton Loading

- **PRIORITY**: Always use Material-UI Skeleton components for loading states
- Replace loading spinners with skeleton placeholders that match the actual content
- Use appropriate skeleton variants (text, rectangular, circular)
- Implement progressive loading for better UX

```typescript
import { Skeleton } from "@mui/material";

// Example: Skeleton loading implementation
const LoadingComponent = () => (
  <Box>
    <Skeleton variant="text" width="60%" height={40} />
    <Skeleton variant="rectangular" width="100%" height={200} sx={{ mt: 2 }} />
    <Skeleton variant="circular" width={40} height={40} sx={{ mt: 2 }} />
  </Box>
);
```

### 4. Vector Illustrations

- **PRIORITY**: Add vector illustrations when necessary for better UX
- Use SVG icons from Material-UI or custom SVG components
- Illustrations should be responsive and accessible
- Consider using illustrations for empty states, success states, and onboarding

```typescript
// Example: Vector illustration component
const IllustrationIcon: React.FC<{ size?: number; color?: string }> = ({
  size = 64,
  color = "primary.main",
}) => (
  <Box sx={{ width: size, height: size, color }}>
    <svg viewBox="0 0 24 24" fill="currentColor">
      {/* SVG path data */}
    </svg>
  </Box>
);
```

### 5. User Experience for Non-Tech-Savvy Users

- **CRITICAL**: Most users are not tech-savvy - prioritize simplicity and guidance
- **PRIORITY**: Use clear, simple language - avoid technical jargon
- **PRIORITY**: Add helpful icons to ALL input fields using Material-UI InputAdornment
- **PRIORITY**: Include helper text below each field to guide users
- **PRIORITY**: Use vector illustrations at the start of each major section
- **PRIORITY**: Provide clear visual feedback for success, error, and loading states
- **PRIORITY**: Use tooltips for additional context without cluttering the UI
- **PRIORITY**: Implement inline validation with clear, friendly error messages
- **PRIORITY**: Show progress indicators so users know where they are in the process

```typescript
// Example: Input field with icon and helper text
<TextField
  label="Numéro de téléphone"
  placeholder="+243 XXX XXX XXX"
  helperText="Nous utiliserons ce numéro pour vous envoyer des notifications importantes"
  InputProps={{
    startAdornment: (
      <InputAdornment position="start">
        <PhoneIcon color="primary" />
      </InputAdornment>
    ),
  }}
/>

// Example: Section with illustration
<Box sx={{ textAlign: 'center', mb: 3 }}>
  <PersonIcon sx={{ fontSize: 80, color: 'primary.main', mb: 2 }} />
  <Typography variant="h5" gutterBottom>
    Informations Personnelles
  </Typography>
  <Typography variant="body2" color="text.secondary">
    Ces informations nous aident à créer votre compte en toute sécurité
  </Typography>
</Box>
```

#### UX Guidelines for Forms:

- Use large, easy-to-read fonts
- Provide clear labels with icons
- Show example formats in placeholders
- Use helper text to explain why information is needed
- Group related fields with clear headings and illustrations
- Provide immediate feedback on field validation
- Use green checkmarks for valid fields
- Use friendly, non-technical error messages
- Show password strength indicators
- Provide "Show/Hide" toggles for password fields
- Use date pickers instead of manual date entry
- Use dropdowns for limited options instead of free text
- Provide autocomplete suggestions where applicable
- Show character counts for fields with limits

## Code Standards

### File Structure

```
src/
├── components/          # Self-contained UI components
│   ├── forms/          # Form-specific components
│   ├── layout/         # Layout components
│   └── ui/             # Reusable UI components
├── hooks/              # Custom hooks for data access
│   ├── useAuth.ts      # Authentication hook
│   ├── useForm.ts      # Form management hook
│   └── useSupabase.ts  # Supabase operations hook
├── types/              # TypeScript type definitions
├── utils/              # Utility functions
├── theme/              # Material-UI theme configuration
└── assets/             # Static assets including SVGs
```

### Component Guidelines

- Use functional components with TypeScript
- Implement proper error boundaries
- Use Material-UI's sx prop for styling
- Follow the theme system for consistent design
- Implement proper accessibility attributes

### Hook Guidelines

- Prefix custom hooks with 'use'
- Return objects with descriptive property names
- Handle loading and error states consistently
- Use proper TypeScript typing for hook parameters and return values

### Loading State Guidelines

- Always show skeleton loading instead of spinners
- Match skeleton shapes to actual content
- Use consistent loading patterns across the app
- Implement progressive loading for complex components

### Illustration Guidelines

- Use SVG format for scalability
- Implement proper color theming
- Add appropriate alt text for accessibility
- Consider animation for better engagement

## Material-UI Best Practices

### Theme Usage

- Always use theme values from `theme.palette`, `theme.typography`, etc.
- Create custom theme variants when needed
- Use sx prop for component-specific styling
- Implement dark mode support when applicable

### Component Selection

- Prefer Material-UI components over custom implementations
- Use appropriate component variants (contained, outlined, text)
- Implement proper form validation with Material-UI
- Use Material-UI's built-in accessibility features

### Responsive Design

- Use Material-UI's breakpoint system
- Implement mobile-first responsive design
- Test components on different screen sizes
- Use appropriate spacing and sizing scales

## Supabase Integration

### Authentication

- Use Supabase Auth hooks for user management
- Implement proper session handling
- Handle authentication errors gracefully
- Use TypeScript types for user data

### Database Operations

- Create custom hooks for database operations
- Implement proper error handling
- Use optimistic updates when appropriate
- Implement proper loading states

## Performance Considerations

### Code Splitting

- Implement lazy loading for route components
- Use dynamic imports for heavy components
- Optimize bundle size with tree shaking

### State Management

- Use React's built-in state management when possible
- Consider Context API for global state
- Implement proper memoization with useMemo and useCallback

## Testing Strategy

### Component Testing

- Test components in isolation
- Mock external dependencies
- Test loading and error states
- Test accessibility features

### Hook Testing

- Test custom hooks with React Testing Library
- Test different hook scenarios
- Test error handling
- Test loading states

## Development Workflow

### Before Creating Components

1. Check if a similar component already exists
2. Determine if it should be self-contained or part of a larger component
3. Plan the component's props interface
4. Consider the loading states needed

### Before Creating Hooks

1. Identify the data access pattern
2. Determine the hook's parameters and return values
3. Plan error handling strategy
4. Consider caching and optimization

### Before Adding Loading States

1. Identify what content will be loaded
2. Design skeleton components that match the content
3. Implement progressive loading if needed
4. Test loading states thoroughly

### Before Adding Illustrations

1. Determine the purpose of the illustration
2. Choose appropriate SVG format
3. Implement proper theming
4. Test accessibility and responsiveness

## Common Patterns

### Form Components

```typescript
const FormComponent: React.FC<FormProps> = ({ onSubmit, initialData }) => {
  const { data, loading, error, handleSubmit } = useForm(initialData);

  if (loading) return <FormSkeleton />;

  return <form onSubmit={handleSubmit}>{/* Form fields */}</form>;
};
```

### Data Display Components

```typescript
const DataComponent: React.FC<DataProps> = ({ dataId }) => {
  const { data, loading, error } = useData(dataId);

  if (loading) return <DataSkeleton />;
  if (error) return <ErrorIllustration />;
  if (!data) return <EmptyStateIllustration />;

  return <DataDisplay data={data} />;
};
```

### Authentication Components

```typescript
const AuthComponent: React.FC = () => {
  const { user, loading, signIn, signOut } = useAuth();

  if (loading) return <AuthSkeleton />;

  return user ? <UserDashboard /> : <SignInForm />;
};
```

## Quality Checklist

Before submitting code, ensure:

- [ ] Components are self-contained and reusable
- [ ] Data access is handled through custom hooks
- [ ] Loading states use Material-UI Skeleton components
- [ ] Vector illustrations are used where appropriate
- [ ] TypeScript types are properly defined
- [ ] Material-UI theme is used consistently
- [ ] Components are responsive and accessible
- [ ] Error states are handled gracefully
- [ ] Code follows the established patterns
- [ ] Performance considerations are addressed

---
> Source: [besongsamuel/rawbank-signup](https://github.com/besongsamuel/rawbank-signup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
