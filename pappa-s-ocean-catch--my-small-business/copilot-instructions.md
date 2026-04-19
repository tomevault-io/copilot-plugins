## my-small-business

> - do not using any type in typescript code, all type need to be well defined to avoid lint and build error

- do not using any type in typescript code, all type need to be well defined to avoid lint and build error
- CRITICAL: In Supabase RLS policies, always use `profiles.id = auth.uid()` NOT `profiles.user_id = auth.uid()`. The profiles table uses `id` as the primary key that references `auth.users(id)`, not a separate `user_id` column.
- When creating RLS policies that check admin roles, use: `exists (select 1 from public.profiles where profiles.id = auth.uid() and profiles.role_slug = 'admin')`
- CRITICAL: When querying profiles table in client code, always use `.eq("id", user.id)` NOT `.eq("user_id", user.id)`. The profiles table uses `id` as the primary key that references `auth.users(id)`.
- CRITICAL: When creating foreign key references to user data, always reference `public.profiles(id)` NOT `auth.users(id)` for consistency with RLS policies and data relationships.
- CRITICAL: Never silently fail database queries. Always display errors to users with clear error messages and suggested solutions (e.g., "Run database migration").
- CRITICAL: Always handle database errors gracefully with proper error states, user feedback, and debugging information.
- CRITICAL: When creating new database migrations, ALWAYS use the current timestamp format YYYYMMDDHHMMSS to ensure migrations are applied in the correct order. Use `date +"%Y%m%d%H%M%S"` to get the current timestamp.
- CRITICAL: Migration file names must be incremental and newer than existing migrations to avoid "Found local migration files to be inserted before the last migration on remote database" errors.
- CRITICAL: In SQL functions/migrations, avoid reserved keywords and ambiguous identifiers for parameter names (e.g., use `current_ts` instead of `current_time`). Prefer snake_case and explicit types.
- CRITICAL: In SQL functions, do not shadow built-ins or use names that can clash with functions (e.g., `time`, `user`, `current_time`).
- CRITICAL: Validate SQL function parameters and return types; test functions with `select` before using in triggers/policies.
- CRITICAL: PostgreSQL does NOT support `IF NOT EXISTS` with `ALTER TABLE ADD CONSTRAINT`. Use DO blocks instead:
  ```sql
  DO $$ 
  BEGIN
      IF NOT EXISTS (
          SELECT 1 FROM pg_constraint 
          WHERE conname = 'constraint_name'
      ) THEN
          ALTER TABLE table_name 
          ADD CONSTRAINT constraint_name 
          CHECK (condition);
      END IF;
  END $$;
  ```
- CRITICAL: When sending emails, always use the `EMAIL_FROM` environment variable for the `from` address. Never hardcode email addresses.
- CRITICAL: NEVER run `npm run build` - ALWAYS use `npm run build:agent` instead to verify the build is green without any error after some change or before reporting big task done
- CRITICAL: The build command is `npm run build:agent` NOT `npm run build` - this is a hard rule that must be followed
- Do not using emoji icon on the react component, using icon from react-icons instead
- All delete action need to be guard by ConfirmationDialog (share component)
- Prefer React Icons over custom SVGs. Do not add inline <svg> icons in components; use icons from react-icons (consistent styling, a11y, theming). If an asset must be used, place it in public/ and render with Next Image or as background, not inline SVG.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Pappa-s-Ocean-Catch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
