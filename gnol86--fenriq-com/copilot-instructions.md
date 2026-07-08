## fenriq-com

> This file provides specific guidance for specialized agents working with this codebase.

# AGENTS.md

This file provides specific guidance for specialized agents working with this codebase.

## About the project Boilerplate

Boilerplate for Next.js 16 projects with Prisma, Better-Auth, Resend, Tailwind CSS 4, Shadcn, lucide-react, React-Hook-Form, zod, Next-Themes, and more.

## Development Commands

### Core Commands

- `bun dev` - Start development server with Turbopack
- `bun build` - Build the application
- `bun start` - Start production server
- `bun lint` - Lint the code
- `bun format` - Format the code

## Architecture Overview

### Technology Stack

- **Framework**: Next.js 15.5 with App Router
- **Language**: Javascript
- **Styling**: TailwindCSS v4 with Shadcn/UI components
- **Database**: PostgreSQL with Prisma ORM
- **Authentication**: Better-Auth with organization and admin support
- **Email**: React Email with Resend
- **Payments**: Stripe integration
- **Package Manager**: bun

### Project Structure

- `app/` - Next.js App Router pages and layouts
- `src/components/` - UI components (Shadcn/UI in `src/components/ui/`, custom in `src/components/`)
- `src/features/` - Feature-specific components and logic
- `src/lib/` - Utilities, configurations, and services
- `src/hooks/` - Custom React hooks
- `src/actions/` - Actions server
- `src/components/email/` - Email templates using React Email
- `prisma/` - Database schema and migrations
- `src/messages/` - Internationalization messages

### Key Features

- **Multi-tenant Organizations**: Full organization management with roles and permissions
- **Authentication**: Email/password
- **Billing**: Stripe subscriptions with plan management
- **Dialog System**: Global dialog manager for modals and confirmations
- **Forms**: React Hook Form with Zod validation and server actions
- **Email System**: Transactional emails with React Email

## Multi-Project Architecture

This boilerplate supports creating multiple projects (A, B, C) that can receive updates from the boilerplate upstream without conflicts.

### Project vs Boilerplate Files

**🔒 Protected Project Files** (never overwritten by upstream updates):

- `src/project/**` - ALL your custom code (components, actions, hooks, lib, features, sidebar)
- `src/messages/*.project.json` - Project-specific translations (fr, en, nl, de)
- `prisma/schema/project.prisma` - Project-specific database models
- `src/app/page.js` - Landing page template
- `src/app/globals.css` - Theme/styling customization
- `src/site-config.js` - Project configuration
- `public/images/logo.png` - Your logo
- `public/images/icon.png` - Your icon

**⬆️ Boilerplate Files** (automatically updated from upstream):

- `src/actions/*.action.js` - Boilerplate server actions
- `src/components/**` - Boilerplate UI components
- `src/lib/**` - Boilerplate utilities
- `src/hooks/**` - Boilerplate hooks
- `src/app/(pages)/**` - Dashboard pages
- `src/messages/*.json` - Boilerplate translations (not \*.project.json)
- `prisma/schema/base.prisma` - Boilerplate database models

### Import Paths

Use `@project/*` for project-specific code:

```js
// Boilerplate imports
import { Button } from "@/components/ui/button";
import { useServerAction } from "@/hooks/use-server-action";

// Project imports
import { MyComponent } from "@project/components/my-component";
import { myAction } from "@project/actions/my.action";
import { useMyHook } from "@project/hooks/use-my-hook";
```

### Where to Put Your Code

**✅ ALWAYS put project-specific code in `src/project/`:**

- Components: `src/project/components/my-component.jsx`
- Server actions: `src/project/actions/my.action.js`
- Hooks: `src/project/hooks/use-my-hook.js`
- Utilities: `src/project/lib/my-util.js`
- Features: `src/project/features/my-feature/`
- Sidebar: `src/project/sidebar/app-sidebar.jsx`

**✅ Project translations in `*.project.json`:**

```json
// src/messages/en.project.json
{
    "my_feature": {
        "title": "My Feature",
        "description": "..."
    }
}
```

**Usage:**

```js
const t = useTranslations("project.my_feature");
t("title"); // "My Feature"
```

