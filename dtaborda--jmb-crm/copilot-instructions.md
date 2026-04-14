## jmb-crm

> This rule defines the complete UI design system for JMB CRM, covering component architecture, styling patterns, Tailwind CSS usage, accessibility guidelines, and responsive design principles.

# UI Design System & Component Patterns

## Overview
This rule defines the complete UI design system for JMB CRM, covering component architecture, styling patterns, Tailwind CSS usage, accessibility guidelines, and responsive design principles.

---

## 1. Design System Foundation

### Color Palette
```typescript
const DESIGN_TOKENS = {
  colors: {
    // Primary brand colors
    primary: 'indigo',
    secondary: 'gray',
    success: 'green',
    warning: 'yellow',
    error: 'red',
    info: 'blue',
    
    // Semantic colors
    text: {
      primary: 'gray-900',
      secondary: 'gray-600',
      tertiary: 'gray-500',
      disabled: 'gray-400'
    },
    
    // Background colors
    background: {
      primary: 'white',
      secondary: 'gray-50',
      tertiary: 'gray-100'
    },
    
    // Border colors
    border: {
      primary: 'gray-200',
      secondary: 'gray-300',
      focus: 'indigo-500'
    }
  },
  
  spacing: {
    xs: '0.5rem',   // 8px
    sm: '0.75rem',  // 12px
    md: '1rem',     // 16px
    lg: '1.25rem',  // 20px
    xl: '1.5rem',   // 24px
    '2xl': '2rem'   // 32px
  },
  
  typography: {
    fontFamily: {
      sans: ['Inter', 'system-ui', 'sans-serif'],
      mono: ['JetBrains Mono', 'monospace']
    },
    fontSize: {
      xs: '0.75rem',
      sm: '0.875rem',
      base: '1rem',
      lg: '1.125rem',
      xl: '1.25rem',
      '2xl': '1.5rem',
      '3xl': '1.875rem'
    }
  }
} as const;
```

### Responsive Breakpoints
```typescript
const BREAKPOINTS = {
  sm: '640px',   // Mobile
  md: '768px',   // Tablet
  lg: '1024px',  // Desktop
  xl: '1280px',  // Large Desktop
  '2xl': '1536px' // Extra Large
} as const;
```

---

## 2. Component Architecture

### Base Component Structure
```typescript
interface BaseComponentProps {
  className?: string;
  children?: React.ReactNode;
  testId?: string;
}

interface ComponentVariants {
  variant: 'primary' | 'secondary' | 'outline' | 'ghost';
  size: 'sm' | 'md' | 'lg';
  disabled?: boolean;
}
```

### Variant System
Use a consistent variant system across all components:

```typescript
const getVariantClasses = (variant: string, size: string) => {
  const variants = {
    primary: 'bg-indigo-600 text-white hover:bg-indigo-700',
    secondary: 'bg-gray-600 text-white hover:bg-gray-700',
    outline: 'border border-gray-300 text-gray-700 hover:bg-gray-50',
    ghost: 'text-gray-700 hover:bg-gray-100'
  };
  
  const sizes = {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg'
  };
  
  return `${variants[variant]} ${sizes[size]}`;
};
```

---

## 3. Core UI Components

### Button Component
```typescript
interface ButtonProps extends BaseComponentProps, ComponentVariants {
  onClick?: () => void;
  type?: 'button' | 'submit' | 'reset';
  loading?: boolean;
  icon?: React.ReactNode;
  fullWidth?: boolean;
}

const Button: React.FC<ButtonProps> = ({
  children,
  className = '',
  variant = 'primary',
  size = 'md',
  disabled = false,
  loading = false,
  icon,
  fullWidth = false,
  testId,
  ...props
}) => {
  const baseClasses = 'inline-flex items-center justify-center rounded-md font-medium transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2';
  const variantClasses = getVariantClasses(variant, size);
  const disabledClasses = disabled || loading ? 'opacity-50 cursor-not-allowed' : '';
  const fullWidthClasses = fullWidth ? 'w-full' : '';
  
  return (
    <button
      className={`${baseClasses} ${variantClasses} ${disabledClasses} ${fullWidthClasses} ${className}`}
      disabled={disabled || loading}
      data-testid={testId}
      {...props}
    >
      {loading && <Loader2 className="mr-2 h-4 w-4 animate-spin" />}
      {!loading && icon && <span className="mr-2">{icon}</span>}
      {children}
    </button>
  );
};
```

