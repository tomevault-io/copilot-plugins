## cms-portfolio

> Provides sidebar navigation and consistent structure for authenticated pages. Use it with:

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **Canvasly**, a portfolio CMS built with Nuxt 4, Vue 3, and Supabase. It allows users to manage portfolio projects, blocks (project components), and social links through a dashboard interface.

## Development Commands

```bash
# Install dependencies
npm install

# Start development server (http://localhost:3000)
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Generate static site
npm generate
```

## Architecture Overview

### Stack
- **Frontend**: Nuxt 4.1.3 + Vue 3.5.22
- **Backend**: Nuxt Server Routes (H3 framework)
- **Database**: Supabase (PostgreSQL)
- **Auth**: Supabase Auth (JWT-based, httpOnly cookies)
- **State**: Pinia stores
- **UI**: shadcn-nuxt + Tailwind CSS
- **Storage**: Supabase Storage (for images)

### Directory Structure

```
/app                    # Frontend application
  ├── pages/           # File-based routes (auto-routed by Nuxt)
  ├── components/      # Vue components (auto-imported)
  ├── stores/          # Pinia state management
  ├── types/           # TypeScript interfaces
  ├── layouts/         # Page layouts
  └── lib/             # Utilities

/server                 # Backend API
  ├── api/            # REST API endpoints (file-based routing)
  └── utils/          # Server utilities (e.g., Supabase client)
```

## Data Models & Relationships

### Database Schema

```
User (Supabase Auth)
├── Projects (1:N)
│   ├── Blocks (1:N)
│   └── Categories (M:N via projectCategory)
└── Links (1:N)
```

**Key Tables**:
- `auth.users`: Managed by Supabase Auth
  - Custom field: `user_metadata.display_name`
- `project`: User's portfolio projects
  - Fields: `id`, `user_id`, `title`, `description`, `thumbnail`, timestamps
- `block`: Components/sections of a project
  - Fields: `id`, `project_id`, `title`, `description`, `url`
- `link`: Social/external links
  - Fields: `id`, `user_id`, `title`, `url`, `icon`
- `categories`: Project categories
- `projectCategory`: Junction table for M:N relationship

**Storage Buckets**:
- `projects`: Project thumbnail images
- `icons`: Link icon images

## Authentication System

### How It Works

1. **Registration/Login**: User submits credentials → API handler calls Supabase Auth → Returns JWT tokens
2. **Token Storage**: Access and refresh tokens stored in **httpOnly cookies** (`sb-access-token`, `sb-refresh-token`)
3. **API Protection**: All API routes (except auth endpoints) validate tokens via `_authGard.ts` middleware
4. **Session Management**: Tokens automatically refreshed by Supabase; cookies updated via auth state listener

### Auth Guard Pattern

**File**: [server/api/_authGard.ts](server/api/_authGard.ts)

The `_` prefix makes this a global middleware. It:
- Whitelists `/api/auth/login` and `/api/auth/register`
- Extracts tokens from cookies
- Validates with `supabase.auth.getUser()`
- Stores authenticated user in `event.context.auth`
- Returns 401 if unauthorized

**Usage in API routes**:
```typescript
import { authGuard } from '~/server/api/_authGard'

export default defineEventHandler(async (event) => {
  const user = await authGuard(event)  // Throws 401 if not authenticated
  // ... use user.id for queries
})
```

## API Structure & Conventions

### File-Based Routing

API routes use HTTP method as file extension:
- `index.get.ts` → GET
- `index.post.ts` → POST
- `index.patch.ts` → PATCH
- `index.delete.ts` → DELETE
- `[id]/index.get.ts` → GET with dynamic parameter

### Standard Patterns

**Reading Request Data**:
```typescript
import { readBody, readMultipartFormData, getRouterParam } from 'h3'

// JSON body
const body = await readBody(event)

// File uploads
const formData = await readMultipartFormData(event)

// Route params
const id = getRouterParam(event, 'id')
```