**✅ Project database models in `prisma/schema/project.prisma`:**

```prisma
model Product {
    id             String       @id @default(cuid())
    name           String
    organizationId String
    organization   Organization @relation(fields: [organizationId], references: [id])

    @@map("product")
}
```

### Updating from Boilerplate

To get the latest boilerplate updates:

```bash
# Fetch and merge upstream changes
git fetch upstream
git merge upstream/main

# Install new dependencies if any
bun install

# Regenerate Prisma if schema changed
bun prisma generate
```

**What happens:**

- ✅ Boilerplate files automatically updated
- ✅ Project files (`src/project/**`) never touched
- ⚠️ `package.json` may need manual merge (keep both dependencies)

### Setup New Project

See `.github/SETUP_NEW_PROJECT.md` for the complete guide.

**Quick start:**

```bash
./scripts/setup-new-project.sh
```

**What the script does:**

1. Clones the boilerplate into a new directory
2. Configures `upstream` (boilerplate) and `origin` (your project) remotes
3. Activates `merge=theirs` strategy (defined in `.gitattributes`) to protect project files
4. Updates `src/site-config.js` with the project name
5. Creates an initial commit and pushes to your remote

**Key files:**

- `.gitattributes` - Defines which files are protected during upstream merges
- `scripts/setup-new-project.sh` - Automated project creation script
- `.github/SETUP_NEW_PROJECT.md` - Full documentation

## 🚨 ABSOLUTE RULES - DO NOT BREAK

### NEVER modify `src/components/ui/` files

The files in `src/components/ui/` are managed by shadcn/ui CLI and updated via the boilerplate upstream. **NEVER edit, modify, or touch any file in `src/components/ui/`**. These are auto-generated wrapper components. If there's a bug or incompatibility in a UI wrapper, inform the user instead of modifying the file.

## Code Conventions

### React/Next.js

- Prefer React Server Components over client components
- Use `"use client"` only for small components
- Wrap client components in `Suspense` with fallback
- Use dynamic loading for non-critical components
- Split components into smaller components

### Styling

- Mobile-first approach with TailwindCSS
- Use Shadcn/UI components from `src/components/ui/`
- Custom components in `src/components/`

### Styling preferences

- For spacing, prefer utility layouts like `flex flex-col gap-4` for vertical spacing and `flex gap-4` for horizontal spacing (instead of `space-y-4`).
- Prefer the card container `@src/components/ui/card.jsx` for styled wrappers rather than adding custom styles directly to `<div>` elements.

### State Management

- Use `nuqs` for URL search parameter state

### Forms and Server Actions

- Use React Hook Form with Zod validation
- Server actions in `.action.js` files
- **ALWAYS** use `useServerAction` hook for executing server actions in client components
- Use `resolveActionResult` helper for mutations
- Follow form creation pattern in `/src/features/form/`

### Dialogs and Confirmations

- **ALWAYS** use `useConfirm` hook for confirmation dialogs
- The `useConfirm` hook provides a centralized dialog system via `DialogProvider`
- Pattern: Import hook, initialize it, then call `confirm()` with config and callback

**✅ REQUIRED Pattern:**

```js
import { useConfirm } from "@/hooks/use-confirm";

const confirm = useConfirm();
const { execute } = useServerAction();

// Usage with server action
await confirm(
    {
        title: t("dialog_title"),
        description: t("dialog_description"),
        variant: "destructive", // or "default"
    },
    () =>
        execute(() => myServerAction(data), {
            successMessage: t("success"),
            errorMessage: t("error"),
        })
);
```

**Key Points:**

- `confirm()` takes 2 arguments: config object and callback function
- Config object: `{ title, description, variant }` where variant can be `"default"` or `"destructive"`
- The callback is executed only if user confirms
- Returns a Promise that resolves when user confirms/cancels
- Must be used within `DialogProvider` context

### Authentication & Access Control

#### **🎯 CRITICAL - Centralized Access Control System**

All authentication and permission checks MUST use the helpers from `src/lib/access-control.js`. This ensures consistency, security, and maintainability.

**Available Helpers:**

1. **`requireAuth()`** - Vérifie l'authentification
   - Returns: `{ session, user }`
   - Throws: `notFound()` if not authenticated
   - Use: Pages requiring any authenticated user

