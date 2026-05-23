## development

> - **Node.js**: Version 18 or higher

# 🛠️ Development Workflow & Tooling

## Development Environment

### Required Tools
- **Node.js**: Version 18 or higher
- **npm**: Package manager
- **Supabase CLI**: For database management
- **Git**: Version control

### Project Setup
```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Run linting
npm run lint
```

## Code Quality Tools

### ESLint Configuration
- **TypeScript Support**: Full TypeScript integration
- **React Hooks**: Enforces rules of hooks
- **React Refresh**: Hot reload optimization
- **Custom Rules**: Disabled unused vars for development flexibility

### TypeScript Configuration
```json
{
  "strictNullChecks": false,    // Relaxed for development
  "noImplicitAny": false,       // Allow implicit any
  "allowJs": true,              // Allow JavaScript files
  "skipLibCheck": true,         // Skip library type checking
  "noUnusedLocals": false,      // Allow unused locals
  "noUnusedParameters": false   // Allow unused parameters
}
```

## Build System

### Vite Configuration
- **Development Server**: Runs on port 8080
- **Hot Module Replacement**: Fast development experience
- **Path Aliases**: `@/` maps to `src/`
- **SWC Compilation**: Fast TypeScript compilation

### Build Optimization
```typescript
// Vite config optimizations
export default defineConfig({
  server: {
    host: "::",    // Allow external connections
    port: 8080,    // Consistent port
  },
  plugins: [
    react(),       // React support
    componentTagger() // Development tooling
  ],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src") // Path aliases
    }
  }
});
```

## Version Control

### Git Workflow
```bash
# Version management scripts
npm run version:patch    # Patch version (1.0.0 → 1.0.1)
npm run version:minor    # Minor version (1.0.0 → 1.1.0)
npm run version:major    # Major version (1.0.0 → 2.0.0)
npm run release         # Build and patch version
```

### Commit Guidelines
- **Conventional Commits**: Use standard commit message format
- **Descriptive Messages**: Clear description of changes
- **Atomic Commits**: One logical change per commit

## Database Management

### Supabase Integration
```bash
# Supabase CLI commands
supabase login                    # Login to Supabase
supabase link --project-ref <ref> # Link to project
supabase db push                  # Apply migrations
supabase gen types typescript     # Generate TypeScript types
```

### Migration Management
- **Migration Files**: Located in `supabase/migrations/`
- **Naming Convention**: `YYYYMMDDHHMMSS_description.sql`
- **Version Control**: All migrations tracked in Git

## Development Best Practices

### Hot Reload Optimization
```typescript
// Use React.memo for expensive components
const ExpensiveComponent = React.memo(() => {
  // Component logic
});

// Use useCallback for stable references
const handleClick = useCallback(() => {
  // Handler logic
}, [dependencies]);
```

### Development Debugging
```typescript
// Consistent logging format
console.log('Component mounted:', componentName);
console.log('API call:', { endpoint, params });
console.error('Error occurred:', { error, context });

// Development-only code
if (process.env.NODE_ENV === 'development') {
  console.log('Debug info:', debugData);
}
```

## Environment Configuration

### Environment Variables
```typescript
// Supabase configuration
const SUPABASE_URL = "https://grukqugorspbwsxqdhru.supabase.co";
const SUPABASE_ANON_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...";

// Development vs Production
const isDevelopment = process.env.NODE_ENV === 'development';
const isProduction = process.env.NODE_ENV === 'production';
```

### Build Modes
```bash
# Development build
npm run build:dev

# Production build
npm run build

# Preview build
npm run preview
```

## Testing Strategy

### Component Testing
- **Unit Tests**: Test individual components
- **Integration Tests**: Test component interactions
- **E2E Tests**: Test complete user workflows

### API Testing
```typescript
// Test API functions
const testApiCall = async () => {
  try {
    const result = await api.getData();
    expect(result).toBeDefined();
  } catch (error) {
    fail('API call should not throw error');
  }
};
```

## Performance Monitoring

### Bundle Analysis
```bash
# Analyze bundle size
npm run build
npx vite-bundle-analyzer dist
```

### Performance Metrics
- **First Contentful Paint**: Measure initial load time
- **Largest Contentful Paint**: Measure main content load
- **Cumulative Layout Shift**: Measure visual stability

## Deployment

### Production Build
```bash
# Create optimized production build
npm run build

# Preview production build locally
npm run preview
```

### Deployment Checklist
- [ ] All tests passing
- [ ] Build successful
- [ ] Environment variables configured
- [ ] Database migrations applied
- [ ] Performance metrics acceptable

## Troubleshooting

### Common Issues
1. **Build Failures**: Check TypeScript errors and dependencies
2. **Hot Reload Issues**: Restart development server
3. **Database Connection**: Verify Supabase credentials
4. **Import Errors**: Check path aliases and file extensions

### Debug Tools
```typescript
// React Developer Tools
// Supabase Dashboard
// Browser DevTools
// Network tab for API debugging
```

## Code Organization

### File Structure Standards
```
src/
├── components/     # Reusable components
├── hooks/         # Custom hooks
├── lib/           # Utilities and API
├── pages/         # Page components
├── types/         # TypeScript definitions
└── integrations/  # External services
```

### Import Organization
```typescript
// 1. React and external libraries
import React from 'react';
import { useNavigate } from 'react-router-dom';

// 2. Internal components and hooks
import { Button } from '@/components/ui/button';
import { useAuth } from '@/hooks/useAuth';

// 3. API services and utilities
import { problemApi } from '@/lib/api';

// 4. Type definitions
import type { Problem } from '@/types/database';
```

---
> Source: [hyunwook20230402/math-learning-management](https://github.com/hyunwook20230402/math-learning-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