### Card Component
```typescript
interface CardProps extends BaseComponentProps {
  padding?: 'sm' | 'md' | 'lg';
  shadow?: 'sm' | 'md' | 'lg';
  rounded?: 'sm' | 'md' | 'lg';
  header?: React.ReactNode;
  footer?: React.ReactNode;
}

const Card: React.FC<CardProps> = ({
  children,
  className = '',
  padding = 'md',
  shadow = 'sm',
  rounded = 'md',
  header,
  footer,
  testId
}) => {
  const paddingClasses = {
    sm: 'p-4',
    md: 'p-6',
    lg: 'p-8'
  };
  
  const shadowClasses = {
    sm: 'shadow-sm',
    md: 'shadow-md',
    lg: 'shadow-lg'
  };
  
  const roundedClasses = {
    sm: 'rounded',
    md: 'rounded-lg',
    lg: 'rounded-xl'
  };
  
  return (
    <div
      className={`bg-white border border-gray-200 ${paddingClasses[padding]} ${shadowClasses[shadow]} ${roundedClasses[rounded]} ${className}`}
      data-testid={testId}
    >
      {header && (
        <div className="border-b border-gray-200 pb-4 mb-4">
          {header}
        </div>
      )}
      
      <div>{children}</div>
      
      {footer && (
        <div className="border-t border-gray-200 pt-4 mt-4">
          {footer}
        </div>
      )}
    </div>
  );
};
```

### Input Component
```typescript
interface InputProps extends BaseComponentProps {
  type?: 'text' | 'email' | 'password' | 'number' | 'tel';
  placeholder?: string;
  value?: string;
  onChange?: (value: string) => void;
  error?: string;
  label?: string;
  required?: boolean;
  disabled?: boolean;
}

const Input: React.FC<InputProps> = ({
  type = 'text',
  placeholder,
  value,
  onChange,
  error,
  label,
  required = false,
  disabled = false,
  className = '',
  testId
}) => {
  const baseClasses = 'block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500';
  const errorClasses = error ? 'border-red-300 focus:border-red-500 focus:ring-red-500' : '';
  const disabledClasses = disabled ? 'bg-gray-50 text-gray-500 cursor-not-allowed' : '';
  
  return (
    <div className={className}>
      {label && (
        <label className="block text-sm font-medium text-gray-700 mb-1">
          {label}
          {required && <span className="text-red-500 ml-1">*</span>}
        </label>
      )}
      
      <input
        type={type}
        placeholder={placeholder}
        value={value}
        onChange={(e) => onChange?.(e.target.value)}
        disabled={disabled}
        className={`${baseClasses} ${errorClasses} ${disabledClasses}`}
        data-testid={testId}
      />
      
      {error && (
        <p className="mt-1 text-sm text-red-600">{error}</p>
      )}
    </div>
  );
};
```

### Badge Component
```typescript
interface BadgeProps extends BaseComponentProps {
  variant?: 'success' | 'warning' | 'error' | 'info' | 'neutral';
  size?: 'sm' | 'md';
  dot?: boolean;
}

const Badge: React.FC<BadgeProps> = ({
  children,
  className = '',
  variant = 'neutral',
  size = 'md',
  dot = false,
  testId
}) => {
  const variantClasses = {
    success: 'bg-green-100 text-green-800',
    warning: 'bg-yellow-100 text-yellow-800',
    error: 'bg-red-100 text-red-800',
    info: 'bg-blue-100 text-blue-800',
    neutral: 'bg-gray-100 text-gray-800'
  };
  
  const sizeClasses = {
    sm: 'px-2 py-0.5 text-xs',
    md: 'px-2.5 py-1 text-sm'
  };
  
  return (
    <span
      className={`inline-flex items-center rounded-full font-medium ${variantClasses[variant]} ${sizeClasses[size]} ${className}`}
      data-testid={testId}
    >
      {dot && (
        <span className={`mr-1.5 h-2 w-2 rounded-full ${variant === 'success' ? 'bg-green-400' : 'bg-current'}`} />
      )}
      {children}
    </span>
  );
};
```

---

## 4. Layout Components

