## myrandomcalendar

> - React components should be **Pure**: Data flows in one direction

# React-Specific Rules

## 🎯 React Architecture Principles

### **Pure Functional Components**
- React components should be **Pure**: Data flows in one direction
- Only manage presentational state, **NO DATA FETCHING**
- If something in React needs to make a POST update, it happens on a form
- Use props for all data, avoid `useEffect` for initial data loading

### **Component Design**
- Prefer functional components over class components
- Use TypeScript for all component props and state
- Keep components small and focused on single responsibility
- Use composition over inheritance

## 🚫 What NOT to Do in React Components

### **Avoid Data Fetching**
```typescript
// ❌ DON'T DO THIS
const [data, setData] = useState([]);

useEffect(() => {
  // Don't fetch initial data in React
  fetchData().then(setData);
}, []);
```

### **Avoid Complex State Management**
```typescript
// ❌ DON'T DO THIS
const [user, setUser] = useState(null);
const [loading, setLoading] = useState(false);
const [error, setError] = useState(null);

useEffect(() => {
  setLoading(true);
  fetchUser().then(setUser).catch(setError).finally(() => setLoading(false));
}, []);
```

### **Avoid Custom Button Styling**
```typescript
// ❌ DON'T DO THIS
<button className="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">
  Click me
</button>

<a className="bg-green-600 hover:bg-green-700 text-white font-bold py-2 px-4 rounded">
  Link button
</a>
```

## ✅ What TO Do in React Components

### **Accept Data as Props**
```typescript
// ✅ DO THIS
interface DashboardProps {
  user: User;
  data: DashboardData[];
  onUpdate: (id: string, data: Partial<DashboardData>) => void;
}

export function Dashboard({ user, data, onUpdate }: DashboardProps) {
  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      {data.map(item => (
        <DashboardItem key={item.id} item={item} onUpdate={onUpdate} />
      ))}
    </div>
  );
}
```

### **Handle Form Submissions**
```typescript
// ✅ DO THIS
interface FormProps {
  onSubmit: (data: FormData) => void;
  initialData?: Partial<FormData>;
}

export function UserForm({ onSubmit, initialData }: FormProps) {
  const [formData, setFormData] = useState(initialData || {});

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSubmit(formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* form fields */}
    </form>
  );
}
```

## 🎨 ShadCN Component Guidelines

### **Button Components (CRITICAL)**
- **Use `Button`** for all `<button>` elements
- **Use `ButtonLink`** for all `<a>` tags styled as buttons
- **Never use custom button styling** - only ShadCN variants
- **Import both components** when needed in Astro files

```typescript
// ✅ CORRECT: React components
import { Button } from './ui/button';

<Button type="submit" variant="destructive">
  Delete
</Button>

<Button variant="outline" size="sm">
  Edit
</Button>
```

```astro
<!-- ✅ CORRECT: Astro files -->
---
import { Button } from '@/components/ui/button';
import ButtonLink from '@/components/ui/button-link';
---

<Button type="submit" className="w-full">
  Submit
</Button>

<ButtonLink variant="secondary" href="/back">
  Back
</ButtonLink>
```

### **Variant Selection Rules**
- `default` - Primary actions (create, submit, main CTAs)
- `secondary` - Secondary actions (back, cancel, navigation)
- `destructive` - Delete, remove, dangerous actions
- `outline` - Alternative actions, form cancel buttons
- `ghost` - Subtle actions, close buttons, icon buttons
- `link` - Text links within content

```typescript
// ✅ CORRECT: Semantic variant usage
<Button variant="default">Create New Event</Button>        // Primary action
<Button variant="secondary">Cancel</Button>               // Secondary action
<Button variant="destructive">Delete</Button>             // Dangerous action
<Button variant="outline">Back to Schedule</Button>       // Alternative action
<Button variant="ghost" size="icon">×</Button>            // Close button
<ButtonLink variant="link" href="/login">Sign in</ButtonLink> // Text link
```

### **Size Guidelines**
- `default` - Standard buttons (most common)
- `sm` - Small buttons (edit, delete actions in lists)
- `lg` - Large buttons (hero CTAs, important actions)
- `icon` - Icon-only buttons (close, menu, etc.)

```typescript
// ✅ CORRECT: Size usage
<Button size="default">Primary Action</Button>
<Button size="sm" variant="destructive">Delete</Button>
<Button size="lg">Get Started</Button>
<Button size="icon" variant="ghost">×</Button>
```

## 🎨 Styling Guidelines

### **Use ShadCN Components First**
- **Prefer ShadCN components** over custom styling
- **Use TailwindCSS utilities** for layout and spacing
- **Use `className` prop** for React components
- **Use `class` prop** for Astro/HTML elements
- **Never override ShadCN component styles** - use variants instead

