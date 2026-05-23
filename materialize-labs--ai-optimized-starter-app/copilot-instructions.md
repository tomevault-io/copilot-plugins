## backend

> Follow these rules when working on the backend.

# Backend Architecture Guidelines

This document outlines our backend architecture and coding standards. Our backend stack includes:

- **Database**: PostgreSQL via Supabase
- **ORM**: Drizzle for type-safe database access
- **API Layer**: Next.js Server Actions
- **Infrastructure**: Serverless via Vercel

## Core Principles

1. **Type Safety**: Maintain full type safety between database and application code
2. **Scalability**: Design for scale from the beginning
3. **Maintainability**: Follow consistent patterns across the codebase
4. **Security**: Validate all inputs and handle user permissions properly
5. **Performance**: Optimize database queries and API responses

## Database Schema Design

### Directory Structure

```
db/
├── db.ts                # Main database configuration
├── index.ts             # Public exports
├── migrations/          # Database migrations
└── schema/              # Database schema definitions
    ├── [entity].ts  # Individual entity schemas
```

### Schema Definition Standards

#### Naming Conventions

- Use kebab-case for files: `contacts.ts`
- Use camelCase for table and column names in code
- Use snake_case for the actual database column names
- Export types with standardized prefixes: `Select[Entity]` and `Insert[Entity]`

#### Required Fields

All tables should include these standard fields:

```typescript
{
  // Always use UUID for primary keys
  id: uuid("id").defaultRandom().primaryKey(),
  
  // User association (when applicable)
  userId: text("user_id").notNull(),
  
  // Timestamps
  createdAt: timestamp("created_at", { withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp("updated_at", { withTimezone: true })
    .defaultNow()
    .notNull()
    .$onUpdate(() => new Date())
}
```

#### Using Enums

For fields with a fixed set of values, use PostgreSQL enums:

```typescript
import { pgEnum } from "drizzle-orm/pg-core"

// Define the enum
export const statusEnum = pgEnum("status", ["active", "pending", "archived"])

// Use it in your schema
status: statusEnum("status").notNull().default("pending")
```

#### Relationships

Always define explicit relationships between tables and include appropriate cascade behavior:

```typescript
// One-to-many relationship example
projectId: uuid("project_id")
  .references(() => projectsTable.id, { onDelete: "cascade" })
  .notNull()
```

### Schema Example

```typescript
// db/schema/contacts.ts
import { pgTable, text, timestamp, uuid } from "drizzle-orm/pg-core"

export const contactsTable = pgTable("contacts", {
  id: uuid("id").defaultRandom().primaryKey(),
  userId: text("user_id").notNull(),
  name: text("name").notNull(),
  email: text("email"),
  phone: text("phone"),
  notes: text("notes"),
  createdAt: timestamp("created_at", { withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp("updated_at", { withTimezone: true })
    .defaultNow()
    .notNull()
    .$onUpdate(() => new Date())
})

export type InsertContact = typeof contactsTable.$inferInsert
export type SelectContact = typeof contactsTable.$inferSelect
```

Make sure to export your schema from the index file:

```typescript
// db/schema/index.ts
export * from "./contacts"
// ... other schema exports
```

And register it in your database configuration:

```typescript
// db/db.ts
import { contactsTable } from "@/db/schema"

export const schema = {
  contacts: contactsTable,
  // ... other tables
}
```

## Server Actions

Server Actions are our primary method for exposing backend functionality. They provide type-safe, secure APIs for frontend components.

### Organization Pattern

```
actions/
├── auth/                # Authentication-related actions
├── db/                  # Database operations
│   ├── [entity].ts
└── utils/               # Utility actions
```

### Action Implementation Guidelines

1. **Standardized Return Type**: Use the `ActionState<T>` pattern for consistency

```typescript
export type ActionState<T> =
  | { isSuccess: true; message: string; data: T }
  | { isSuccess: false; message: string; data?: never }
```

2. **Naming Convention**: All action functions should end with `Action` suffix

3. **Input Validation**: Validate inputs before database operations

4. **Error Handling**: Use try/catch blocks and provide meaningful error messages

5. **Organization**: Group actions by entity and order methods by CRUD operations

### Server Action Example

```typescript
// actions/db/contacts.ts
'use server'

import { eq } from 'drizzle-orm'

import { db } from '@/db/db'
import { contactsTable, InsertContact, SelectContact } from '@/db/schema/contacts'
import { ActionState } from '@/types/server-action'

export async function createContact(contact: InsertContact): Promise<ActionState<SelectContact>> {
  try {
    const [newContact] = await db.insert(contactsTable).values(contact).returning()

    return {
      isSuccess: true,
      message: 'Contact created successfully',
      data: newContact,
    }
  } catch (error) {
    console.error('Error creating contact:', error)

    return { isSuccess: false, message: 'Failed to create contact' }
  }
}

export async function getContacts(userId: string): Promise<ActionState<SelectContact[]>> {
  try {
    const contacts = await db.query.contacts.findMany({
      where: eq(contactsTable.userId, userId),
    })

    return {
      isSuccess: true,
      message: 'Contacts retrieved successfully',
      data: contacts,
    }
  } catch (error) {
    console.error('Error getting contacts:', error)

    return { isSuccess: false, message: 'Failed to get contacts' }
  }
}

export async function updateContact(
  id: string,
  data: Partial<InsertContact>,
): Promise<ActionState<SelectContact>> {
  try {
    const [updatedContact] = await db
      .update(contactsTable)
      .set(data)
      .where(eq(contactsTable.id, id))
      .returning()

    return {
      isSuccess: true,
      message: 'Contact updated successfully',
      data: updatedContact,
    }
  } catch (error) {
    console.error('Error updating contact:', error)

    return { isSuccess: false, message: 'Failed to update contact' }
  }
}

export async function deleteContact(id: string): Promise<ActionState<void>> {
  try {
    await db.delete(contactsTable).where(eq(contactsTable.id, id))

    return {
      isSuccess: true,
      message: 'Contact deleted successfully',
      data: undefined,
    }
  } catch (error) {
    console.error('Error deleting contact:', error)

    return { isSuccess: false, message: 'Failed to delete contact' }
  }
}


// Additional action methods would follow...
```

