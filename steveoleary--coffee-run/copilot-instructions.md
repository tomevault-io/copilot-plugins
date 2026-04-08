## coffee-run

> Coffee Run is a fun React + Supabase + Vite app for tracking team coffee runs.

# Coffee Run — AI Coding Agent Instructions

## Intro

Coffee Run is a fun React + Supabase + Vite app for tracking team coffee runs.
It exists to finally answer the eternal office question:

"Whose turn is it to get the coffee?"

## React Component Patterns

### ParticipantContext Loading Race Condition

**CRITICAL:** Components that use `useOptionalParticipantContext()` or `useParticipantContext()` and make Supabase queries MUST check the loading state before querying.

**The Problem:**
When users access the app via magic link (`/p/:token`), the `ParticipantContext` must:
1. Set the participant token in the database session (`app.participant_token`)
2. Allow RLS policies to evaluate correctly

There's a race condition where:
- `ParticipantContext` sets `loading = false`
- Components render and start making queries
- **But** the token may not be fully set in the DB session yet
- RLS policies block the queries → empty results

**The Solution:**
Always check `participantContext?.loading` before making any database queries in `useEffect` hooks.

**Pattern to follow:**

```typescript
const participantContext = useOptionalParticipantContext();

useEffect(() => {
  // CRITICAL: Wait for participant context to finish loading
  if (participantContext?.loading) {
    return;
  }

  if (!accountId) {
    return;
  }

  // Now safe to query - token is set in DB session
  void fetchData();
}, [accountId, fetchData, participantContext?.loading]);
```

**Components that need this pattern:**
- Any view or component accessible via participant routes (`/p/:token/*`)
- Any component that calls Supabase with `participantContext?.token`
- Examples: `IOUsView`, `DashboardView`, `RunHistory`, `NextUp`

**Why it matters:**
- RLS policies check `current_setting('app.participant_token', true)`
- If token not set → policies block queries → empty results
- This causes intermittent "no data" bugs that are hard to reproduce

**When adding new components:**
1. Check if it uses `useOptionalParticipantContext()`
2. Check if it makes Supabase queries in `useEffect`
3. If yes to both → add the loading check

## Supabase Development

**⚠️ IMPORTANT:** Refer to `WORKTREE_SUPABASE.md` for current instance details, including:
- Service Ports (Studio UI, API, DB, etc.)
- Docker Container Names
- Database Connection Commands

### Migrations & Reset

```bash
# Create new migration
npx supabase migration new <descriptive_name>

# List migrations
npx supabase migration list

# Generate TypeScript types from schema
npx supabase gen types --local > src/types/database.ts
```

### Querying the Database

**Via Docker** (when psql isn't installed locally):

> Note: Confirm the container name in `WORKTREE_SUPABASE.md`.

```bash
# Run a single query
docker exec supabase_db_coffee_run psql -U postgres -c "SELECT * FROM participants LIMIT 5;"

# Interactive session
docker exec -it supabase_db_coffee_run psql -U postgres

# Describe table structure
docker exec supabase_db_coffee_run psql -U postgres -c "\d participants"

# List all tables
docker exec supabase_db_coffee_run psql -U postgres -c "\dt"
```

### Testing RLS Policies

The app uses `app.participant_token` for participant authentication:

```sql
-- Simulate participant access
SET app.participant_token = 'some-access-token-here';

-- Test participant can see their group
SELECT * FROM participants 
WHERE account_id IN (
  SELECT account_id FROM participants 
  WHERE access_token = current_setting('app.participant_token', true)
);

-- Reset to owner/admin context
RESET app.participant_token;
```

### Tips

- Pin CLI version in `package.json` to avoid breaking changes
- Use `--local` flag for local dev commands (it's often the default)
- Use the Studio UI for quick schema inspection (URL in `WORKTREE_SUPABASE.md`)
- RLS policies require `app.participant_token` to be set — test both owner and participant contexts

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/steveoleary)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/steveoleary)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
