## e-commerce

> - Framework: Next.js 16 (App Router), React 19, TypeScript 5


# The Main Rule No 57

# ============================================================
# PROJECT CONTEXT
# ============================================================

<project_context>
- Framework: Next.js 16 (App Router), React 19, TypeScript 5
- Styling: Tailwind CSS 4 with shadcn/ui pattern (class-variance-authority, tailwind-merge)
- Database: Drizzle ORM with PostgreSQL (Neon Database)
- API: tRPC with React Query (@tanstack/react-query)
- Auth: Better Auth with email/password and OAuth
- Forms: React Hook Form + Zod validation (@hookform/resolvers)
- UI Components: Radix UI primitives + custom components
- Icons: Lucide React
- State: React Query for server state, React hooks for local state
- File Upload: Vercel Blob
- Email: React Email + Resend
</project_context>

# ============================================================
# RULE 1: FILE STRUCTURE & ORGANIZATION
# ============================================================

<file_structure>
- App Router: src/app/(group)/page.tsx pattern with route groups
- API Routes: src/app/api/[...trpc]/route.ts for tRPC
- Modules: src/module/[feature]/[feature].[type].ts pattern
- Core: src/core/[api|auth|db|query]/ for shared infrastructure
- Shared: src/shared/[components|config|schema|utils]/ for cross-cutting concerns
- Components: src/shared/components/ui/ for primitive components
- Forms: src/shared/components/form/ with field configuration pattern
- Utils: src/shared/utils/lib/ for utility functions
</file_structure>

# ============================================================
# RULE 2: MODULE NAMING CONVENTIONS
# ============================================================

<module_naming>
- Schema: [feature].schema.ts (e.g., product.schema.ts)
- API: [feature].api.ts (e.g., product.api.ts)
- Database Schema: db.schema.ts in core/db/
- Types: Inline in schema files or types.ts in shared/types/
- Forms: [feature]-form.tsx or [feature].form.tsx
- Actions: [feature].actions.ts for server actions
</module_naming>

# ============================================================
# RULE 3: ZOD SCHEMA PATTERNS
# ============================================================

<zod_schemas>
- Use zod/v3 import: import z from 'zod/v3'
- Base schema: export const base[Feature]Schema with all fields
- Select schema: export const [feature]SelectSchema = base[Feature]Schema
- Insert schema: export const [feature]InsertSchema = baseSchema.omit({ id, createdAt, updatedAt, deletedAt })
- Update schema: export const [feature]UpdateSchema = baseSchema.partial()
- Use .min(1) for required strings, not .nonempty()
- Use .nullable().optional() for optional nullable fields
- Use .default(() => new Date()) for timestamps
- Use .enum() for status fields with specific values
</zod_schemas>

# ============================================================
# RULE 4: API CONTRACT PATTERN (tRPC)
# ============================================================

<api_contracts>
- Define contract object in schema file: export const [feature]Contract
- Each endpoint has: input (z.object with params/query/body), output (detailedResponse)
- Input structure: { params: {...} } for route params, { query: {...} } for search params, { body: {...} } for POST data
- Output always uses detailedResponse(wrapperSchema) helper
- Use paginationSchema.extend({...}) for list endpoints
- Use pick/omit/extend for related data in responses
</api_contracts>

# ============================================================
# RULE 5: tRPC ROUTER PATTERNS
# ============================================================

<trpc_routers>
- Import: import { createTRPCRouter, protectedProcedure, publicProcedure } from '@/core/api/api.methods'
- Router name: export const [feature]Router = createTRPCRouter({...})
- Use publicProcedure for unauthenticated endpoints
- Use protectedProcedure for authenticated endpoints only
- Pattern: .input(contract.endpoint.input).output(contract.endpoint.output).query/mutation(async ({ input, ctx }) => {...})
- Always wrap in try/catch with debugError logging
- Return API_RESPONSE(STATUS, MESSAGE, data, error?) format
</trpc_routers>

# ============================================================
# RULE 6: API RESPONSE FORMAT
# ============================================================

<api_response>
- Use API_RESPONSE helper from @/shared/config/api.utils
- Status: STATUS.SUCCESS | STATUS.ERROR | STATUS.FAILED
- Message: Import from MESSAGE constant object (e.g., MESSAGE.PRODUCT.GET.SUCCESS)
- Data: Nullable, typed from schema
- Error: Optional Error object for error cases
- Always return consistent shape even on failure
</api_response>

# ============================================================
# RULE 7: DRIZZLE ORM PATTERNS
# ============================================================