2. **`requireActiveOrganization()`** - Vérifie l'organisation active
   - Returns: `{ session, user, organization }`
   - Throws: `notFound()` if no active organization
   - Use: Pages requiring organization context

3. **`requirePermission({ permissions })`** - Vérifie les permissions
   - Returns: `{ session, user, organization }`
   - Throws: `notFound()` if insufficient permissions
   - Use: Pages with specific organization permissions

4. **`requireAdmin()`** - Vérifie le rôle admin
   - Returns: `{ session, user }`
   - Throws: `notFound()` if not admin
   - Use: Admin-only pages

5. **`checkPermission({ permissions })`** - Vérifie sans erreur
   - Returns: `boolean`
   - Use: Conditional UI rendering

6. **`checkAdmin()`** - Vérifie admin sans erreur
   - Returns: `boolean`
   - Use: Conditional UI for admin features

#### **Usage Patterns**

**Server Components - Pages with Organization Permissions:**

```js
import { requirePermission, checkPermission } from "@/lib/access-control";

export default async function MembersPage() {
    // Vérifie les permissions ET récupère les données
    const { user, organization } = await requirePermission({
        permissions: { member: ["read"] }
    });

    // Pour l'UI conditionnelle
    const canDelete = await checkPermission({
        permissions: { member: ["delete"] }
    });

    return <div>...</div>;
}
```

**Server Components - Admin Pages:**

```js
import { requireAdmin } from "@/lib/access-control";

export default async function AdminPage() {
    // Vérifie le rôle admin
    const { user } = await requireAdmin();

    return <div>...</div>;
}
```

**Server Components - User Pages:**

```js
import { requireAuth } from "@/lib/access-control";

export default async function UserSettingsPage() {
    // Vérifie l'authentification
    const { user } = await requireAuth();

    return <div>...</div>;
}
```

**Server Components - Sidebar/Navigation:**

```js
import { checkPermission, checkAdmin } from "@/lib/access-control";

export default async function Sidebar() {
    // Vérifications pour affichage conditionnel
    const canManageOrg = await checkPermission({
        permissions: { organization: ["update"] }
    });
    const isAdmin = await checkAdmin();

    return (
        <>
            {canManageOrg && <Link href="/org/manage">Manage</Link>}
            {isAdmin && <Link href="/admin">Admin</Link>}
        </>
    );
}
```

#### **Security Note**

- **Why `notFound()` instead of `unauthorized()`?**
  - Better security: doesn't reveal page existence to unauthorized users
  - Principle: "security through obscurity" + real security
  - Unauthorized users see 404, not 401

#### **Rules**

2. **NEVER** manually check `user.role === "admin"` - use `checkAdmin()`
3. **ALWAYS** use `require*` functions for page-level checks
4. **ALWAYS** use `check*` functions for conditional UI
5. **ALWAYS** consider cache invalidation after permission changes

### Database

- Prisma ORM with PostgreSQL
- Database hooks for user creation setup
- Organization-based data access patterns

## Important Files

### **🚨 CRITICAL - Access Control System**

- `src/lib/access-control.js` - **CENTRALIZED** access control - ALWAYS use these helpers for auth/permissions
  - `requireAuth()` - Pages requiring authentication
  - `requirePermission()` - Pages requiring organization permissions
  - `requireAdmin()` - Admin-only pages
  - `checkPermission()` - Conditional UI based on permissions
  - `checkAdmin()` - Conditional UI for admin features

### **🚨 CRITICAL - Server Actions System**

- `src/hooks/use-server-action.js` - **PRIMARY** hook for server actions - ALWAYS use this in client components

### **Dialog System**

- `src/hooks/use-confirm.js` - Hook for confirmation dialogs - ALWAYS use this for confirmations
- `src/components/providers/dialog-provider.jsx` - Global dialog provider

### **Other Core Files**

- `src/components/ui/form.jsx` - Form components
- `prisma/schema.prisma` - Database schema
- `src/lib/auth.js` - Better-Auth configuration (DO NOT import directly for auth checks)

## Development Notes