### Container Component
```typescript
interface ContainerProps extends BaseComponentProps {
  maxWidth?: 'sm' | 'md' | 'lg' | 'xl' | '2xl' | 'full';
  padding?: boolean;
  centered?: boolean;
}

const Container: React.FC<ContainerProps> = ({
  children,
  className = '',
  maxWidth = 'lg',
  padding = true,
  centered = true,
  testId
}) => {
  const maxWidthClasses = {
    sm: 'max-w-sm',
    md: 'max-w-md',
    lg: 'max-w-4xl',
    xl: 'max-w-6xl',
    '2xl': 'max-w-7xl',
    full: 'max-w-full'
  };
  
  const paddingClasses = padding ? 'px-4 sm:px-6 lg:px-8' : '';
  const centeredClasses = centered ? 'mx-auto' : '';
  
  return (
    <div
      className={`${maxWidthClasses[maxWidth]} ${paddingClasses} ${centeredClasses} ${className}`}
      data-testid={testId}
    >
      {children}
    </div>
  );
};
```

### Grid Component
```typescript
interface GridProps extends BaseComponentProps {
  cols?: 1 | 2 | 3 | 4 | 6 | 12;
  gap?: 'sm' | 'md' | 'lg';
  responsive?: boolean;
}

const Grid: React.FC<GridProps> = ({
  children,
  className = '',
  cols = 1,
  gap = 'md',
  responsive = true,
  testId
}) => {
  const colClasses = {
    1: 'grid-cols-1',
    2: responsive ? 'grid-cols-1 sm:grid-cols-2' : 'grid-cols-2',
    3: responsive ? 'grid-cols-1 sm:grid-cols-2 lg:grid-cols-3' : 'grid-cols-3',
    4: responsive ? 'grid-cols-1 sm:grid-cols-2 lg:grid-cols-4' : 'grid-cols-4',
    6: responsive ? 'grid-cols-2 sm:grid-cols-3 lg:grid-cols-6' : 'grid-cols-6',
    12: 'grid-cols-12'
  };
  
  const gapClasses = {
    sm: 'gap-4',
    md: 'gap-6',
    lg: 'gap-8'
  };
  
  return (
    <div
      className={`grid ${colClasses[cols]} ${gapClasses[gap]} ${className}`}
      data-testid={testId}
    >
      {children}
    </div>
  );
};
```

---

## 5. State Components

### Loading State
```typescript
interface LoadingStateProps extends BaseComponentProps {
  message?: string;
  size?: 'sm' | 'md' | 'lg';
  centered?: boolean;
}

const LoadingState: React.FC<LoadingStateProps> = ({
  message = 'Loading...',
  size = 'md',
  centered = true,
  className = '',
  testId
}) => {
  const sizeClasses = {
    sm: 'h-4 w-4',
    md: 'h-8 w-8',
    lg: 'h-12 w-12'
  };
  
  const centeredClasses = centered ? 'flex flex-col items-center justify-center' : '';
  
  return (
    <div className={`${centeredClasses} ${className}`} data-testid={testId}>
      <Loader2 className={`animate-spin text-indigo-600 ${sizeClasses[size]}`} />
      {message && (
        <p className="mt-2 text-sm text-gray-600">{message}</p>
      )}
    </div>
  );
};
```

### Empty State
```typescript
interface EmptyStateProps extends BaseComponentProps {
  title: string;
  description?: string;
  icon?: React.ReactNode;
  action?: React.ReactNode;
}

const EmptyState: React.FC<EmptyStateProps> = ({
  title,
  description,
  icon,
  action,
  className = '',
  testId
}) => {
  return (
    <div className={`text-center py-12 ${className}`} data-testid={testId}>
      {icon && (
        <div className="mx-auto h-12 w-12 text-gray-400 mb-4">
          {icon}
        </div>
      )}
      
      <h3 className="text-lg font-medium text-gray-900 mb-2">
        {title}
      </h3>
      
      {description && (
        <p className="text-gray-600 mb-6 max-w-sm mx-auto">
          {description}
        </p>
      )}
      
      {action && action}
    </div>
  );
};
```

---

## 6. Tailwind CSS Best Practices

### Configuration
```javascript
// tailwind.config.js
module.exports = {
  content: ['./src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eef2ff',
          500: '#6366f1',
          600: '#4f46e5',
          700: '#4338ca'
        }
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif']
      }
    }
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography')
  ]
};
```

### Utility-First Approach
```typescript
// ✅ Good: Use utility classes
<div className="flex items-center justify-between p-4 bg-white rounded-lg shadow-sm">
  <h2 className="text-lg font-semibold text-gray-900">Title</h2>
  <button className="px-4 py-2 bg-indigo-600 text-white rounded-md hover:bg-indigo-700">
    Action
  </button>
</div>

// ❌ Avoid: Custom CSS when utilities exist
<div className="custom-header">
  <h2 className="custom-title">Title</h2>
  <button className="custom-button">Action</button>
</div>
```

