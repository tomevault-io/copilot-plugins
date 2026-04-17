## smartcallr-frontend

> - Next.js 15 with App Router (app directory)

# SmartCallr Frontend - Next.js + ShadCN UI Project Rules

## Project Structure
- Next.js 15 with App Router (app directory)
- TypeScript for type safety
- ShadCN UI as the primary component library
- Tailwind CSS for styling
- SWR for data fetching and state management

## Code Style & Standards

### TypeScript/React Standards
- Use TypeScript for all files (.tsx, .ts)
- Follow React functional component patterns
- Use descriptive component and variable names
- Use PascalCase for components, camelCase for variables/functions
- Maximum line length: 100 characters
- Use proper type definitions and interfaces

### Folder Structure (MANDATORY)
```
src/
├── app/                    # Next.js app directory
│   └── feature-name/
│       └── page.tsx       # Import from components
├── components/            # UI components
│   ├── ui/               # ShadCN UI components
│   └── feature-name/
│       └── feature-name.tsx
├── interfaces/           # TypeScript interfaces
│   └── feature.ts
├── services/            # API services with SWR
│   └── feature.ts
└── lib/
    └── utils.ts         # Utility functions
```

### Component Pattern (MANDATORY)
```typescript
// Page Component (app directory)
import FeatureName from '@/components/feature-name/feature-name'

export default function FeatureNamePage() {
  return <FeatureName />
}

// Main Component
'use client';

import { useState } from 'react';
import { Button } from '@/components/ui/button';

export default function FeatureName() {
  // Component logic here
  return (
    <div>
      {/* JSX here */}
    </div>
  );
}
```

### ShadCN UI Usage (MANDATORY)
- ALWAYS use ShadCN UI components as the primary choice
- Import from `@/components/ui/*`
- Use proper ShadCN component props and variants
- Follow ShadCN design system patterns
- Custom styling only when ShadCN doesn't provide the needed component

### Services Pattern (MANDATORY)
```typescript
// API Service with SWR
import useSWR, { mutate } from 'swr';
import { api } from './api';
import toast from 'react-hot-toast';

export const featureAPI = {
  create: async (data: CreateRequest) => {
    try {
      const response = await api.post('/endpoint/', data);
      const result = response.data;
      
      if (result.status === 'success') {
        toast.success('Success message');
        mutate('/endpoint/'); // Update SWR cache
      }
      
      return result;
    } catch (error: unknown) {
      const apiError = error as ApiError;
      toast.error(apiError.response?.data?.message || 'Error message');
      throw error;
    }
  }
};

// Custom Hook
export const useFeature = () => {
  const { data, error, isLoading, mutate } = useSWR('/endpoint/', fetcher);
  
  return {
    feature: data?.data,
    isLoading,
    isError: error,
    mutate
  };
};
```

### Interface Definitions (MANDATORY)
```typescript
// interfaces/feature.ts
export interface Feature {
  id: number;
  name: string;
  created_at: string;
}

export interface CreateFeatureRequest {
  name: string;
}

export interface ApiResponse<T> {
  status: "success" | "error";
  status_code: number;
  data?: T;
  message?: string;
}
```

### File Naming (MANDATORY)
- **App folder**: kebab-case (`feature-name/page.tsx`)
- **Components folder**: kebab-case folder, kebab-case file (`feature-name/feature-name.tsx`)
- **Interfaces**: camelCase files (`feature.ts`)
- **Services**: camelCase files (`feature.ts`)

### Form Handling
- Use React Hook Form for all forms
- Zod for validation schemas
- Proper error handling and display
- TypeScript integration with form types

### State Management
- SWR for server state
- React hooks (useState, useEffect) for local state
- Proper loading and error states
- Optimistic updates where appropriate

### Styling Guidelines
- Use ShadCN UI components first
- Tailwind CSS for custom styling
- CSS modules or styled-components only when necessary
- Responsive design with Tailwind breakpoints
- Consistent spacing using Tailwind classes

### API Integration
- Use Axios for HTTP requests
- Proper error handling with try/catch
- Toast notifications for user feedback
- SWR for caching and revalidation
- TypeScript interfaces for all API responses

### Authentication
- JWT token storage in localStorage
- Automatic token injection in API requests
- Protected routes and authentication checks
- Proper logout and token cleanup

## Performance & Best Practices

### React Best Practices
- Use React.memo for expensive components
- Proper dependency arrays in useEffect
- Avoid unnecessary re-renders
- Use proper key props in lists

### Next.js Optimization
- Use Next.js Image component for images
- Proper metadata for SEO
- Client/server component separation
- Dynamic imports for code splitting

### Accessibility
- Proper ARIA labels and roles
- Keyboard navigation support
- Color contrast compliance
- Screen reader compatibility

## Error Handling
- Proper TypeScript error typing
- User-friendly error messages
- Error boundaries for component errors
- Network error handling

## Testing
- Unit tests for components using Jest/React Testing Library
- Integration tests for API services
- E2E tests for critical user flows
- Type checking with TypeScript

## Dependencies
- Keep package.json updated
- Use exact versions for critical dependencies
- Regular security updates
- Document any new dependencies

## Git Workflow
- Use descriptive commit messages
- Feature branches for new functionality
- Code review before merging
- Consistent commit message format 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yz-01) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
