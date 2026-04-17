## mindqna-admin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is "mindBridge-admin", a Next.js 13 based admin panel application built with TypeScript, Ant Design, and Tailwind CSS. It's a Korean admin interface for managing various game/social platform features including users, assets, products, games, and analytics.

## Development Commands

```bash
# Development server (runs on port 4000)
npm run dev

# Build application
npm run build

# Production server
npm run start

# Lint code
npm run lint

# Static export
npm run export
```

## Architecture

### Tech Stack
- **Framework**: Next.js 13 with TypeScript
- **UI Libraries**: Ant Design + Tailwind CSS + Radix UI components
- **State Management**: Zustand for global state, TanStack Query for server state
- **Authentication**: NextAuth.js with JWT tokens and refresh token rotation
- **HTTP Client**: Axios with request/response interceptors for automatic token handling
- **Charts**: Chart.js with react-chartjs-2
- **Editor Components**: Quill.js and CodeMirror

### Key Configuration
- **Port**: Development server runs on port 4000 (not default 3000)
- **TypeScript**: `ignoreBuildErrors: true` in next.config.js
- **Tailwind**: `preflight: false` to avoid conflicts with Ant Design
- **API Proxy**: `/s3-proxy/*` routes to `https://mindqna.s3.amazonaws.com/*`

### Authentication Flow
1. Login via credentials through NextAuth.js (`/api/auth/[...nextauth].ts`)
2. JWT tokens stored in session with automatic refresh
3. Axios interceptors handle token attachment and refresh logic
4. 401 responses trigger automatic logout and redirect to `/login`
5. AuthProvider component enforces authentication on all pages except `/login`

### Directory Structure

```
src/
├── client/           # API client functions (one file per domain)
│   ├── @base.ts     # Axios instance with auth interceptors
│   ├── assets.ts    # Asset management APIs
│   ├── user.ts      # User management APIs
│   └── ...
├── components/
│   ├── layout/      # Layout components (sidebar, header, navigation)
│   ├── page/        # Page-specific components organized by route
│   │   ├── assets/  # Components for /product/assets page
│   │   ├── user/    # Components for /user/list page
│   │   └── ...
│   ├── shared/      # Reusable components
│   └── ui/          # Base UI components (shadcn/ui style)
├── hooks/           # Custom React hooks (one per domain)
├── lib/             # Utilities and providers
├── pages/           # Next.js pages (file-based routing)
├── styles/          # Global CSS and Ant Design customizations
└── types/           # TypeScript type definitions
```

### Component Organization
- **Pages**: Minimal logic, import components from `components/page/[route]`
- **API Clients**: Organized by domain in `src/client/`, using TanStack Query hooks in `src/hooks/`
- **Forms**: Heavily use Ant Design Form components with custom wrapper components
- **Tables**: Use custom `DefaultTable` component wrapping Ant Design Table
- **Modals**: Use custom `DefaultModal` component for consistent styling

### Key Patterns

**API Client Pattern**:
```typescript
// src/client/example.ts
export const getExamples = () => client.get('/examples');
export const createExample = (data) => client.post('/examples', data);

// src/hooks/useExamples.ts  
export const useExamples = () => useQuery(['examples'], getExamples);
```

**Page Component Pattern**:
```typescript
// src/pages/example/list.tsx
const ExampleListPage = () => <ExampleList />;
ExampleListPage.getLayout = getDefaultLayout;
export default ExampleListPage;

// src/components/page/example/ExampleList.tsx
export default function ExampleList() { /* implementation */ }
```

**Form Pattern**:
```typescript
// Uses Ant Design Form with custom FormSection/FormGroup wrappers
<DefaultForm form={form} onFinish={onFinish}>
  <FormSection title="기본 정보">
    <FormGroup title="제목" required>
      <Form.Item name="title" rules={[{ required: true }]}>
        <Input />
      </Form.Item>
    </FormGroup>
  </FormSection>
</DefaultForm>
```

### Custom Components
- `DefaultTable`: Enhanced Ant Design Table with consistent styling
- `DefaultModal`: Modal wrapper with consistent styling and behavior
- `FormSection`/`FormGroup`: Form layout helpers for consistent form structure
- `DefaultForm`: Form wrapper with standard configuration
- `CodeMirrorEditor`/`QuillEditor`: Rich text editing components

### State Management
- **Server State**: TanStack Query for API data caching and synchronization
- **Client State**: Zustand for global application state
- **Form State**: Ant Design Form for form-specific state

### Important Notes
- All pages require authentication except `/login`
- Korean language interface with `koKR` locale for Ant Design
- Asset management includes S3 integration via proxy
- Development uses Pretendard font family
- TypeScript errors ignored in production builds for faster deployment

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/prestcompany) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
