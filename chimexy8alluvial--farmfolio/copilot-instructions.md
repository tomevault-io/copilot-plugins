## farmfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Farmfolio is a React SPA that connects food producers with conscious consumers through digital storytelling and a marketplace discovery platform. Producers create story-based profiles using customizable templates, which can be shared via QR codes at physical locations.

## Development Commands

```bash
npm run dev      # Start Vite dev server (port 3000, auto-opens)
npm run build    # Build for production (outputs to dist/ with sourcemaps)
npm run preview  # Preview production build locally
npm run lint     # Run ESLint
```

## Architecture Overview

### Core Stack
- **React 18** + **Vite** as the build tool
- **React Router** for client-side routing ([src/App.jsx](src/App.jsx))
- **TailwindCSS** with custom color schemes (farmfolio green theme)
- **React Query** (@tanstack/react-query) for server state management
  - Configured in [src/main.jsx](src/main.jsx) with retry=2, staleTime=5min
- **Axios** for HTTP requests with global interceptors
- **Formik** + **Yup** for form handling and validation
- **Framer Motion** for animations

### State Management
- **Authentication**: Context-based ([src/context/AuthContext.jsx](src/context/AuthContext.jsx)) with localStorage persistence
  - Access via `useAuth()` hook ([src/hooks/useAuth.js](src/hooks/useAuth.js))
  - Token and user data stored in localStorage (keys defined in constants)
- **Other state**: React Query for server state, local component state for UI

### API Communication
- Centralized axios instance in [src/api/axiosConfig.js](src/api/axiosConfig.js)
- Service modules in `src/api/` (authApi, storiesApi, producerProfilesApi, etc.)
- Request interceptor: Automatically attaches `Authorization: Bearer <token>`
- Response interceptor: Handles 401 (clear auth + redirect), 400 (validation), 404

### Authentication Flow
- Token/user stored in localStorage with keys from `STORAGE_KEYS` constant
- Auth interceptor defensively handles multiple API response shapes:
  - `{ token, user }` or `{ data: { token, user } }`
  - Falls back to JWT decoding if only token is returned
  - See [src/context/AuthContext.jsx](src/context/AuthContext.jsx) lines 56-110 for robust handling
- Protected routes use `PrivateRoute` wrapper component

### Template System (Core Feature)
Farmfolio uses a template-driven approach for creating producer profiles and stories:

- **Story Templates**: Predefined layouts for farm stories, artisan products, sustainability narratives
  - Config: [src/templates/templateConfig.js](src/templates/templateConfig.js)
  - Backend templates: [src/templates/storyTemplatesForBackend.js](src/templates/storyTemplatesForBackend.js)
  - Users select template → customize content → publish as story

- **Producer Profile Templates**: Similar system for producer marketplace profiles
  - Backend templates: [src/templates/producerProfileTemplatesForBackend.js](src/templates/producerProfileTemplatesForBackend.js)

- **Inline Editing**: Recent feature allowing direct template editing
- **Admin Upload**: [src/pages/SimpleAdminPage.jsx](src/pages/SimpleAdminPage.jsx) provides interface to upload new templates

### Routing Structure
Routes defined in [src/utils/constants.js](src/utils/constants.js) under `ROUTES`:
- **Public**: `/marketplace`, `/producer/:id`, `/story/:id` (story viewer)
- **Auth**: `/login`, `/register` (redirect if authenticated)
- **Protected**: `/dashboard`, `/story/create`, `/story/edit/:id`, `/profile/edit`, `/admin`
- Root `/` redirects to dashboard (auth) or login (guest)

### Key Pages
- **MarketplacePage**: Searchable grid + Google Maps view of producers (filters by category, location)
- **ProducerProfilePage**: Public-facing producer profile (can be template-based or custom)
- **DashboardPage**: Producer's control center for managing stories and profile
- **EditProfilePage**: Edit producer profile using template system
- **StoryViewer**: Public story view (accessed via QR code or direct link)
- **TemplateSelector** → **StoryEditor** → **QRCodeGenerator**: Story creation flow

## Important Patterns & Conventions

