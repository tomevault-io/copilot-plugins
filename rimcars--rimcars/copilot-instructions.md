## rimcars

> Based on the centralized type system documented in [CODEBASE_ANALYSIS.md](mdc:CODEBASE_ANALYSIS.md), follow these TypeScript patterns:

# TypeScript Patterns & Best Practices

## 🔧 Type System Organization

Based on the centralized type system documented in [CODEBASE_ANALYSIS.md](mdc:CODEBASE_ANALYSIS.md), follow these TypeScript patterns:

### Central Type Exports
All types should be exported from `src/types/index.ts` for consistent imports across the codebase.

```typescript
// ✅ Always import from centralized types
import type { DatabaseUser, CarListing, ListingFormData } from "@/types";

// ❌ Don't import from scattered type files
import type { User } from "../features/auth/types/user";
```

### Database Type Integration
The project uses auto-generated Supabase types as the source of truth:

```typescript
// ✅ Current pattern - Use database schema types
import type { Tables } from "@/types/database.types";

export type DatabaseUser = Tables<'users'>;
export type DatabaseListing = Tables<'listings'>;

// ✅ For forms and extended types
export interface ListingFormData extends Omit<DatabaseListing, 'id' | 'created_at' | 'updated_at'> {
  images: File[]; // File objects for upload
}

export interface ExtendedListing extends DatabaseListing {
  seller?: DatabaseUser; // Joined data
}
```

## 🎯 Component Type Patterns

### Props Interface Definitions
```typescript
// ✅ Clear interface naming and optional props
interface UserMenuProps {
  user: DatabaseUser | null;
  className?: string;
  onSignOut?: () => void;
}

// ✅ Generic component props with constraints
interface CardProps<T = any> {
  data: T;
  onSelect?: (item: T) => void;
  className?: string;
}

// ✅ Event handler types
interface CarCardProps {
  car: DatabaseListing;
  onFavoriteToggle?: (carId: string) => void;
  onView?: (car: DatabaseListing) => void;
}
```

### Server Action Type Safety
```typescript
// ✅ Server action return types
interface ActionResult<T = any> {
  success: boolean;
  data?: T;
  error?: string;
}

// ✅ Server action definitions
export async function createListing(
  formData: ListingFormData
): Promise<ActionResult<DatabaseListing>> {
  try {
    // Implementation
    return { success: true, data: newListing };
  } catch (error) {
    return { success: false, error: "فشل في إنشاء الإعلان" };
  }
}

// ✅ Client-side usage with proper typing
const handleSubmit = async (data: ListingFormData) => {
  const result = await createListing(data);
  if (result.success && result.data) {
    // TypeScript knows result.data is DatabaseListing
    router.push(`/cars/${result.data.id}`);
  } else {
    toast.error(result.error);
  }
};
```

## 🔄 State Management Types

### Zustand Store Types
```typescript
// ✅ Favorites store typing (client-side state)
interface FavoritesState {
  favorites: string[];
  addFavorite: (carId: string) => void;
  removeFavorite: (carId: string) => void;
  isFavorite: (carId: string) => boolean;
  toggleFavorite: (carId: string) => void;
  clearFavorites: () => void;
}

export const useFavoritesStore = create<FavoritesState>()(
  persist(
    (set, get) => ({
      favorites: [],
      addFavorite: (carId) => {
        const { favorites } = get();
        if (!favorites.includes(carId)) {
          set({ favorites: [...favorites, carId] });
        }
      },
      // ... other methods
    }),
    { name: "car-favorites" }
  )
);
```

### Form Validation Types
```typescript
// ✅ Zod schema with TypeScript integration
import { z } from "zod";

export const createListingSchema = z.object({
  car_name: z.string().min(1, "اسم السيارة مطلوب"),
  description: z.string().min(10, "الوصف يجب أن يكون 10 أحرف على الأقل"),
  price: z.number().min(1, "السعر مطلوب"),
  make: z.string().min(1, "الماركة مطلوبة"),
  model: z.string().min(1, "الموديل مطلوب"),
  year: z.number().min(1990).max(new Date().getFullYear() + 1),
  mileage: z.number().min(0),
  condition: z.enum(["new", "used", "certified"]),
  transmission: z.enum(["manual", "automatic"]),
  fuel_type: z.enum(["gasoline", "diesel", "hybrid", "electric"]),
  location: z.string().min(1, "الموقع مطلوب"),
});

// ✅ Infer TypeScript type from Zod schema
export type CreateListingData = z.infer<typeof createListingSchema>;
```

## 🌐 API & Server Types

### Next.js App Router Types
```typescript
// ✅ Page component types
interface PageProps {
  params: { id: string };
  searchParams: { [key: string]: string | string[] | undefined };
}

export default async function CarDetailPage({ params }: PageProps) {
  const car = await getPublicListingById(params.id);
  // TypeScript knows car is DatabaseListing | null
}

// ✅ Layout component types
interface LayoutProps {
  children: React.ReactNode;
  params?: { [key: string]: string };
}

export default async function PublicLayout({ children }: LayoutProps) {
  const user = await getUserWithProfile();
  return (
    <>
      <Header user={user} />
      {children}
    </>
  );
}
```