## Advanced Patterns

### Pagination

For lists that may contain many items, implement pagination:

```typescript
export async function getPaginatedContacts(
  userId: string,
  page = 1,
  pageSize = 10
): Promise<ActionState<{ contacts: SelectContact[], totalCount: number }>> {
  try {
    const offset = (page - 1) * pageSize
    
    const contacts = await db.query.contacts.findMany({
      where: eq(contactsTable.userId, userId),
      limit: pageSize,
      offset,
      orderBy: [desc(contactsTable.updatedAt)]
    })
    
    const [{ count }] = await db
      .select({ count: count() })
      .from(contactsTable)
      .where(eq(contactsTable.userId, userId))
    
    return {
      isSuccess: true,
      message: "Contacts retrieved successfully",
      data: {
        contacts,
        totalCount: Number(count)
      }
    }
  } catch (error) {
    console.error("Error fetching paginated contacts:", error)

    return { isSuccess: false, message: "Failed to retrieve contacts" }
  }
}
```

### Soft Deletes

Consider implementing soft deletes for data that shouldn't be permanently removed:

```typescript
// In schema
deletedAt: timestamp("deleted_at", { withTimezone: true }),

// In queries
const nonDeletedFilter = isNull(contactsTable.deletedAt)

// Soft delete action
export async function softDeleteContact(
  id: string
): Promise<ActionState<void>> {
  try {
    await db
      .update(contactsTable)
      .set({ 
        deletedAt: new Date().toISOString(),
        // Optionally anonymize data
        email: null,
        phone: null 
      })
      .where(eq(contactsTable.id, id))
      
    return {
      isSuccess: true,
      message: "Contact deleted successfully",
      data: undefined
    }
  } catch (error) {
    console.error("Error soft-deleting contact:", error)

    return { isSuccess: false, message: "Failed to delete contact" }
  }
}
```

### Optimistic Updates

Pair your server actions with frontend optimistic updates for better UX:

```typescript
// Client component example
const handleDeleteContact = async (id: string) => {
  // Optimistically update UI
  setContacts(prev => prev.filter(contact => contact.id !== id))
  
  // Call server action
  const result = await deleteContactAction(id)
  
  // Roll back on failure
  if (!result.isSuccess) {
    toast.error(result.message)
    // Reload the contacts
    const refreshedData = await getContactsAction(userId)
    if (refreshedData.isSuccess) {
      setContacts(refreshedData.data)
    }
  }
}
```

### Working with Dates

When handling dates:

1. Store timestamps with timezone information
2. Convert JavaScript Date objects to ISO strings before database operations
3. Use comparison operators that respect time zones

```typescript
// Example of date filtering
const today = new Date()
today.setHours(0, 0, 0, 0)

const todayContacts = await db.query.contacts.findMany({
  where: gte(contactsTable.createdAt, today.toISOString())
})
```

## Database Migrations

### Managing Schema Changes

We use Supabase migrations for database changes:

1. Generate a timestamp for the migration:
   ```ts
   const timestamp = new Date().toISOString().replace(/[-:]/g, "").split(".")[0]
   ```

2. Create migration files in the supabase/migrations directory
   
3. Use Supabase CLI to apply migrations:
   - Development: `supabase db reset`
   - Production: `supabase db push`

4. Always test migrations in a development environment first

5. Include both "up" and "down" migrations where possible

## Performance Considerations

1. **Indexing**: Add indexes for fields frequently used in WHERE clauses
   ```typescript
   email: text("email").notNull().$unique()
   ```

2. **Select Only What You Need**: Use projections to limit returned data
   ```typescript
   const userEmails = await db
     .select({ id: usersTable.id, email: usersTable.email })
     .from(usersTable)
   ```

3. **Batch Operations**: Use batch inserts/updates when possible
   ```typescript
   await db.insert(contactsTable).values(multipleContacts)
   ```

4. **Transaction Support**: Use transactions for multi-step operations
   ```typescript
   await db.transaction(async (tx) => {
     // Multiple operations that succeed or fail together
   })
   ```

## Security Best Practices

1. **Never Trust Client Data**: Always validate all inputs
2. **Row-Level Security**: Enforce user ownership at the database level
3. **Parameterized Queries**: Never use string interpolation in SQL
4. **Rate Limiting**: Implement rate limiting for public-facing endpoints
5. **Audit Trails**: Consider logging sensitive operations

---
> Source: [materialize-labs/ai-optimized-starter-app](https://github.com/materialize-labs/ai-optimized-starter-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