- Always use `bun` for package management
- Use TypeScript strict mode - no `any` types
- Prefer server components and avoid unnecessary client-side state
- Prefer using `??` than `||`
- After every file creation or modification, run `bun lint` before finishing the task

## Files naming

- All server actions should be suffix by `.action.js` eg. `user.action.js`, `dashboard.action.js`

## Debugging and complexe tasks

- For complexe logic and debugging, use logs. Add a lot of logs at each steps and ASK ME TO SEND YOU the logs so you can debugs easily.

## TypeScript imports

Important, when you import thing try to always use TypeScript paths :

- `@/*` is link to @src
- `@app/*` is link to @src/app
- `@components/*` is link to @src/components
- `@lib/*` is link to @src/lib
- `@hooks/*` is link to @src/hooks
- `@actions/*` is link to @src/actions
- `@email/*` is link to @src/components/email

## Workflow modification

🚨 **CRITICAL RULE - ALWAYS FOLLOW THIS** 🚨

**BEFORE editing any files, you MUST Read at least 3 files** that will help you to understand how to make a coherent and consistency.

This is **NON-NEGOTIABLE**. Do not skip this step under any circumstances. Reading existing files ensures:

- Code consistency with project patterns
- Proper understanding of conventions
- Following established architecture
- Avoiding breaking changes

**Types of files you MUST read:**

1. **Similar files**: Read files that do similar functionality to understand patterns and conventions
2. **Imported dependencies**: Read the definition/implementation of any imports you're not 100% sure how to use correctly - understand their API, types, and usage patterns

**Steps to follow:**

1. Read at least 3 relevant existing files (similar functionality + imported dependencies)
2. Understand the patterns, conventions, and API usage
3. Only then proceed with creating/editing files

### **🚨 FORBIDDEN Patterns**

```js
// ❌ NEVER use these legacy patterns
import { getUser, needUser } from "@/lib/auth"; // FORBIDDEN
const user = await getUser(); // FORBIDDEN
const user = await needUser(); // FORBIDDEN

// ❌ NEVER manually check user role
if (user.role !== "admin") notFound(); // FORBIDDEN - use requireAdmin instead
{user.role === "admin" && <AdminLink />} // FORBIDDEN - use checkAdmin instead

// ❌ NEVER manually get session and check auth
const session = await auth.api.getSession({ headers: await headers() });
if (!session?.user) notFound(); // FORBIDDEN - use requireAuth instead
```

### **✅ REQUIRED Patterns**

```js
// ✅ ALWAYS use centralized access control helpers
import { requireAuth, requirePermission, requireAdmin } from "@/lib/access-control";
import { checkPermission, checkAdmin } from "@/lib/access-control";

// ✅ For pages requiring authentication
const { user } = await requireAuth();

// ✅ For pages requiring organization permissions
const { user, organization } = await requirePermission({
    permissions: { member: ["read"] }
});

// ✅ For admin pages
const { user } = await requireAdmin();

// ✅ For conditional UI
const canDelete = await checkPermission({ permissions: { member: ["delete"] } });
const isAdmin = await checkAdmin();

// ✅ ALWAYS use useServerAction hook for server actions in client components
import { useServerAction } from "@/hooks/use-server-action";
const { execute, isPending, isSuccess, isError } = useServerAction();
await execute(() => myServerAction(data), {
    loadingMessage: "En cours...",
    successMessage: "Succès!",
    errorMessage: "Erreur",
});

// ✅ ALWAYS use useConfirm hook for confirmation dialogs
import { useConfirm } from "@/hooks/use-confirm";
const confirm = useConfirm();
await confirm(
    {
        title: t("dialog_title"),
        description: t("dialog_description"),
        variant: "destructive",
    },
    () =>
        execute(() => myServerAction(data), {
            successMessage: "...",
            errorMessage: "...",
        })
);
```

# **Internationalization (i18n)**

This project uses **next-intl** for internationalization with French (fr) as the default language and English (en) support.

## Configuration Files

- `@lib/i18n/config.js` - Locale configuration (locales array, defaultLocale)
- `@lib/i18n/request.js` - Request configuration for loading translations based on cookie
- `@/messages/fr-FR.json` - French translations
- `@/messages/en-US.json` - English translations
- `@/messages/nl-NL.json` - Dutch translations
- `@/messages/de-DE.json` - German translations
- Cookie: `NEXT_LOCALE` stores user's locale preference