### Component Extraction
```typescript
// Extract repeated patterns into components
const ButtonGroup: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <div className="flex space-x-2">
    {children}
  </div>
);

const ActionButtons = () => (
  <ButtonGroup>
    <Button variant="outline">Cancel</Button>
    <Button variant="primary">Save</Button>
  </ButtonGroup>
);
```

---

## 7. Dark Mode Support

### Dark Mode Implementation
```typescript
// Add dark mode variants
const darkModeClasses = {
  card: 'bg-white dark:bg-gray-800 border-gray-200 dark:border-gray-700',
  text: 'text-gray-900 dark:text-gray-100',
  textSecondary: 'text-gray-600 dark:text-gray-400',
  button: 'bg-indigo-600 dark:bg-indigo-500 hover:bg-indigo-700 dark:hover:bg-indigo-600'
};

// Use in components
const Card: React.FC<CardProps> = ({ children, className = '' }) => (
  <div className={`${darkModeClasses.card} p-6 rounded-lg ${className}`}>
    {children}
  </div>
);
```

---

## 8. Accessibility Guidelines

### ARIA Labels and Descriptions
```typescript
// Always provide proper ARIA attributes
<button
  aria-label="Close dialog"
  aria-describedby="close-description"
  onClick={onClose}
>
  <X size={16} aria-hidden />
  <span id="close-description" className="sr-only">
    Closes the current dialog
  </span>
</button>
```

### Focus Management
```typescript
// Ensure proper focus management
const Modal: React.FC<ModalProps> = ({ isOpen, onClose, children }) => {
  const modalRef = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    if (isOpen && modalRef.current) {
      modalRef.current.focus();
    }
  }, [isOpen]);
  
  return (
    <div
      ref={modalRef}
      className="fixed inset-0 z-50 flex items-center justify-center"
      role="dialog"
      aria-modal="true"
      tabIndex={-1}
      onKeyDown={(e) => e.key === 'Escape' && onClose()}
    >
      {children}
    </div>
  );
};
```

### Keyboard Navigation
```typescript
// Support keyboard navigation
const Menu: React.FC<MenuProps> = ({ items, onSelect }) => {
  const [selectedIndex, setSelectedIndex] = useState(0);
  
  const handleKeyDown = (e: KeyboardEvent) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setSelectedIndex((prev) => (prev + 1) % items.length);
        break;
      case 'ArrowUp':
        e.preventDefault();
        setSelectedIndex((prev) => (prev - 1 + items.length) % items.length);
        break;
      case 'Enter':
        e.preventDefault();
        onSelect(items[selectedIndex]);
        break;
    }
  };
  
  return (
    <ul
      className="py-1 bg-white border border-gray-200 rounded-md shadow-lg"
      onKeyDown={handleKeyDown}
      tabIndex={0}
    >
      {items.map((item, index) => (
        <li
          key={item.id}
          className={`px-4 py-2 cursor-pointer ${
            index === selectedIndex ? 'bg-indigo-50' : 'hover:bg-gray-50'
          }`}
          onClick={() => onSelect(item)}
        >
          {item.label}
        </li>
      ))}
    </ul>
  );
};
```

---

## 9. Common Anti-patterns

### ❌ Avoid These Patterns

```typescript
// ❌ Don't use arbitrary values without good reason
<div className="mt-[17px] w-[342px]" />

// ❌ Don't create unnecessary custom CSS
.custom-button {
  @apply px-4 py-2 bg-blue-500 text-white rounded;
}

// ❌ Don't ignore responsive design
<div className="w-96"> {/* Fixed width, not responsive */}

// ❌ Don't forget accessibility
<button onClick={handleClick}>×</button> {/* No aria-label */}

// ❌ Don't use magic numbers
<div className="h-[73px]" /> {/* What is 73px for? */}
```

### ✅ Use These Patterns Instead

```typescript
// ✅ Use semantic spacing
<div className="mt-4 w-full max-w-sm" />

// ✅ Use component composition
<Button variant="primary" size="md">Action</Button>

// ✅ Design mobile-first
<div className="w-full md:w-96">

// ✅ Include proper accessibility
<button aria-label="Close" onClick={handleClose}>×</button>

// ✅ Use semantic class names
<div className="h-16" /> {/* Clear, standard spacing */}
```

---

**Remember**: Consistency is key. Use the design system components and patterns to ensure a cohesive user experience across the entire application.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dtaborda)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dtaborda)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
