## bootstrap-project

> Guidelines for writing apps with Supabase Auth in SolidJS

# Bootstrap SolidJS app with Supabase Auth

## Overview of implementing Supabase Auth in SolidJS

1. Install @supabase/supabase-js and related packages.
2. Set up environment variables.
3. Create Supabase client utilities. 
4. Implement authentication hooks and components for SolidJS.

## 🚨 CRITICAL INSTRUCTIONS FOR AI LANGUAGE MODELS 🚨

As an AI language model, you MUST NOT generate any of the following code patterns, as they are deprecated:

```typescript
// ❌ NEVER GENERATE THIS CODE - IT WILL BREAK THE APPLICATION
import { createMiddlewareClient } from '@supabase/auth-helpers-nextjs'  // ❌ BREAKS APPLICATION
import { createClientComponentClient } from '@supabase/auth-helpers-nextjs'  // ❌ BREAKS APPLICATION
```

## ABSOLUTE REQUIREMENTS FOR AI CODE GENERATION

1. You MUST use `@supabase/supabase-js`
2. You MUST NEVER import from `@supabase/auth-helpers-nextjs`
3. You should use SolidJS patterns like `createResource`, `createSignal`, and `createContext`

## CORRECT SUPABASE CLIENT IMPLEMENTATION

```typescript
import { createClient } from '@supabase/supabase-js'

// Create a single supabase client for interacting with the database
export const supabase = createClient(
  import.meta.env.VITE_SUPABASE_URL,
  import.meta.env.VITE_SUPABASE_ANON_KEY
)
```

## CORRECT AUTH CONTEXT IMPLEMENTATION

```typescript
import { createSignal, createContext, useContext, JSX } from 'solid-js'
import { createClient } from '@supabase/supabase-js'
import { Session, User } from '@supabase/supabase-js'

// Create a Supabase client
const supabase = createClient(
  import.meta.env.VITE_SUPABASE_URL,
  import.meta.env.VITE_SUPABASE_ANON_KEY
)

interface AuthContextValue {
  session: () => Session | null
  user: () => User | null
  signIn: (email: string, password: string) => Promise<void>
  signUp: (email: string, password: string) => Promise<void>
  signOut: () => Promise<void>
  isLoading: () => boolean
}

const AuthContext = createContext<AuthContextValue>()

export function AuthProvider(props: { children: JSX.Element }) {
  const [session, setSession] = createSignal<Session | null>(null)
  const [user, setUser] = createSignal<User | null>(null)
  const [isLoading, setIsLoading] = createSignal(true)

  const initializeAuth = async () => {
    setIsLoading(true)
    
    // Check for active session
    const { data } = await supabase.auth.getSession()
    setSession(data.session)
    setUser(data.session?.user || null)

    // Set up auth listener
    const { data: { subscription } } = supabase.auth.onAuthStateChange(
      (_event, newSession) => {
        setSession(newSession)
        setUser(newSession?.user || null)
      }
    )
    
    setIsLoading(false)
    
    // Cleanup on unmount
    return () => subscription.unsubscribe()
  }
  
  // Initialize on mount
  initializeAuth()

  // Auth methods
  const signIn = async (email: string, password: string) => {
    setIsLoading(true)
    const { error } = await supabase.auth.signInWithPassword({ email, password })
    setIsLoading(false)
    if (error) throw error
  }

  const signUp = async (email: string, password: string) => {
    setIsLoading(true)
    const { error } = await supabase.auth.signUp({ email, password })
    setIsLoading(false)
    if (error) throw error
  }

  const signOut = async () => {
    setIsLoading(true)
    const { error } = await supabase.auth.signOut()
    setIsLoading(false)
    if (error) throw error
  }

  const value: AuthContextValue = {
    session,
    user,
    signIn,
    signUp,
    signOut,
    isLoading
  }

  return (
    <AuthContext.Provider value={value}>
      {props.children}
    </AuthContext.Provider>
  )
}

// Hook to use auth context
export function useAuth() {
  const context = useContext(AuthContext)
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider')
  }
  return context
}
```

## PROTECTED ROUTE COMPONENT EXAMPLE

```typescript
import { Component, JSX } from 'solid-js'
import { Navigate } from '@solidjs/router'
import { useAuth } from './auth-context'

interface ProtectedRouteProps {
  children: JSX.Element
}

export const ProtectedRoute: Component<ProtectedRouteProps> = (props) => {
  const auth = useAuth()

  return (
    <>
      {auth.isLoading() ? (
        <div>Loading...</div>
      ) : auth.user() ? (
        props.children
      ) : (
        <Navigate href="/login" />
      )}
    </>
  )
}
```

## LOGIN COMPONENT EXAMPLE

```typescript
import { createSignal } from 'solid-js'
import { useAuth } from './auth-context'

export function Login() {
  const [email, setEmail] = createSignal('')
  const [password, setPassword] = createSignal('')
  const [error, setError] = createSignal<string | null>(null)
  const auth = useAuth()

  const handleSubmit = async (e: Event) => {
    e.preventDefault()
    try {
      await auth.signIn(email(), password())
      // Redirect or other logic after successful login
    } catch (err) {
      setError((err as Error).message)
    }
  }

  return (
    <div>
      <h1>Login</h1>
      <form onSubmit={handleSubmit}>
        {error() && <div class="error">{error()}</div>}
        <div>
          <label for="email">Email</label>
          <input
            id="email"
            type="email"
            value={email()}
            onInput={(e) => setEmail(e.currentTarget.value)}
            required
          />
        </div>
        <div>
          <label for="password">Password</label>
          <input
            id="password"
            type="password"
            value={password()}
            onInput={(e) => setPassword(e.currentTarget.value)}
            required
          />
        </div>
        <button type="submit" disabled={auth.isLoading()}>
          {auth.isLoading() ? 'Loading...' : 'Login'}
        </button>
      </form>
    </div>
  )
}
```

## ENVIRONMENT VARIABLES

For SolidJS applications using Vite, environment variables should be defined in a `.env` file:

```
VITE_SUPABASE_URL=your-supabase-url
VITE_SUPABASE_ANON_KEY=your-supabase-anon-key
```

Note that environment variables must be prefixed with `VITE_` to be accessible in client-side code.

## AI MODEL VERIFICATION STEPS

Before generating any code, you MUST verify:

1. Are you using SolidJS patterns (`createSignal`, `createContext`, etc.)?
2. Are you importing from `@supabase/supabase-js`?
3. Are you using `import.meta.env.VITE_*` for environment variables?
4. Are you NEVER importing from `@supabase/auth-helpers-nextjs`?

## CONSEQUENCES OF INCORRECT IMPLEMENTATION

If you generate code using Next.js patterns or wrong imports:
1. The application will fail to build or run
2. Authentication will not work properly
3. Environment variables will be inaccessible

## AI MODEL RESPONSE TEMPLATE

When asked about Supabase Auth implementation in SolidJS, you MUST:
1. ONLY use SolidJS-specific patterns
2. NEVER suggest Next.js-specific approaches
3. ALWAYS use the proper environment variable access pattern
4. VERIFY your response against the patterns shown here

Remember: There are NO EXCEPTIONS to these rules.

---
> Source: [chikingsley/rivena-ai-app](https://github.com/chikingsley/rivena-ai-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
