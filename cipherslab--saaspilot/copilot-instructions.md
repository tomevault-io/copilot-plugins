## saaspilot

> Welcome! This guide helps AI coding agents (Claude, Cursor, etc.) quickly understand and work with SaaSPilot.

# Building with Claude - Quick Start Guide

Welcome! This guide helps AI coding agents (Claude, Cursor, etc.) quickly understand and work with SaaSPilot.

## 🚀 Quick Setup

1. **Environment Setup**:
```bash
cp .env.example .env.local
# Fill in required variables
```

2. **Install Dependencies**:
```bash
npm install
```

3. **Database Setup**:
```bash
npx prisma generate
npx prisma db push
```

4. **Run Development Server**:
```bash
npm run dev
```

Visit: `http://localhost:3000`

---

## 📚 Essential Reading for AI Agents

Before making changes, read these files IN ORDER:

1. **[/docs/ai-instructions.md](docs/ai-instructions.md)** - Core instructions and conventions
2. **[/docs/architecture.md](docs/architecture.md)** - System architecture overview
3. **[/docs/coding-patterns.md](docs/coding-patterns.md)** - Standard code patterns
4. **[/docs/api-documentation.md](docs/api-documentation.md)** - API reference

---

## 🎯 Common Tasks

### Adding a New Feature
```bash
# 1. Read the prompt template
# 2. Follow the pattern in /docs/coding-patterns.md
# 3. Create feature structure:
/app/feature-name/
  ├── page.tsx
  ├── components/
  └── layout.tsx

/actions/feature-name/
  └── action.ts
```

See: [/prompts/add-new-feature.md](prompts/add-new-feature.md)

### Creating an API Endpoint
See: [/prompts/create-api-endpoint.md](prompts/create-api-endpoint.md)

### Modifying Database Schema
See: [/prompts/modify-database-schema.md](prompts/modify-database-schema.md)

### Adding a Payment Plan
See: `/prompts/add-payment-plan.md` (coming soon)

### Customizing UI Components
See: `/prompts/customize-ui-component.md` (coming soon)

---

## 🛠️ AI-Friendly Features

✅ **Comprehensive JSDoc comments** on all major functions
✅ **Consistent naming conventions** throughout codebase
✅ **Type-safe** with TypeScript
✅ **Clear folder structure** with purpose-specific directories
✅ **Modular architecture** - easy to extend
✅ **Detailed error handling** patterns
✅ **Ready-to-use examples** (coming soon)

---

## 📁 Key Directories

```
/app              → Pages and routes (Next.js App Router)
/actions          → Server actions (business logic)
/components       → React components
  /ui             → Base components (Shadcn)
  /auth           → Authentication components
/db               → Database client
/lib              → Utility functions and services
/hooks            → React hooks
/types            → TypeScript types
/schemas          → Zod validation schemas
/prisma           → Database schema
/docs             → Documentation (AI instructions)
/prompts          → Task templates for AI agents
/examples         → Complete example projects (coming soon)
```

---

## 🔐 Authentication Flow

```typescript
// Check if user is authenticated
import { auth } from '@/auth'

const session = await auth()
if (!session?.user) {
  // Handle unauthorized
}
```

---

## 💾 Database Queries

```typescript
// Always use Prisma
import { db } from '@/db'

const user = await db.user.findUnique({
  where: { id: userId }
})
```

---

## ⚡ Server Actions

```typescript
'use server'

export async function myAction(formData: FormData) {
  // 1. Auth check
  // 2. Validation
  // 3. Database operation
  // 4. Return result
}
```

---

## 🎨 UI Components

```typescript
// Use Shadcn/ui components
import { Button } from '@/components/ui/button'
import { Card } from '@/components/ui/card'

// Tailwind for styling
<div className="flex items-center gap-4">
  <Button variant="default">Click me</Button>
</div>
```

---

## 🚨 Important Files (Modify with Caution)

- [/auth.ts](auth.ts), [/auth.config.ts](auth.config.ts) - Authentication core
- [/middleware.ts](middleware.ts) - Request middleware
- [/prisma/schema.prisma](prisma/schema.prisma) - Database schema (requires migration)
- `/app/api/webhooks/` - Webhook handlers (security-sensitive)

---

## 🧪 Testing Changes

```bash
# Type checking
npm run type-check

# Linting
npm run lint

# Build test
npm run build
```

---

## 📦 Adding Dependencies

```bash
npm install package-name

# Update after adding packages
npx prisma generate  # If DB-related
```

---

## 🐛 Debugging Tips

1. **Check logs**: Console in development
2. **Database**: Use `npx prisma studio` to view data
3. **API testing**: Use Thunder Client or Postman
4. **Type errors**: Run `npm run type-check`

---

## 💡 When Unsure

1. Check `/docs` for specific guidance
2. Look at `/examples` for similar implementations (coming soon)
3. Review `/prompts` for task templates
4. Ask for clarification before making major changes

---

## 🤖 AI Agent Best Practices

**DO**:
✅ Read relevant docs before coding
✅ Follow established patterns
✅ Add comments for complex logic
✅ Use TypeScript types
✅ Test changes locally
✅ Update documentation when adding features

**DON'T**:
❌ Modify core auth/security files without understanding
❌ Skip validation in server actions
❌ Ignore TypeScript errors
❌ Add dependencies without necessity
❌ Commit `.env` files
❌ Break existing patterns without reason

---

## 🔗 Useful Commands

```bash
# Development
npm run dev                    # Start dev server
npm run build                  # Production build
npm run start                  # Start production server

# Database
npx prisma studio              # Visual database browser
npx prisma generate            # Regenerate Prisma client
npx prisma db push             # Push schema changes
npx prisma migrate dev         # Create migration

# Code Quality
npm run lint                   # Run ESLint
npm run type-check             # TypeScript check
```

---

## 📞 Getting Help

- **Documentation**: [/docs](docs/) directory
- **Examples**: `/examples` directory (coming soon)
- **Templates**: [/prompts](prompts/) directory
- **Architecture**: [/docs/architecture.md](docs/architecture.md)
- **Patterns**: [/docs/coding-patterns.md](docs/coding-patterns.md)

---

## 🎯 Your First Task

Try this:

1. Read [/docs/ai-instructions.md](docs/ai-instructions.md)
2. Look at `/examples` for reference (coming soon)
3. Choose a task from [/prompts](prompts/)
4. Follow the pattern
5. Test your changes

**Good luck coding! 🚀**

---
> Source: [CiphersLab/SaaSPilot](https://github.com/CiphersLab/SaaSPilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