## Translation Patterns

### Server Components

```js
import { getTranslations } from "next-intl/server";

export default async function MyPage() {
    const t = await getTranslations("namespace.subnamespace");

    return <h1>{t("page_title")}</h1>;
}
```

### Client Components

```js
"use client";
import { useTranslations } from "next-intl";

export default function MyComponent() {
    const t = useTranslations("namespace.subnamespace");

    return <h1>{t("page_title")}</h1>;
}
```

### Zod Schemas in Client Components

Schemas must be defined **inside** the component to access translation hooks:

```js
"use client";
import { useTranslations } from "next-intl";

export default function MyForm() {
    const t = useTranslations("validation");

    // ✅ Schema inside component to access t()
    const formSchema = z.object({
        name: z
            .string()
            .min(2, t("name.min_length"))
            .max(50, t("name.max_length")),
    });

    // ... rest of component
}
```

## Translation Organization

Translations are organized by namespaces in `messages/xx.json` :

```json
{
    "auth": {
        "signin": { "page_title": "...", "email_label": "..." },
        "signup": { "page_title": "...", "password_label": "..." }
    },
    "user": {
        "settings": { "page_title": "...", "name_label": "..." }
    },
    "organization": {
        "members": { "page_title": "...", "table_user": "..." }
    },
    "validation": {
        "email": { "invalid": "...", "required": "..." },
        "password": { "min_length": "...", "mismatch": "..." }
    },
    "common": { "cancel": "...", "save": "...", "n_a": "..." },
    "roles": { "owner": "...", "admin": "...", "member": "..." },
    "breadcrumbs": { "dashboard": "...", "settings": "..." }
}
```

## Helper Patterns for Dynamic Content

Translate dynamic labels with `next-intl` directly.

```js
// Client component example
const tRoles = useTranslations("roles");
const roleLabel = tRoles(member.role);

// Server component example
const tInvitationStatus = await getTranslations("invitation_status");
const statusLabel = tInvitationStatus(statusKey);
```

Utilities like `getInvitationDisplayStatus` return status keys (`pending`, `outdated`, etc.). Convert those keys to user-facing text with the appropriate translation namespace inside the component.

## Translation Keys Best Practices

1. **Namespace by feature**: `auth.signin`, `user.settings`, `organization.members`
2. **Descriptive keys**: `page_title`, `email_label`, `save_button`, `success_message`
3. **Validation namespace**: Group all validation messages under `validation.*`
4. **Common namespace**: Shared translations like buttons, labels, N/A
5. **Use parameters**: `t("confirm_delete", { name: org.name })`

## Adding Translations

When adding new user-facing text:

1. **Never hardcode text** - Always use translation keys
2. **🚨 CRITICAL: Add to ALL language files** - `messages/xx-XX.json`
3. **ALWAYS verify all language files exist** - Use `ls src/messages/` or `Glob` to list all .json files before adding translations
4. **Use appropriate namespace** based on the feature
5. **Keep keys consistent** across all languages
6. **Test all languages** to ensure translations work

### Supported Languages

- 🇫🇷 French
- 🇬🇧 English
- 🇳🇱 Dutch
- 🇩🇪 German

## Example: Complete Page Translation

```js
// Server Component
import { getTranslations } from "next-intl/server";

export default async function MembersPage() {
    const t = await getTranslations("organization.members");

    return (
        <Card>
            <CardHeader>
                <CardTitle>{t("page_title")}</CardTitle>
                <CardDescription>{t("page_description")}</CardDescription>
            </CardHeader>
            <CardContent>
                <Table>
                    <TableHeader>
                        <TableRow>
                            <TableHead>{t("table_user")}</TableHead>
                            <TableHead>{t("table_role")}</TableHead>
                        </TableRow>
                    </TableHeader>
                </Table>
            </CardContent>
        </Card>
    );
}
```

# **Documentation:**

✅ ALWAYS use web-search to find documentation about the library you're using.

---
> Source: [Gnol86/fenriq.com](https://github.com/Gnol86/fenriq.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