### API Service Pattern
Create small service modules in `src/api/` that export functions using the shared axios instance:
```js
// Example from authApi.js
import axiosInstance from './axiosConfig';
export const login = async (credentials) => {
  const response = await axiosInstance.post(API_ENDPOINTS.LOGIN, credentials);
  return response.data;
};
```

Service calls return `response.data`. Callers should handle multiple possible response shapes (see AuthContext for defensive patterns).

### File Uploads
Use the `createFormData` helper from [src/api/axiosConfig.js](src/api/axiosConfig.js):
- Properly handles arrays (e.g., `CategoryIds`) and files
- Automatically removes `Content-Type` header for FormData requests
- Example: `const formData = createFormData(data, filesArray);`

### Adding Routes
1. Add route constant to `ROUTES` in [src/utils/constants.js](src/utils/constants.js)
2. Add endpoint to `API_ENDPOINTS` if backend route needed
3. Create route in [src/App.jsx](src/App.jsx)
4. Wrap with `<PrivateRoute>` if authentication required

### Error Handling
- API errors follow shape: `{ message, errors?, originalError }`
- Common error messages defined in `ERROR_MESSAGES` constant
- Toast notifications via `react-hot-toast` (configured in [src/main.jsx](src/main.jsx))

### Environment Variables
- **Prefix**: Must use `REACT_APP_` prefix (configured in [vite.config.js](vite.config.js))
- Access via `import.meta.env.REACT_APP_*`
- All env vars consolidated in [src/utils/constants.js](src/utils/constants.js) for type-safe access
- Key vars: `REACT_APP_API_BASE_URL`, `REACT_APP_GOOGLE_MAPS_API_KEY`, feature flags

### Styling Conventions
- Custom Tailwind colors: `farmfolio-green`, `farmfolio-darkgreen`, `farmfolio-cream`, `farmfolio-brown`
- Custom animations: `animate-fade-in`, `animate-slide-up`, `animate-pulse-soft`
- Forms use `@tailwindcss/forms` plugin
- Rich text uses `@tailwindcss/typography` plugin

## Common Pitfalls

### 1. API Response Shape Handling
Backend may return `{ token, user }`, `{ data: { token, user } }`, or just `{ token }`. Always handle multiple shapes or decode JWT as fallback (see [src/context/AuthContext.jsx](src/context/AuthContext.jsx) lines 56-110).

### 2. FormData Requests
When uploading files:
- Use `createFormData` helper for correct array/file handling
- Don't manually set `Content-Type` — axios interceptor removes it
- Endpoint must expect `multipart/form-data`

### 3. 401 Redirect Loop
Axios interceptor auto-redirects to `/login` on 401 and clears localStorage. Avoid retry loops by checking `_retry` flag (already implemented in [src/api/axiosConfig.js](src/api/axiosConfig.js)).

### 4. useAuth Outside Provider
`useAuth()` throws error if used outside `AuthProvider`. Always ensure component is wrapped by provider tree (configured in [src/main.jsx](src/main.jsx)).

### 5. Environment Variable Not Exposed
Vite only exposes vars with `REACT_APP_` prefix. Check [vite.config.js](vite.config.js) `envPrefix` setting.

## Google Maps Integration

- API key in `.env` as `REACT_APP_GOOGLE_MAPS_API_KEY`
- MapView component: [src/components/marketplace/MapView.jsx](src/components/marketplace/MapView.jsx)
- Uses `@react-google-maps/api` library with marker clustering
- Default center: UK (lat: 54.5, lng: -3.5), zoom: 6
- Clustering config in `MAPS_CONFIG` constant

## Deployment

- Hosted on **Vercel** (SPA routing configured in [vercel.json](vercel.json))
- Backend API on **Azure** (farmfoliodev.azurewebsites.net)
- Media storage on **Azure Blob Storage** with CDN
- Build outputs to `dist/` with sourcemaps enabled

## Path Alias

`@` is aliased to `./src` in [vite.config.js](vite.config.js). Use `import { foo } from '@/utils/bar'` instead of relative paths.

## Testing Notes

No test runner configured yet. When adding tests:
- Consider Vitest (native Vite support) or Jest
- Prioritize testing API service functions and auth flow
- Mock axios instance for API tests

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chimexy8alluvial)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chimexy8alluvial)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
