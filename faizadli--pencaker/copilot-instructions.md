## pencaker

> > **ADIKARA**: Aplikasi Data dan Informasi Ketenagakerjaan Area Regional Paser - a job portal and employment services platform.

# GEMINI.md - AI Coding Guide for ADIKARA/Pencaker Frontend

> **ADIKARA**: Aplikasi Data dan Informasi Ketenagakerjaan Area Regional Paser - a job portal and employment services platform.

## Tech Stack

- **Framework**: Next.js 16 (App Router) with React 19
- **Styling**: TailwindCSS v4 with custom theme in `app/globals.css`
- **Validation**: Zod v4 (schemas in `utils/zod-schemas.ts`)
- **Icons**: [Remix Icons](https://remixicon.com/) via CDN (`ri-*` classes)
- **Rich Text**: TipTap editor (`@tiptap/react`)
- **Language**: TypeScript with strict mode

## Project Structure

```
app/                    # Next.js App Router pages
├── dashboard/          # Authenticated dashboard (candidate/company/disnaker)
├── jobs/               # Public job listings
├── login/              # Auth pages
├── register/           # Registration flow
├── globals.css         # Tailwind config + custom design tokens
└── layout.tsx          # Root layout with Navbar, Footer, SiteProvider

components/
├── ui/                 # Reusable UI (Input, Modal, Table, Pagination, etc.)
├── layout/             # Navbar, Footer, Sidebar
└── laporan/            # Report generation components

services/               # API service layer (all external API calls)
utils/                  # Zod schemas, formatters, export utilities
context/                # React contexts (SiteContext for site settings)
```

## Development Commands

```bash
npm run dev        # Start dev server (localhost:3000)
npm run build      # Production build
npm run lint       # ESLint check
npm run typecheck  # TypeScript type check
npm run lint:fix   # Auto-fix lint issues
```

## Key Patterns

### 1. Service Layer Pattern

All API calls go through `services/*.ts`. Each service:

- Uses `BASE` URL from `NEXT_PUBLIC_API_BASE_URL`
- Uses `authHeader()` helper for JWT auth via localStorage
- Returns typed responses with error handling

```typescript
// services/jobs.ts pattern
const BASE = process.env.NEXT_PUBLIC_API_BASE_URL || "http://localhost:4000";
function authHeader(): Record<string, string> {
  const token = localStorage.getItem("token") || "";
  return token ? { Authorization: `Bearer ${token}` } : {};
}

export async function listJobs(params?: {
  company_id?: string;
  status?: string;
}) {
  const resp = await fetch(`${BASE}/api/jobs?${new URLSearchParams(params)}`, {
    headers: { ...authHeader() },
  });
  if (!resp.ok) throw new Error("Gagal mengambil jobs");
  return resp.json();
}
```

### 2. Form Components with Validation UX

UI components in `components/ui/field.tsx` support:

- `label`, `hint`, `error` props for consistent form UX
- `submitted` prop to show validation only after form submit
- `required` defaults to true when `label` is provided

```tsx
<Input
  label="Nama Lengkap"
  value={name}
  onChange={(e) => setName(e.target.value)}
  error={errors.name}
  submitted={submitted}
/>
```

### 3. Zod Schema Validation

Form schemas in `utils/zod-schemas.ts` follow patterns:

- All messages in Indonesian ("wajib diisi", "tidak valid")
- File validation with size/type checks
- Reusable schemas via `.extend()`

```typescript
export const candidateProfileSchema = z.object({
  full_name: z.string().min(1, "Nama Lengkap wajib diisi"),
  nik: z
    .string()
    .length(16, "NIK harus 16 digit")
    .regex(/^\d+$/, "NIK hanya boleh angka"),
  photo: fileSchema(IMAGE_TYPES, MAX_IMAGE_SIZE, MIN_FILE_SIZE, "Foto Profil"),
});
```

### 4. Dashboard with RBAC

Dashboard layout (`app/dashboard/layout.tsx`) handles:

- Session validation via `getUserById()`
- Role-based permissions fetched from `getRolePermissions()`
- SidebarData passed to `Sidebar` component

Roles: `candidate`, `company`, `disnaker`

### 5. Design Tokens

Custom colors defined in `app/globals.css`:

```css
--primary: #419823; /* Green */
--secondary: #f4d348; /* Yellow */
--accent: #f66338; /* Orange */
--danger: #dc2626; /* Red */
```

Use via Tailwind: `text-primary`, `bg-secondary`, `border-accent`

## Conventions

1. **Path Alias**: Use `@/*` for imports (e.g., `@/services/auth`)
2. **Indonesian UI**: All user-facing text in Indonesian
3. **Error Messages**: Format as "Gagal [action]" (e.g., "Gagal mengambil data")
4. **API Response**: Expect `{ data: ... }` or `{ message: string }` envelopes
5. **Icons**: Use Remix Icons with `ri-` prefix: `<i className="ri-user-line" />`
6. **No SSR for Auth**: Dashboard pages are `"use client"` with client-side auth checks

## Common Tasks

### Adding a New Dashboard Page

1. Create page in `app/dashboard/[feature]/page.tsx`
2. Mark as `"use client"` if needs auth/interactivity
3. Use existing UI components from `components/ui/`
4. Add route to Sidebar menu items in `components/layout/Sidebar.tsx`

### Adding a New API Service

1. Create function in appropriate `services/*.ts` file
2. Use `authHeader()` for authenticated endpoints
3. Use TypeScript types for payload/response
4. Throw descriptive Error on failure

### Form with Validation

1. Define schema in `utils/zod-schemas.ts`
2. Use `useState` for form data + errors
3. Validate with `schema.safeParse()` on submit
4. Pass `submitted={true}` to inputs after first submit attempt

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/faizadli)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/faizadli)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