### Server Action Parameter Types
```typescript
// ✅ Form action types with proper validation
export async function updateUserProfile(
  userId: string,
  data: {
    name?: string;
    phone_number?: string;
    avatar_url?: string;
  }
): Promise<ActionResult<DatabaseUser>> {
  // Implementation with type safety
}

// ✅ File upload types
export async function uploadCarImages(
  listingId: string,
  images: File[]
): Promise<ActionResult<string[]>> {
  // Returns array of image URLs
}
```

## 🎨 UI Component Types

### Common Component Patterns
```typescript
// ✅ Variant-based component props
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: "default" | "destructive" | "outline" | "secondary" | "ghost" | "link";
  size?: "default" | "sm" | "lg" | "icon";
  isLoading?: boolean;
}

// ✅ Polymorphic component types
interface TypographyProps<T extends React.ElementType = "p"> {
  as?: T;
  variant?: "h1" | "h2" | "h3" | "h4" | "body" | "caption";
  children: React.ReactNode;
  className?: string;
}

function Typography<T extends React.ElementType = "p">({
  as,
  variant = "body",
  className,
  children,
  ...props
}: TypographyProps<T> & Omit<React.ComponentPropsWithoutRef<T>, keyof TypographyProps<T>>) {
  const Component = as || "p";
  return <Component className={cn(typographyVariants({ variant }), className)} {...props}>{children}</Component>;
}
```

### Modal and Dialog Types
```typescript
// ✅ Modal state management types
interface ModalState {
  isOpen: boolean;
  data?: any;
  onClose: () => void;
}

interface ConfirmDialogProps {
  title: string;
  description: string;
  confirmText?: string;
  cancelText?: string;
  onConfirm: () => void | Promise<void>;
  onCancel?: () => void;
}
```

## 🔍 Search and Filter Types

### Advanced Search Types
```typescript
// ✅ Search filter types
interface CarSearchFilters {
  query?: string;
  make?: string;
  model?: string;
  yearMin?: number;
  yearMax?: number;
  priceMin?: number;
  priceMax?: number;
  mileageMax?: number;
  condition?: DatabaseListing['condition'];
  transmission?: DatabaseListing['transmission'];
  fuelType?: DatabaseListing['fuel_type'];
  location?: string;
  sortBy?: 'price_asc' | 'price_desc' | 'year_desc' | 'mileage_asc' | 'created_at_desc';
}

// ✅ Search result types with pagination
interface SearchResult<T> {
  items: T[];
  total: number;
  page: number;
  pageSize: number;
  hasMore: boolean;
}

export type CarSearchResult = SearchResult<DatabaseListing>;
```

## 🛡️ Error Handling Types

### Centralized Error Types
```typescript
// ✅ Application error types
interface AppError {
  code: string;
  message: string;
  details?: Record<string, any>;
}

interface ValidationError extends AppError {
  code: 'VALIDATION_ERROR';
  field: string;
  value: any;
}

interface DatabaseError extends AppError {
  code: 'DATABASE_ERROR';
  operation: 'CREATE' | 'READ' | 'UPDATE' | 'DELETE';
  table?: string;
}

// ✅ Result type for operations that can fail
type Result<T, E = AppError> = 
  | { success: true; data: T }
  | { success: false; error: E };
```

## 🧪 Testing Types (Future Implementation)

### Test Utility Types
```typescript
// ✅ Mock data factory types
interface MockUserOptions {
  role?: DatabaseUser['role'];
  verified?: boolean;
  withProfile?: boolean;
}

function createMockUser(options: MockUserOptions = {}): DatabaseUser {
  return {
    id: crypto.randomUUID(),
    email: "test@example.com",
    name: "Test User",
    role: options.role || "buyer",
    // ... other fields
  };
}

// ✅ Component test props
interface RenderOptions {
  user?: DatabaseUser | null;
  router?: Partial<NextRouter>;
  wrapper?: React.ComponentType<{ children: React.ReactNode }>;
}
```

## 🚨 TypeScript Anti-Patterns to Avoid

### ❌ Avoid These Patterns
```typescript
// ❌ Don't use 'any' unless absolutely necessary
function handleData(data: any) { ... }

// ✅ Use proper typing instead
function handleData<T extends DatabaseListing>(data: T) { ... }

// ❌ Don't ignore TypeScript errors
// @ts-ignore
const user = getUserData();

// ✅ Fix the underlying type issue
const user = await getUserWithProfile();

// ❌ Don't create duplicate type definitions
interface User { ... } // in multiple files

// ✅ Use centralized types
import type { DatabaseUser } from "@/types";
```

### ⚡ Type Performance Considerations
```typescript
// ✅ Use type assertions sparingly and safely
const element = document.getElementById('my-element') as HTMLInputElement;

// ✅ Prefer type guards for runtime checks
function isDatabaseUser(user: unknown): user is DatabaseUser {
  return typeof user === 'object' && user !== null && 'id' in user;
}

// ✅ Use const assertions for literal types
const LISTING_STATUSES = ['draft', 'published', 'sold'] as const;
type ListingStatus = typeof LISTING_STATUSES[number];
```

## 📚 Type References

- **Database Types**: Auto-generated from Supabase schema in `src/types/database.types.ts`
- **Component Types**: Based on Radix UI and shadcn/ui patterns
- **Form Types**: Zod schemas with TypeScript inference
- **State Types**: Zustand store interfaces for client-side state

Always refer to [CODEBASE_ANALYSIS.md](mdc:CODEBASE_ANALYSIS.md) for current type organization and architectural decisions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rimcars) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