```typescript
// ✅ DO THIS: ShadCN components
<div className="flex items-center justify-between p-4">
  <h2 className="text-xl font-semibold">Title</h2>
  <Button variant="outline">Action</Button>
</div>

// ❌ DON'T DO THIS: Custom button styling
<button className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600">
  Action
</button>
```

### **Component Composition**
- Compose ShadCN components rather than overriding styles
- Use variants and sizes instead of custom CSS
- Leverage ShadCN's built-in accessibility features
- Pass styling props correctly to React components

## 🔗 Astro + React Integration

### **Import Patterns**
```typescript
// ✅ CORRECT: Astro page imports
import { Button } from '@/components/ui/button';
import ButtonLink from '@/components/ui/button-link';
import MyReactComponent from '../components/MyReactComponent';

// ✅ CORRECT: React component imports
import { Button } from './ui/button';
```

### **Styling in Astro**
- Use `class` for Astro/HTML elements
- Use `className` for React components
- Pass styling props correctly to React components

```astro
<!-- ✅ CORRECT: Astro with React components -->
<Button type="submit" className="w-full">
  Submit
</Button>

<ButtonLink variant="secondary" href="/back">
  Back
</ButtonLink>

<!-- ✅ CORRECT: Native HTML elements -->
<div class="flex items-center space-x-4">
  <input class="px-3 py-2 border rounded-md" />
</div>
```

### **Component Usage Patterns**
```astro
<!-- ✅ CORRECT: Form submission -->
<form method="POST" action={action}>
  <Button type="submit" className="w-full">
    {editing ? 'Update' : 'Create'}
  </Button>
</form>

<!-- ✅ CORRECT: Navigation -->
<div class="flex space-x-4">
  <ButtonLink href="/calendar">Calendar</ButtonLink>
  <ButtonLink variant="secondary" href="/schedule">Schedule</ButtonLink>
</div>

<!-- ✅ CORRECT: Actions in lists -->
<div class="flex space-x-2">
  <ButtonLink variant="outline" size="sm" href={`/edit/${item.id}`}>
    Edit
  </ButtonLink>
  <Button variant="destructive" size="sm">
    Delete
  </Button>
</div>
```

## 🔧 React Best Practices

### **Props Interface**
- Always define TypeScript interfaces for props
- Use descriptive prop names
- Group related props together

```typescript
interface CalendarEventProps {
  event: CalendarEvent;
  isSelected: boolean;
  onSelect: (event: CalendarEvent) => void;
  onEdit: (event: CalendarEvent) => void;
  onDelete: (eventId: string) => void;
}
```

### **Event Handlers**
- Use descriptive handler names
- Pass minimal data in callbacks
- Use `useCallback` for performance optimization when needed

```typescript
const handleEventClick = useCallback((event: CalendarEvent) => {
  onSelect(event);
}, [onSelect]);

const handleEditClick = useCallback((event: React.MouseEvent) => {
  event.stopPropagation();
  onEdit(event);
}, [onEdit]);
```

### **Conditional Rendering**
- Use clear conditional patterns
- Prefer early returns for complex conditions

```typescript
// ✅ DO THIS
if (!user) {
  return <LoginPrompt />;
}

if (loading) {
  return <LoadingSpinner />;
}

return (
  <div>
    {/* main content */}
  </div>
);
```

## 🎯 Component Patterns

### **Presentational Components**
- Focus on UI and presentation
- Accept all data via props
- Handle user interactions via callback props

### **Form Components**
- Use controlled components
- Validate input on the client side
- Submit data via form actions or callback props

## 📝 **Form Management Patterns (PROVEN)**

### **Complex Forms with React Components**
For complex forms with multiple fields, validation, and dynamic behavior:

```typescript
// ✅ DO THIS: React form component with client:load
interface FormProps {
  initialData?: Partial<FeatureType>;
  formOptions: PageData['formOptions'];
}

export default function FeatureForm({ initialData, formOptions }: FormProps) {
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget as HTMLFormElement);
    
    try {
      // Dynamic import to avoid server-side issues
      const { actions } = await import('astro:actions');
      const result = await actions.feature.action(formData);
      
      if (!result.error) {
        window.location.href = '/success-page';
      } else {
        console.error('Form submission error:', result.error);
        alert('Error: ' + result.error.message);
      }
    } catch (error) {
      console.error('Form submission failed:', error);
      alert('Form submission failed. Please try again.');
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-6">
      {/* ShadCN form fields */}
    </form>
  );
}
```