<drizzle_patterns>
- Import db from @/core/db/db
- Import tables from @/core/db/db.schema
- Use db.query.[table].findMany/findFirst for relational queries with "with" clause
- Use db.select().from().where() for simple queries
- Use and(), or(), eq(), ilike(), isNull() from drizzle-orm for conditions
- Soft delete: Check isNull(table.deletedAt) in all queries
- Use .returning() for mutations to get affected rows
- Order by: orderBy: (t, { desc }) => [desc(t.createdAt)]
</drizzle_patterns>

# ============================================================
# RULE 8: DATABASE SCHEMA DEFINITION
# ============================================================

<db_schema>
- Use timestamp('created_at', { withTimezone: true }).defaultNow().notNull()
- Use timestamp('updated_at', { withTimezone: true }).$onUpdate(() => new Date())
- Use timestamp('deleted_at', { withTimezone: true }) for soft deletes
- Use varchar('id', { length: 36 }).primaryKey().$defaultFn(() => uuidv4())
- Use text() for long strings, varchar(length) for short
- Use jsonb() for JSON data with .$type<SpecificType>()
- Use boolean().default(true) for flags
- Use integer() or numeric() for numbers
- Define relations with relations() helper for relational queries
</db_schema>

# ============================================================
# RULE 9: FORM CREATION PATTERNS
# ============================================================

<form_patterns>
- Use Form component from @/shared/components/form/form
- Schema: Pass Zod schema to Form component via schema prop
- Default values: Always provide defaultValues matching schema structure
- Submit handler: Use onSubmitAction prop with async function
- Fields: Use Form.FieldsWrapper with fieldsConfig array
- Field config: { name, type, label, description?, placeholder?, required? }
- Available field types: text, textarea, number, select, checkbox, switch, radio, password, otp, slug, color, currency, image-upload, multi-select
- Mode: 'onChange' for real-time validation
</form_patterns>

# ============================================================
# RULE 10: FORM FIELD CONFIGURATION
# ============================================================

<form_fields>
- Basic: { name: 'title', type: 'text', label: 'Title', required: true }
- Select: { name: 'status', type: 'select', label: 'Status', options: [{ label: 'Active', value: 'active' }] }
- Relations: Use custom field types like 'series', 'subcategory' for relation pickers
- Images: Use type: 'image-upload' with maxFiles, accept props
- Validation: Add description for validation hints (e.g., "At least 2 characters")
- Nested: Use Form.FormGroup for array fields with add/remove functionality
</form_fields>

# ============================================================
# RULE 11: REACT HOOK FORM INTEGRATION
# ============================================================

<react_hook_form>
- Use zodResolver from @hookform/resolvers/zod
- Mode: 'onChange' for immediate feedback
- ReValidateMode: 'onChange' for consistency
- CriteriaMode: 'all' to show all errors
- shouldFocusError: true for UX
- progressive: true for progressive enhancement
- Access form state via Form.FormContext or useFormContext()
- Watch values via Form.FormWatch for conditional rendering
</react_hook_form>

# ============================================================
# RULE 12: TYPE DEFINITION PATTERNS
# ============================================================

<type_definitions>
- Infer from Zod: type [Feature] = z.infer<typeof base[Feature]Schema>
- Use type over interface for object shapes
- Export types from schema files: export type [Feature]Select = z.infer<typeof [feature]SelectSchema>
- For DB types: Use typeof table.$inferSelect and typeof table.$inferInsert
- Props types: type [Component]Props = { ... } in component file
- Reuse base types with Pick, Omit, Partial where appropriate
</type_definitions>

# ============================================================
# RULE 13: VALIDATION RULES
# ============================================================

<validation>
- Required strings: z.string().min(1, 'Field is required')
- Optional strings: z.string().nullable().optional()
- Numbers: z.number().min(0) for positive, z.number().int() for integers
- Enums: z.enum(['value1', 'value2']).default('value1')
- Arrays: z.array(itemSchema).min(1, 'At least one item required')
- Dates: z.date().default(() => new Date())
- UUIDs: z.string().uuid() for ID validation
- Custom: Use .refine() or .transform() for complex validation
- Coerce: z.coerce.number() for form string-to-number conversion
</validation>

# ============================================================
# RULE 14: SERVER ACTIONS PATTERNS
# ============================================================

<server_actions>
- Use 'use server' directive at top of file
- Import auth session via Better Auth
- Validate input with Zod before processing
- Revalidate paths with revalidatePath() from next/cache
- Return typed responses matching API response format
- Handle errors with try/catch and return error objects
- Use redirect() from next/navigation after mutations
</server_actions>

# ============================================================
# RULE 15: COMPONENT PATTERNS
# ============================================================

