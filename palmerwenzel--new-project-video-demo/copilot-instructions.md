## drizzle-rules-example

> Rules for Drizzle ORM in the context of this project's server.

# Drizzle ORM Best Practices

This document outlines the best practices for managing database schemas and migrations using Drizzle ORM within the our project's server application.

## Workflow: TypeScript-First Schema Management

We adopt a **TypeScript-first** approach for managing the database schema. This means:

1.  **Source of Truth:** The TypeScript schema definition files located in `server/src/db/schema/` are the definitive source of truth for the database structure.
2.  **Schema Changes:** All changes to the database structure (adding tables, columns, constraints, etc.) MUST be made by modifying these TypeScript files first.
3.  **Generating Migrations:** After modifying the TypeScript schema, the `drizzle-kit generate` command is used to automatically create corresponding SQL migration files (`*.sql`).
4.  **Applying Migrations:** The generated SQL migrations are then applied to the database (both local development and production environments) using the standard Supabase migration workflow (e.g., `supabase db push` locally, CI/CD deployment).

**Rationale:**

- **Type Safety:** Leverages TypeScript's full potential for defining and interacting with the database schema.
- **Consistency:** Ensures the database schema always matches the application's understanding (defined in TypeScript).
- **Maintainability:** Changes are tracked and managed within the codebase alongside application logic.
- **Avoids `pull` Limitations:** Sidesteps potential issues with `drizzle-kit pull` overwriting manual schema adjustments or struggling with split file structures.

## Directory Structure (`server/src/db/schema/`)

- **Individual Table Files:** Each database table should have its own schema definition file (e.g., `users.schema.ts`, `pets.schema.ts`). This prevents circular dependencies and improves organization.
- **Shared Definitions:** Common elements like PostgreSQL enums should be defined in a dedicated shared file (e.g., `shared.schema.ts`) and imported where needed.
- **Relations File:** Database relationships should be defined separately, typically in `relations.ts`, importing the necessary table schemas.

## Process for Schema Changes

1.  **Modify TypeScript Schema:** Make the desired changes directly in the relevant `*.schema.ts` file(s) within `server/src/db/schema/`.
2.  **Generate SQL Migration:** Run the following command from the `server/` directory:

    ```bash
    npx drizzle-kit generate
    ```

    > **Note:** In the SAT Monsters project, the preferred way to run this command and handle the generated output is by using the root-level script: `pnpm run db:generate-migration`. This script automatically runs `drizzle-kit generate` and then moves/renames the output file to the correct `supabase/migrations/` directory with the proper naming convention.
    > If you run `npx drizzle-kit generate` manually within the `server/` directory, you **must** remember to manually move the generated `.sql` file from `server/drizzle/migrations/` to `server/supabase/migrations/` and rename it according to the project's timestamp convention (e.g., `YYYYMMDDHHmmss_description.sql`) before it can be used by the Supabase CLI.

    This command reads the changes in your `.ts` files, compares them to the previous schema state (using snapshots typically stored in the migration output directory), and generates a new `.sql` file in the directory specified by the `out` property in `drizzle.config.ts` (e.g., `./drizzle/migrations/`).

3.  **Review SQL Migration:** Inspect the generated `.sql` file to ensure it accurately reflects the intended changes.
4.  **Apply Migration:**
    - **Local:** Use Supabase CLI: `supabase db push` (if applicable for local dev) or apply manually.
    - **Production:** Integrate the new SQL migration file into your deployment process (e.g., Supabase migration CI/CD).
5.  **Commit Changes:** Commit the modified `*.schema.ts` files AND the newly generated `.sql` migration file.

## `drizzle.config.ts` Setup

The `server/drizzle.config.ts` file should be configured to support this workflow:

```typescript
import type { Config } from "drizzle-kit";
import * as dotenv from "dotenv";

dotenv.config({ path: ".env" });

if (!process.env.DATABASE_URL) {
  throw new Error("DATABASE_URL environment variable is not set");
}

export default {
  // Point schema to the DIRECTORY containing all individual *.schema.ts files
  schema: "./src/db/schema/",
  // Point out to the directory where SQL migrations should be GENERATED
  out: "./drizzle/migrations", // Or './prisma/migrations' if preferred, but './drizzle/' is conventional
  dialect: "postgresql",
  dbCredentials: {
    url: process.env.DATABASE_URL,
  },
  verbose: true,
  strict: true,
} satisfies Config;
```

By following this TypeScript-first approach, we ensure a robust, type-safe, and maintainable system for evolving our database schema.

### Migration File Handling

1.  **Generation**: When you run `(cd server && npx drizzle-kit generate)`, Drizzle Kit creates SQL migration files in `server/drizzle/migrations/` based on changes in your schema (`server/src/db/schema/*`).
2.  **Timestamping & Renaming (Manual or Scripted)**: For Supabase to recognize and apply these migrations in the correct order alongside other custom migrations, they need to be:
    - Moved from `server/drizzle/migrations/` to `server/supabase/migrations/`.
    - Renamed to follow a `YYYYMMDDHHMMSS_description.sql` format (e.g., `20231027143000_add_user_profiles.sql`).
    - The `_scripts/db_generate_migration.sh` script automates this process. It finds the latest Drizzle-generated file, prompts for a description, and renames/moves it appropriately.
3.  **Application**: Migrations in `server/supabase/migrations/` are applied by Supabase CLI commands, typically `supabase db reset` for local development and test environments, or `supabase migration up` (though `db reset` is often preferred for its clean-slate approach locally).

### Test Database Initialization (`test_db_reinit.sh`)

This script (located in `_scripts/`) is crucial for consistently setting up your test database. It typically performs the following:

- Wipes the existing test database.
- Copies the latest Drizzle-generated schema (which should represent your full current schema after Drizzle generation) to `server/supabase/migrations/0000_initial_schema.sql`.
- Uses `supabase db reset` to apply this initial schema along with any other custom SQL migrations (like triggers) you have in `server/supabase/migrations/`.

This ensures your test environment accurately reflects your Drizzle schema plus any manual SQL additions.

### Important Considerations

- **Single Source of Truth**: While Drizzle manages schema changes, the `server/supabase/migrations/` directory becomes the ultimate source of truth for what gets applied to your Supabase databases (local and deployed).
- **Custom SQL**: For things Drizzle doesn't handle (or handle easily), like complex triggers, RLS policies not expressible via Drizzle, or specific data seeding steps that must run with migrations, create separate SQL files in `server/supabase/migrations/` with appropriate timestamps after your Drizzle-generated schema file.
- **`0000_initial_schema.sql`**: The `test_db_reinit.sh` script treats the Drizzle-generated output as a complete baseline schema, naming it `0000_initial_schema.sql`. Subsequent custom migrations should have later timestamps.
- **Review Migrations**: Always review the SQL generated by Drizzle Kit before committing and applying it, especially for potentially destructive changes.

By following this workflow, you can leverage Drizzle Kit for schema management while ensuring compatibility and proper ordering with Supabase's migration system.

---
> Source: [palmerwenzel/new-project-video-demo](https://github.com/palmerwenzel/new-project-video-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
