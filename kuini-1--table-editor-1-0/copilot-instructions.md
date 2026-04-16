## table-editor-1-0

> During your interaction with the user, if you find anything reusable in this project (e.g. version of a library, model name), especially about a fix to a mistake you made or a correction you received, you should take note in the `Lessons` section in the `.cursorrules` file so you will not make the same mistake again.

# Instructions

During your interaction with the user, if you find anything reusable in this project (e.g. version of a library, model name), especially about a fix to a mistake you made or a correction you received, you should take note in the `Lessons` section in the `.cursorrules` file so you will not make the same mistake again. 

You should also use the `.cursorrules` file as a Scratchpad to organize your thoughts. Especially when you receive a new task, you should first review the content of the Scratchpad, clear old different task if necessary, first explain the task, and plan the steps you need to take to complete the task. You can use todo markers to indicate the progress, e.g.
[X] Task 1
[ ] Task 2

Also update the progress of the task in the Scratchpad when you finish a subtask.
Especially when you finished a milestone, it will help to improve your depth of task accomplishment to use the Scratchpad to reflect and plan.
The goal is to help you maintain a big picture as well as the progress of the task. Always refer to the Scratchpad when you plan the next step.

Before providing an answer, ensure that it is not the same as any previous answers that did not work. You must verify this by reviewing our chat history to avoid repeating failed solutions.

Finally, you should always use the official documentation of the library you are using.

# Lessons

## User Specified Lessons

- User Role Management (Owner/Sub-Owner): When implementing role-based access control (RBAC), it's crucial to establish clear ownership hierarchies, where the owner automatically has full control and sub-owners are granted permissions based on their assigned roles. This simplifies the user management process and ensures a secure and scalable system.
- Automatic Permission Creation for Sub-Owners: When automatically generating permissions for new users (e.g., sub-owners), it's essential to default permissions to the lowest level of access (e.g., false for all operations like GET, PUT, POST, DELETE) and provide an interface to easily modify them. This is important for security and user control.
- Table Permission Management: Allowing the owner to modify permissions for sub-owners on a per-table basis is a powerful feature. It's important to clearly define what actions (read, write, delete) sub-owners can perform and make sure these permissions are enforced consistently across the system.
- Database Schema Evolution: When modifying database schemas, it's important to use migrations to track changes and ensure they can be applied consistently across different environments. This includes properly handling column renames, additions, and maintaining data integrity.
- Column Naming Conventions: When working with database columns, maintain consistent naming conventions (e.g., camelCase for specific types like "dwExp" or "wStageWinSolo") to reflect the data type and purpose of the field.
- Supabase Text Search: When performing text pattern matching in Supabase, use the match operator: `query.match({ column: value })`. This works for both text and numeric columns.

## Cursor learned

- When you are editing a file, you should first read the file and understand the context.
- When modifying database schemas, always create proper migration files with clear comments and appropriate constraints.
- Use descriptive column comments to document the purpose of each field in the database.
- Always check the official documentation first before implementing complex solutions.
- Cache user profile and permission data to reduce redundant API calls.
- Use state management for shared data across components.
- When implementing filter functionality, ensure proper handling of different data types (text, number, boolean) with appropriate UI controls and database query methods.
- For boolean fields in forms, use Checkbox components instead of text inputs for better user experience.
- When adding new column types, update all relevant interfaces and components to maintain type safety.
- When adding a new type to an interface, ensure it's consistently added to all related interfaces across components to avoid type errors.
- Performance Optimization for Large Datasets: For tables with 70k+ rows, implement filter debouncing (300-500ms), use estimated counts instead of exact counts for better performance, add database indexes on commonly filtered columns, and show loading states during filter operations. Server-side pagination is already optimal - only load what's needed per page.
- Stripe Subscription Sync: When a user has a `stripe_customer_id` but no `stripe_subscription_id` in their profile, the API should check Stripe for active subscriptions for that customer. This handles cases where webhooks may have failed or subscriptions were created outside the normal flow. Always sync subscription status from Stripe to keep the database in sync.
- Activity Logs Performance: When queries on `activity_logs` table timeout (PostgreSQL error code 57014), add proper indexes. The most critical index is a composite index on `(table_id, created_at DESC)` for queries that filter by table_id and order by created_at. Also add indexes on `table_id`, `created_at`, and `user_id` for optimal performance. Without these indexes, queries with RLS policies can timeout on large datasets.
- For npm-to-pnpm migrations on Windows, if `pnpm install` fails while moving npm-managed modules, generate `pnpm-lock.yaml` with `pnpm import` first, then proceed with a clean pnpm install workflow.

# Scratchpad

Current Task: Linux deploy + local pnpm migration

Task Description:
Update GitHub deployment workflow from Windows to Linux and switch project tooling from npm to pnpm for both CI and local development.

Planned Steps:
[X] Migrate `.github/workflows/deploy.yml` to Linux + pnpm + SSH/SCP deploy flow
[X] Configure repo for local pnpm usage (`packageManager`, `pnpm-lock.yaml`, `.npmrc`, remove `package-lock.json`)
[X] Update README and local runtime messaging from npm commands to pnpm commands
[X] Verify no npm references remain in migrated workflow or local docs/messages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kuini-1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
