## partydrinks

> The Party Drinks application uses Supabase for database, authentication, and file storage. The setup and configuration are documented in [SUPABASE_SETUP.md](mdc:SUPABASE_SETUP.md).

# Supabase Integration

The Party Drinks application uses Supabase for database, authentication, and file storage. The setup and configuration are documented in [SUPABASE_SETUP.md](mdc:SUPABASE_SETUP.md).

## Configuration

- Environment variables defined in `.env.local` file:
  ```
  NEXT_PUBLIC_SUPABASE_URL=your-supabase-project-url
  NEXT_PUBLIC_SUPABASE_ANON_KEY=your-supabase-anon-key
  SUPABASE_URL=your-supabase-project-url
  SUPABASE_SERVICE_ROLE_KEY=your-supabase-service-role-key
  ```

- Supabase client configured in [lib/supabase.ts](mdc:lib/supabase.ts):
  - Browser client with user session persistence 
  - Server client with service role for admin operations

## Database Usage

- Strongly typed with TypeScript using [types/supabase.ts](mdc:types/supabase.ts)
- Database operations in [lib/db.ts](mdc:lib/db.ts)
- Row-level security policies for data protection

## Storage Usage

- Storage operations in [lib/storage.ts](mdc:lib/storage.ts)
- Two buckets:
  - `profile-images`: For user profile pictures
  - `tweet-images`: For images attached to tweets
- Initialization endpoint at [app/api/init-storage/route.ts](mdc:app/api/init-storage/route.ts)

## API Integration Points

- User registration: [app/api/auth/register/route.ts](mdc:app/api/auth/register/route.ts)
- Image upload during signup: [app/auth/signup/page.tsx](mdc:app/auth/signup/page.tsx)
- Leaderboard data retrieval
- Tweet creation and retrieval
- Drink and cigarette tracking

## Database Schema

The database schema includes tables for users, drinks, drink points, cigarettes, and tweets, with relationships between them. The full schema can be found in [SUPABASE_SETUP.md](mdc:SUPABASE_SETUP.md).

## Troubleshooting

Common issues and their solutions:
- Storage bucket creation issues: Use the `/api/init-storage` endpoint
- Image upload failures: Check browser console for specific error messages
- Authentication errors: Verify Supabase credentials and roles

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/viftode4)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/viftode4)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