<component_patterns>
- Server components by default (no 'use client')
- 'use client' only when using: useState, useEffect, useContext, browser APIs
- Props interface: type [Component]Props = { ... }
- Export: export function Component(props: ComponentProps) { ... }
- Composition: Use children prop for flexible composition
- Styling: Use cn() utility from @/shared/utils/lib/utils for conditional classes
- Forward refs with React.forwardRef for interactive components
</component_patterns>

# ============================================================
# RULE 16: TAILWIND CSS PATTERNS
# ============================================================

<tailwind_patterns>
- Use v4 syntax with @import "tailwindcss" in CSS
- Use theme() function for theme values: theme(colors.primary)
- Spacing: Use standard scale (1, 2, 4, 8, 12, 16, etc.)
- Colors: Use semantic names (primary, secondary, destructive, muted, accent)
- Typography: Use text-xs/sm/base/lg/xl/2xl etc.
- Layout: Use flex, grid utilities with gap utilities
- Responsive: Use sm:, md:, lg:, xl: prefixes
- State: Use hover:, focus:, active:, disabled:, data-[state=open]:
- Animation: Use animate-in, animate-out with fade-in, zoom-in, slide-in
</tailwind_patterns>

# ============================================================
# RULE 17: UI COMPONENT PATTERNS
# ============================================================

<ui_components>
- Use class-variance-authority (cva) for component variants
- Pattern: const componentVariants = cva('base classes', { variants: { variant: { ... }, size: { ... } } })
- Forward refs with React.forwardRef
- Export type ComponentRef = React.ElementRef<typeof Component>
- Use Radix UI primitives as base (Dialog, Dropdown, etc.)
- Icons: Use Lucide React icons only
- Slots: Use @radix-ui/react-slot for polymorphic components
</ui_components>

# ============================================================
# RULE 18: AUTHENTICATION PATTERNS
# ============================================================

<authentication>
- Use Better Auth for all auth operations
- Import auth client from @/core/auth/auth-client
- Server-side: Import { auth } from @/core/auth/auth
- Session: Use auth.api.getSession() in server components
- Protected routes: Use protectedProcedure in tRPC
- Client auth: Use useSession() hook from Better Auth
- Two-factor: Use authClient.twoFactor.verify() for 2FA
</authentication>

# ============================================================
# RULE 19: ERROR HANDLING
# ============================================================

<error_handling>
- Use debugError from @/shared/utils/lib/logger.utils for errors
- Format: debugError('CONTEXT:OPERATION:ERROR', error)
- Return API_RESPONSE(STATUS.ERROR, 'Message', null, error) for caught errors
- Client errors: Use toast from sonner for user feedback
- Never expose internal error details to client in production
- Use error.tsx and not-found.tsx for route-level errors
</error_handling>

# ============================================================
# RULE 20: DATA FETCHING PATTERNS
# ============================================================

<data_fetching>
- Server components: Use direct DB queries or tRPC calls
- Client components: Use @tanstack/react-query with tRPC client
- Query keys: Use structured keys ['feature', 'action', params]
- Prefetch: Use queryClient.prefetchQuery in server components
- Infinite scroll: Use useInfiniteQuery with cursor pagination
- Mutations: Use useMutation with onSuccess/onError callbacks
- Revalidation: Use invalidateQueries after mutations
</data_fetching>

# ============================================================
# RULE 21: ROUTE HANDLER PATTERNS
# ============================================================

<route_handlers>
- tRPC handler: src/app/api/trpc/[trpc]/route.ts with fetchRequestHandler
- Auth handler: src/app/api/auth/[...all]/route.ts with auth.handler
- Blob handler: src/app/api/blob/route.ts for file uploads
- Use Route Handler pattern from Next.js 13+
- Return Response.json() with proper status codes
- Validate with Zod before processing
</route_handlers>

# ============================================================
# RULE 22: FILE UPLOAD PATTERNS
# ============================================================