### **Astro Integration for Complex Forms**
```astro
<!-- ✅ CORRECT: Astro page with React form component -->
---
import FeatureForm from '../../components/FeatureForm.tsx';
import type { PageData } from '@/features/feature/services/page-handler';
---

<Card>
  <CardContent>
    <FeatureForm 
      client:load
      initialData={pageData.editingItem}
      formOptions={pageData.formOptions}
    />
  </CardContent>
</Card>
```

### **Type Reuse Pattern**
```typescript
// ✅ DO THIS: Reuse existing feature types
import type { FeatureType } from '@/features/feature/models/Feature.types';
import type { PageData } from '@/features/feature/services/page-handler';

interface FormProps {
  initialData?: Partial<FeatureType>;
  formOptions: PageData['formOptions'];
}

// ❌ DON'T DO THIS: Create new types
interface FormProps {
  initialData?: {
    id?: string;
    title?: string;
    // ... manual field definitions
  };
  formOptions: {
    // ... manual form option definitions
  };
}
```

### **ShadCN Form Components**
```typescript
// ✅ Form inputs with ShadCN
<Input 
  type="text" 
  name="title" 
  defaultValue={initialData?.title || ''} 
  className="w-full"
  required 
/>

<Textarea 
  name="description" 
  defaultValue={initialData?.description || ''} 
  className="w-full"
  rows={3}
/>

<Select name="category" defaultValue={initialData?.category || 'default'} required>
  <SelectTrigger className="w-full">
    <SelectValue placeholder="Select category" />
  </SelectTrigger>
  <SelectContent>
    {formOptions.categories.map((option) => (
      <SelectItem key={option.value} value={option.value}>
        {option.label}
      </SelectItem>
    ))}
  </SelectContent>
</Select>

<Checkbox 
  name="isActive" 
  defaultChecked={initialData?.isActive ?? true}
  className="mr-2"
/>
```

### **Form Complexity Decision Matrix**
- **Simple Forms** (1-3 fields): Use ShadCN components directly in Astro
- **Complex Forms** (4+ fields, validation, dynamic behavior): Use React component with `client:load`
- **Forms with complex state**: Always use React component
- **Forms with server actions**: Use React component with dynamic action imports

### **List Components**
- Use proper `key` props for list items
- Implement proper accessibility attributes
- Handle empty states gracefully

## 🔍 Common React Patterns in This Project

### **Calendar Components**
```typescript
interface FullCalendarProps {
  events: CalendarEvent[];
  onEventClick: (event: CalendarEvent) => void;
  onDateSelect: (date: Date) => void;
  initialDate?: Date;
}

export function FullCalendar({ events, onEventClick, onDateSelect, initialDate }: FullCalendarProps) {
  // Pure presentation logic only
  return (
    <div className="calendar-container">
      {/* calendar implementation */}
    </div>
  );
}
```

### **Modal Components**
```typescript
interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  children: React.ReactNode;
}

export function Modal({ isOpen, onClose, title, children }: ModalProps) {
  if (!isOpen) return null;

  return (
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={(e) => e.stopPropagation()}>
        <div className="flex justify-between items-center">
          <h2>{title}</h2>
          <Button variant="ghost" size="icon" onClick={onClose}>
            ×
          </Button>
        </div>
        {children}
      </div>
    </div>
  );
}
```

### **Button Usage Patterns**
```typescript
// ✅ Primary actions
<Button type="submit">Create Event</Button>
<ButtonLink href="/create">Add New</ButtonLink>

// ✅ Secondary actions
<Button variant="secondary">Cancel</Button>
<ButtonLink variant="secondary" href="/back">Back</ButtonLink>

// ✅ Destructive actions
<Button variant="destructive">Delete</Button>
<Button variant="destructive" size="sm">Remove</Button>

// ✅ Alternative actions
<Button variant="outline">Edit</Button>
<ButtonLink variant="outline" href="/edit">Modify</ButtonLink>

// ✅ Subtle actions
<Button variant="ghost" size="icon">×</Button>
<Button variant="ghost">Close</Button>

// ✅ Text links
<ButtonLink variant="link" href="/login">Sign in</ButtonLink>
```

## 🚨 Critical Rules Summary

1. **ALWAYS use ShadCN Button components** - Never custom button styling
2. **Use `ButtonLink` for all link buttons** - Never style `<a>` tags as buttons
3. **Choose variants semantically** - Match variant to action type
4. **Import correctly** - Use `@/components/ui/button` in Astro, `./ui/button` in React
5. **Use `className` for React, `class` for Astro** - Maintain proper prop usage
6. **Compose, don't override** - Use variants instead of custom CSS

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dvelasquez) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
