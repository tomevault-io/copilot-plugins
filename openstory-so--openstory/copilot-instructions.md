## drizzle-patterns

> Drizzle ORM patterns and database conventions


# Drizzle ORM Patterns

## Schema Definition

```typescript
import {
  pgTable,
  uuid,
  text,
  timestamp,
  boolean,
  jsonb,
  integer,
} from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

export const teams = pgTable('teams', {
  id: uuid('id').primaryKey().defaultRandom(),
  name: text('name').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

export const users = pgTable('users', {
  id: uuid('id').primaryKey().defaultRandom(),
  email: text('email'),
  isAnonymous: boolean('is_anonymous').default(true).notNull(),
  teamId: uuid('team_id').references(() => teams.id),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});

export const sequences = pgTable('sequences', {
  id: uuid('id').primaryKey().defaultRandom(),
  name: text('name').notNull(),
  script: text('script'),
  teamId: uuid('team_id')
    .references(() => teams.id)
    .notNull(),
  createdBy: uuid('created_by')
    .references(() => users.id)
    .notNull(),
  styleId: uuid('style_id').references(() => styles.id),
  metadata: jsonb('metadata'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

export const frames = pgTable('frames', {
  id: uuid('id').primaryKey().defaultRandom(),
  sequenceId: uuid('sequence_id')
    .references(() => sequences.id, { onDelete: 'cascade' })
    .notNull(),
  order: integer('order').notNull(),
  description: text('description').notNull(),
  thumbnailUrl: text('thumbnail_url'),
  status: text('status').notNull().default('pending'), // pending, processing, completed, failed
  error: text('error'),
  createdBy: uuid('created_by')
    .references(() => users.id)
    .notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

export const styles = pgTable('styles', {
  id: uuid('id').primaryKey().defaultRandom(),
  name: text('name').notNull(),
  description: text('description'),
  teamId: uuid('team_id')
    .references(() => teams.id)
    .notNull(),
  styleData: jsonb('style_data').notNull(), // Style Stack JSON
  createdBy: uuid('created_by')
    .references(() => users.id)
    .notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

// Relations
export const teamsRelations = relations(teams, ({ many }) => ({
  users: many(users),
  sequences: many(sequences),
  styles: many(styles),
}));

export const usersRelations = relations(users, ({ one, many }) => ({
  team: one(teams, {
    fields: [users.teamId],
    references: [teams.id],
  }),
  sequences: many(sequences),
  frames: many(frames),
  styles: many(styles),
}));

export const sequencesRelations = relations(sequences, ({ one, many }) => ({
  team: one(teams, {
    fields: [sequences.teamId],
    references: [teams.id],
  }),
  frames: many(frames),
  style: one(styles, {
    fields: [sequences.styleId],
    references: [styles.id],
  }),
  creator: one(users, {
    fields: [sequences.createdBy],
    references: [users.id],
  }),
}));

export const framesRelations = relations(frames, ({ one }) => ({
  sequence: one(sequences, {
    fields: [frames.sequenceId],
    references: [sequences.id],
  }),
  creator: one(users, {
    fields: [frames.createdBy],
    references: [users.id],
  }),
}));

export const stylesRelations = relations(styles, ({ one, many }) => ({
  team: one(teams, {
    fields: [styles.teamId],
    references: [teams.id],
  }),
  creator: one(users, {
    fields: [styles.createdBy],
    references: [users.id],
  }),
  sequences: many(sequences),
}));
```

## Database Client Setup

```typescript
// apps/backend/src/db/index.ts
import { drizzle } from 'drizzle-orm/postgres-js';
import postgres from 'postgres';
import * as schema from './schema';

const connectionString = process.env.POSTGRES_URL;

if (!connectionString) {
  throw new Error('POSTGRES_URL environment variable is not set');
}

// Create PostgreSQL connection
const queryClient = postgres(connectionString);

// Create Drizzle instance
export const db = drizzle(queryClient, { schema });

// Type exports
export type Database = typeof db;
```

## Query Patterns

