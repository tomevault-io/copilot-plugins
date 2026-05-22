## supabase-client-management

> Based on [src/utils/supabase/client.ts](mdc:src/utils/supabase/client.ts):

# Supabase Client Management & Authentication (MANDATORY)

## 🔐 **Supabase Client Pattern (CRITICAL, SSR-aware)**

Based on [src/utils/supabase/client.ts](mdc:src/utils/supabase/client.ts):

- **ALWAYS** use singleton pattern for Supabase client creation
- **NEVER** create multiple Supabase client instances
- **MUST** export a single client instance from the module
- **ALWAYS** use `createClient()` function to get the singleton instance

```typescript
// ✅ CORRECT - Browser (SSR) client singleton
import { createBrowserClient } from '@supabase/ssr';
const supabase = createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
);
export const createClient = () => supabase;

// ✅ CORRECT - Server/Edge client per-request
// Use createServerClient from '@supabase/ssr' inside handlers/middleware
// with cookies.getAll() and cookies.setAll() implementations.
```

## 🚫 **FORBIDDEN Supabase Practices**

- **NEVER** use `useState` to create Supabase client in React components
- **NEVER** create new client instances in useEffect or event handlers
- **NEVER** import `createClient` from `@supabase/supabase-js` directly in components
- **NEVER** create multiple GoTrueClient instances in the same browser context
- **NEVER** send access/refresh tokens in API bodies for "session sync"

## ✅ **REQUIRED Authentication Patterns**

Based on [src/hooks/useAuth.ts](mdc:src/hooks/useAuth.ts):

- **ALWAYS** use React Query for session and profile management
- **ALWAYS** use Zustand for local authentication state
- **NEVER** mix React Query and Zustand for the same data
- **ALWAYS** implement proper error handling for authentication failures

```typescript
// ✅ CORRECT - Proper authentication hook usage
export function useAuth() {
    const { isLoading, setLoading, clearAuth } = useAuthStore();

    // React Query for server state
    const { data: session } = useQuery({
        queryKey: ['auth', 'session'],
        queryFn: async () => {
            const supabase = createClient(); // Browser SSR singleton
            const {
                data: { session },
            } = await supabase.auth.getSession();
            return session;
        },
        staleTime: 60 * 1000,
        refetchOnWindowFocus: false,
    });

    // Zustand for local state
    const signOut = useCallback(async () => {
        const supabase = createClient(); // Use singleton
        await supabase.auth.signOut();
        clearAuth();
    }, [clearAuth]);
}
```

## 🏗️ **Provider Architecture (CURRENT STATE)**

Based on [src/components/providers/supabase-provider.tsx](mdc:src/components/providers/supabase-provider.tsx):

- **CURRENTLY**: SupabaseProvider is temporarily disabled
- **REASON**: Multiple GoTrueClient instances issue resolved with singleton pattern
- **FUTURE**: Will be reimplemented when authentication system is complete
- **NEVER** enable SupabaseProvider until authentication system is fully implemented

```typescript
// ✅ CORRECT - Current temporary implementation
export function SupabaseProvider({ children }: { children: React.ReactNode }) {
    return (
        <SupabaseContext.Provider value={{ supabase: null }}>
            {children}
        </SupabaseContext.Provider>
    );
}

// ❌ WRONG - Don't enable until authentication system is complete
// <SupabaseProvider> // This is currently disabled
//     {children}
// </SupabaseProvider>
```

## 📁 **File Organization for Supabase**

- **ALWAYS** keep Supabase client in [src/utils/supabase/client.ts](mdc:src/utils/supabase/client.ts)
- **ALWAYS** use `createClient()` function from this file
- **NEVER** create Supabase clients in other files
- **ALWAYS** import configuration from the same file

```typescript
// ✅ CORRECT - Import from centralized location
import { createClient, supabaseConfig } from '@/utils/supabase/client';

// Use the singleton instance
const supabase = createClient();

// ❌ WRONG - Creating client elsewhere
import { createClient } from '@supabase/supabase-js';
const supabase = createClient(/* ... */);
```

## 🔧 **Configuration Management**

Based on [src/utils/supabase/client.ts](mdc:src/utils/supabase/client.ts):

- **ALWAYS** define Supabase configuration in the same file as the client
- **ALWAYS** use consistent naming for configuration objects
- **NEVER** hardcode configuration values in components
- **ALWAYS** export configuration for reuse

```typescript
// ✅ CORRECT - Centralized configuration
export const supabaseConfig = {
    storageBucket: 'files',
    maxFileSize: 50 * 1024 * 1024, // 50MB
    allowedImageTypes: ['image/jpeg', 'image/png', 'image/webp', 'image/gif'],
    maxFiles: 20,
};

// ❌ WRONG - Scattered configuration
const config = { bucket: 'files' }; // In component file
```

## 🚨 **Error Prevention**

- **ALWAYS** check for environment variables before client creation
- **ALWAYS** handle authentication errors gracefully
- **NEVER** expose sensitive configuration in client-side code
- **ALWAYS** implement proper error boundaries for authentication failures

## 📚 **Best Practices Summary**

1. **Browser Client**: Use `createBrowserClient` singleton in `src/utils/supabase/client.ts`
2. **Server/Edge Client**: Use `createServerClient` per-request with cookies getAll/setAll
3. **Callback Handling**: Use Route Handler + `exchangeCodeForSession` (httpOnly cookies)
4. **Middleware Rule**: Always call `auth.getUser()` to trigger auto-refresh and sync cookies
5. **Protected Routes**: Gate `/admin`, `/profile` in middleware; redirect unauthenticated users
6. **No Token Sync API**: Do not send tokens via body/headers to sync sessions
7. **Authentication Separation**: React Query for server state, Zustand for local state
8. **Error Handling**: Implement proper error boundaries and user feedback

description: Supabase client management and authentication patterns to prevent Multiple GoTrueClient instances
globs: ["**/*.ts", "**/*.tsx"]
alwaysApply: true

---
> Source: [kimyoungyin/myblog](https://github.com/kimyoungyin/myblog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