<file_uploads>
- Use Vercel Blob for image/file storage
- Client upload: Use put() from @vercel/blob
- Generate unique filenames: uuid + original extension
- Validate file types: accept only image/*, specific extensions
- Validate file sizes: max 4MB for images, 10MB for documents
- Use presigned URLs for secure uploads
- Store URLs in database, not file contents
</file_uploads>

# ============================================================
# RULE 23: MESSAGE & CONFIGURATION
# ============================================================

<messaging>
- Import STATUS from @/shared/config/api.config (SUCCESS, ERROR, FAILED)
- Import MESSAGE from same location with nested structure
- Pattern: MESSAGE.FEATURE.ACTION.SUCCESS|FAILED|ERROR
- Keep messages user-friendly but consistent
- Use constant objects for API endpoints, table names, etc.
</messaging>

# ============================================================
# RULE 24: SOFT DELETE IMPLEMENTATION
# ============================================================

<soft_deletes>
- Add deletedAt timestamp column to all tables
- Query pattern: and(eq(id, id), isNull(deletedAt))
- Delete operation: Update deletedAt to new Date(), not actual DELETE
- Archive slug: Append timestamp to slug on delete to allow reuse
- Disable on delete: Set isActive: false when soft deleting
- Include deleted records: Remove isNull(deletedAt) condition explicitly
</soft_deletes>

# ============================================================
# RULE 25: SEARCH & PAGINATION
# ============================================================

<search_pagination>
- Pagination: Use limit (default 20, max 100) and offset
- Search: Use ilike(column, `%${term}%) for case-insensitive
- Sorting: Allow { column: string, order: 'asc' | 'desc' }
- Cursor pagination: For large datasets, use cursor-based with id
- Default ordering: createdAt DESC for lists
- Meta: Return count, hasMore in response meta
</search_pagination>

# ============================================================
# RULE 26: SKILLS USAGE GUIDELINES
# ============================================================

<skills_usage>
- Available skills: better-auth-best-practices, atelier-typescript-drizzle-orm, form-react
- Invoke skills: Use @skill-name mention in prompt for specific guidance
- Auto-invoke: Skills trigger automatically on relevant topics
- Custom skills: Create in .windsurf/skills/ with SKILL.md format
- Frontmatter: --- description: title --- for skill metadata
- Use skills for complex patterns: auth setup, DB optimization, form validation
</skills_usage>

# ============================================================
# RULE 27: TESTING PATTERNS
# ============================================================

<testing>
- Unit tests: Co-locate with __tests__/ folder or *.test.ts files
- Component tests: Use Playwright for E2E, React Testing Library for unit
- API tests: Test tRPC routers with mock context
- Mock data: Use factories with faker.js for test data
- E2E: Test critical user flows (auth, purchase, admin)
- Prefer testing user interactions over implementation details
</testing>

# ============================================================
# RULE 28: IMPORT ORGANIZATION
# ============================================================

<import_organization>
- Order: React/Next > External libs > Internal aliases > Relative imports
- Aliases: Use @/core/*, @/module/*, @/shared/* paths
- Group: Separate groups with blank line
- Sort: Alphabetical within groups
- Types: Use import type { ... } for type-only imports
- Barrel exports: Use index.ts for clean module exports
</import_organization>

# ============================================================
# RULE 29: NAMING CONVENTIONS
# ============================================================

<naming_conventions>
- Files: PascalCase for components (ProductCard.tsx), camelCase for utils (formatDate.ts)
- Variables: camelCase for values, PascalCase for types/components
- Constants: UPPER_SNAKE_CASE for true constants
- Database: snake_case for table names and columns
- API: [action][Entity] pattern (getProduct, createOrder)
- Events: handle[Event] pattern (handleSubmit, handleClick)
- Hooks: use[Feature] pattern (useProduct, useAuth)
- Boolean props: is[State] pattern (isLoading, isActive)
</naming_conventions>

# ============================================================
# RULE 30: COMMENTING & DOCUMENTATION
# ============================================================

<documentation>
- JSDoc: Add /** */ comments for exported functions and complex types
- Sections: Use // ==================================================== for file sections
- FIXME/TODO: Use // TODO: or // FIXME: for actionable items
- Why not what: Comments explain reasoning, not restate code
- README: Keep module-level README.md for complex features
- Examples: Include usage examples in JSDoc @example tags
- No unnecessary comments: Code should be self-documenting where possible
</documentation>

# ============================================================
# SKILL TRIGGERS
# ============================================================

<skill_triggers>
- When user mentions "auth", "login", "session", "better-auth": Use @better-auth-best-practices
- When user mentions "drizzle", "database", "schema", "migration": Use @atelier-typescript-drizzle-orm
- When user mentions "form", "validation", "react-hook-form", "zod": Use @form-react
- When user mentions "tailwind", "design", "ui", "component": Use @tailwind-design-system or @frontend-design
- When user mentions "nextjs", "next", "app router", "rsc": Use @next-best-practices or @vercel-react-best-practices
- When user mentions "debug", "fix", "bug", "error": Use @systematic-debugging
- When user mentions "test", "testing", "spec": Use @test-driven-development
- When user mentions "postgres", "sql", "query optimization": Use @supabase-postgres-best-practices
</skill_triggers>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jayantrohila57) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