```typescript
import { db } from './index';
import { sequences, frames } from './schema';
import { eq, and, desc, asc, count } from 'drizzle-orm';

// Simple select with relations
export async function getSequenceWithFrames(sequenceId: string) {
  return db.query.sequences.findFirst({
    where: eq(sequences.id, sequenceId),
    with: {
      frames: {
        orderBy: [asc(frames.order)],
      },
      style: true,
      creator: {
        columns: {
          id: true,
          email: true,
          isAnonymous: true,
        },
      },
    },
  });
}

// List with pagination
export async function listSequences(
  teamId: string,
  page: number = 1,
  pageSize: number = 20
) {
  const offset = (page - 1) * pageSize;

  const [items, totalCount] = await Promise.all([
    db.query.sequences.findMany({
      where: eq(sequences.teamId, teamId),
      limit: pageSize,
      offset,
      orderBy: [desc(sequences.createdAt)],
      with: {
        frames: {
          columns: {
            id: true,
            status: true,
          },
        },
        style: {
          columns: {
            id: true,
            name: true,
          },
        },
      },
    }),
    db
      .select({ count: count() })
      .from(sequences)
      .where(eq(sequences.teamId, teamId)),
  ]);

  return {
    items,
    total: totalCount[0].count,
    page,
    pageSize,
    totalPages: Math.ceil(totalCount[0].count / pageSize),
  };
}

// Insert with returning
export async function createSequence(data: typeof sequences.$inferInsert) {
  const [newSequence] = await db.insert(sequences).values(data).returning();
  return newSequence;
}

// Update
export async function updateSequence(
  id: string,
  data: Partial<typeof sequences.$inferInsert>
) {
  const [updated] = await db
    .update(sequences)
    .set({ ...data, updatedAt: new Date() })
    .where(eq(sequences.id, id))
    .returning();
  return updated;
}

// Delete with cascade
export async function deleteSequence(id: string) {
  await db.delete(sequences).where(eq(sequences.id, id));
  // Frames are automatically deleted via onDelete: 'cascade'
}

// Complex query with joins
export async function getFramesWithSequenceInfo(userId: string) {
  return db
    .select({
      frameId: frames.id,
      frameDescription: frames.description,
      frameStatus: frames.status,
      sequenceName: sequences.name,
      sequenceId: sequences.id,
    })
    .from(frames)
    .innerJoin(sequences, eq(frames.sequenceId, sequences.id))
    .where(eq(sequences.createdBy, userId))
    .orderBy(desc(frames.createdAt));
}
```

## Transaction Pattern

```typescript
import { db } from './index';
import { sequences, frames } from './schema';
import { eq } from 'drizzle-orm';

export async function transferSequenceOwnership(
  sequenceId: string,
  newTeamId: string,
  newUserId: string
) {
  await db.transaction(async (tx) => {
    // Update sequence ownership
    await tx
      .update(sequences)
      .set({ teamId: newTeamId, createdBy: newUserId, updatedAt: new Date() })
      .where(eq(sequences.id, sequenceId));

    // Update all frames ownership
    await tx
      .update(frames)
      .set({ createdBy: newUserId, updatedAt: new Date() })
      .where(eq(frames.sequenceId, sequenceId));
  });
}

// Transaction with error handling
export async function createSequenceWithFrames(
  sequenceData: typeof sequences.$inferInsert,
  framesData: Array<typeof frames.$inferInsert>
) {
  return db.transaction(async (tx) => {
    // Create sequence
    const [sequence] = await tx
      .insert(sequences)
      .values(sequenceData)
      .returning();

    // Create frames
    const createdFrames = await tx
      .insert(frames)
      .values(
        framesData.map((frame) => ({
          ...frame,
          sequenceId: sequence.id,
        }))
      )
      .returning();

    return {
      sequence,
      frames: createdFrames,
    };
  });
}
```

## Migration Pattern

```typescript
// drizzle.config.ts
import type { Config } from 'drizzle-kit';

export default {
  schema: './src/db/schema/*',
  out: './src/db/migrations',
  driver: 'pg',
  dbCredentials: {
    connectionString: process.env.POSTGRES_URL!,
  },
} satisfies Config;
```

```bash
# Generate migration
bun run drizzle-kit generate:pg

# Run migrations
bun run drizzle-kit push:pg

# Drop migration
bun run drizzle-kit drop
```

## Type-Safe Queries

```typescript
import { db } from './index';
import { sequences } from './schema';

// Infer types from schema
type Sequence = typeof sequences.$inferSelect;
type NewSequence = typeof sequences.$inferInsert;

// Use types in functions
export async function getSequence(id: string): Promise<Sequence | undefined> {
  return db.query.sequences.findFirst({
    where: eq(sequences.id, id),
  });
}

export async function createSequence(data: NewSequence): Promise<Sequence> {
  const [sequence] = await db.insert(sequences).values(data).returning();
  return sequence;
}
```

## Prepared Statements

```typescript
import { db } from './index';
import { sequences } from './schema';
import { eq } from 'drizzle-orm';

// Prepare statement for better performance
const getSequenceById = db.query.sequences
  .findFirst({
    where: eq(sequences.id, placeholder('id')),
  })
  .prepare('get_sequence_by_id');

// Use prepared statement
export async function findSequence(id: string) {
  return getSequenceById.execute({ id });
}
```

## Aggregations

```typescript
import { db } from './index';
import { frames, sequences } from './schema';
import { eq, count, sql } from 'drizzle-orm';

// Count frames by status
export async function getFrameStatsBySequence(sequenceId: string) {
  return db
    .select({
      status: frames.status,
      count: count(),
    })
    .from(frames)
    .where(eq(frames.sequenceId, sequenceId))
    .groupBy(frames.status);
}

// Custom SQL aggregation
export async function getSequenceProgress(sequenceId: string) {
  const result = await db
    .select({
      total: count(),
      completed: sql<number>`count(*) filter (where ${frames.status} = 'completed')`,
      pending: sql<number>`count(*) filter (where ${frames.status} = 'pending')`,
      processing: sql<number>`count(*) filter (where ${frames.status} = 'processing')`,
      failed: sql<number>`count(*) filter (where ${frames.status} = 'failed')`,
    })
    .from(frames)
    .where(eq(frames.sequenceId, sequenceId));

  return result[0];
}
```

---
> Source: [openstory-so/openstory](https://github.com/openstory-so/openstory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