**Response Format**:
```typescript
// Success
return { data: result }

// Error
throw createError({
  statusCode: 400,
  message: 'User-friendly message'
})
```

**File Upload Pattern**:
```typescript
// 1. Read multipart form data
const formData = await readMultipartFormData(event)
const file = formData.find(f => f.name === 'thumbnail')

// 2. Upload to Supabase Storage
const { data, error } = await supabase.storage
  .from('bucket-name')
  .upload(`path/${file.filename}`, file.data)

// 3. Get public URL
const { data: { publicUrl } } = supabase.storage
  .from('bucket-name')
  .getPublicUrl(data.path)

// 4. Store URL in database
```

**Ownership Verification**:
Always filter by `user_id` to ensure users can only access their own resources:
```typescript
const { data } = await supabase
  .from('table')
  .select('*')
  .eq('user_id', user.id)  // Critical: prevents access to other users' data
```

## State Management (Pinia)

### Store Pattern

All stores follow this structure:
```typescript
import { defineStore } from 'pinia'

export const useStoreName = defineStore('storeName', {
  state: () => ({
    items: [],
    loading: false,
    error: null
  }),

  getters: {
    // Computed properties
  },

  actions: {
    // Use $fetch() for API calls (auto-includes cookies)
    async fetchItems() {
      this.loading = true
      try {
        const { data } = await $fetch('/api/endpoint')
        this.items = data
      } catch (err) {
        this.error = err.message
      } finally {
        this.loading = false
      }
    }
  }
})
```

### Available Stores

- **[app/stores/auth.ts](app/stores/auth.ts)**: User authentication (sign up, sign in, sign out, session)
- **[app/stores/projects.ts](app/stores/projects.ts)**: Project CRUD operations
- **[app/stores/links.ts](app/stores/links.ts)**: Social link CRUD operations

### Usage in Components

```typescript
const store = useStoreName()

// Access state
console.log(store.items)

// Call actions
await store.fetchItems()

// Reactive in template
<div v-if="store.loading">Loading...</div>
```

## Frontend Patterns

### Page Structure

**Public pages**: [app/pages/index.vue](app/pages/index.vue), [app/pages/account/sign-in.vue](app/pages/account/sign-in.vue), [app/pages/account/sign-up.vue](app/pages/account/sign-up.vue)

**Protected pages** (dashboard): All pages under [app/pages/dashboard/](app/pages/dashboard/) use the dashboard layout.

### Dashboard Layout

**File**: [app/layouts/dashboard.vue](app/layouts/dashboard.vue)

Provides sidebar navigation and consistent structure for authenticated pages. Use it with:
```vue
<template>
  <NuxtLayout name="dashboard">
    <template #header>
      <DashboardPageHeader title="Page Title" />
    </template>

    <!-- Page content -->
  </NuxtLayout>
</template>
```

### Component Auto-Import

Nuxt auto-imports components from [app/components/](app/components/). No need to import them manually:
```vue
<template>
  <AppSidebar />  <!-- Auto-imported -->
  <Button />      <!-- Auto-imported from ui/ -->
</template>
```

### Form Patterns

**File Upload with Preview**:
```typescript
const previewUrl = ref<string | null>(null)

const handleFileChange = (event: Event) => {
  const file = (event.target as HTMLInputElement).files?.[0]
  if (!file) return

  formData.thumbnail = file

  // Create preview
  const reader = new FileReader()
  reader.onload = (e) => {
    previewUrl.value = e.target?.result as string
  }
  reader.readAsDataURL(file)
}

// Submit as FormData
const submit = async () => {
  const data = new FormData()
  data.append('title', formData.title)
  data.append('thumbnail', formData.thumbnail)

  await $fetch('/api/projects', {
    method: 'POST',
    body: data
  })
}
```

**Standard Form Submission**:
```typescript
const handleSubmit = async () => {
  store.clearError()

  // Client-side validation
  if (!isValid.value) {
    store.error = 'Validation message'
    return
  }

  try {
    await store.action(formData)
    // Navigate or show success
  } catch (error) {
    // Error already set in store
  }
}
```

## Environment Configuration

Create a `.env` file in the root:

```bash
NUXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NUXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
```

**Important**:
- `NUXT_PUBLIC_*` variables are exposed to the client
- `SUPABASE_SERVICE_ROLE_KEY` is server-only and has admin privileges
- Never commit `.env` to version control

## Supabase Integration

### Client Creation

**Client-side** ([app/plugins/supabase.client.ts](app/plugins/supabase.client.ts)):
```typescript
// Uses public anon key (limited permissions)
const supabase = createClient(supabaseUrl, supabaseAnonKey)
```

**Server-side** ([server/utils/supabase.server.ts](server/utils/supabase.server.ts)):
```typescript
// Uses service role key (full privileges)
const supabase = createClient(supabaseUrl, supabaseServiceRoleKey)
```

### Usage in API Routes

```typescript
import { getSupabaseClient } from '~/server/utils/supabase.server'

export default defineEventHandler(async (event) => {
  const supabase = getSupabaseClient(event)

  // Query with RLS (Row Level Security)
  const { data, error } = await supabase
    .from('projects')
    .select('*')
    .eq('user_id', user.id)

  if (error) throw createError({ statusCode: 500, message: error.message })

  return { data }
})
```

## Known Issues

1. **Links Store Bug** ([app/stores/links.ts:51](app/stores/links.ts#L51)):
   - `formData.append('icon', newLink.icons)` should be `newLink.icon` (typo)

2. **Missing User Filter** ([server/api/links/index.get.ts](server/api/links/index.get.ts)):
   - Currently fetches all links without filtering by `user_id`
   - Should add `.eq('user_id', user.id)` for security

3. **No Client-Side Route Protection**:
   - Dashboard pages are not protected by middleware
   - Users can access `/dashboard` routes before authentication
   - Consider adding [app/middleware/auth.ts](app/middleware/auth.ts) to redirect unauthenticated users

## Key Conventions

### Naming
- **Components**: PascalCase (e.g., `AppSidebar.vue`)
- **Composables**: `use` prefix (e.g., `useAuthStore()`)
- **API routes**: HTTP method suffix (e.g., `index.post.ts`)
- **Types**: Named interfaces in [app/types/](app/types/)

### Code Style
- TypeScript strict mode enabled
- ESLint configured via `@nuxt/eslint`
- Prefer composition API over options API
- Use `$fetch()` for API calls (not `fetch()` or `axios`)

### Error Handling
- Frontend: Store error state, display in UI
- Backend: Use `createError()` from H3
- Always clear previous errors before new actions

## Working with This Codebase

### Adding a New API Endpoint

1. Create file in [server/api/](server/api/) with appropriate path and method suffix
2. Import and call `authGuard(event)` if authentication is required
3. Use Supabase client to interact with database
4. Always filter by `user_id` to prevent unauthorized access
5. Return `{ data }` format for consistency

### Adding a New Page

1. Create file in [app/pages/](app/pages/) (auto-routed by Nuxt)
2. For dashboard pages, wrap with `<NuxtLayout name="dashboard">`
3. Import and use relevant Pinia store
4. Handle loading and error states

### Adding a New Store

1. Create file in [app/stores/](app/stores/)
2. Follow the pattern: state, getters, actions
3. Use `$fetch()` for API calls
4. Manage loading and error states
5. Clear errors before new actions

### File Uploads

1. Frontend: Use `<input type="file">` and FormData
2. Backend: Read with `readMultipartFormData(event)`
3. Upload to Supabase Storage bucket
4. Get public URL and store in database
5. Return URL to frontend for display

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kicksbld) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
