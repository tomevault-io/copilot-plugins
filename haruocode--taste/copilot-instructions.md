## taste

> Taste is a social app that connects users through their taste in music and media.

# AGENTS.md

## Project

Taste is a social app that connects users through their taste in music and media.
The first release is a Spotify-first MVP focused on music taste matching.

## Product goal

Build a working MVP where users can:

- sign in
- connect Spotify
- fetch top artists and top tracks
- create a taste profile
- discover similar users
- view match reasons
- follow other users

Do not build a generic SNS first.
Do not start with posting, chat, comments, groups, Apple Music, YouTube, or Netflix integrations unless explicitly requested.

## Target stack

- Frontend: React + TypeScript
- App framework: TanStack Start preferred
- Styling: Tailwind CSS
- Backend: Fastify + TypeScript
- Database: PostgreSQL
- ORM: Drizzle ORM preferred
- Validation: Zod
- API style: REST first
- Auth: simple and practical approach; prefer app auth + Spotify OAuth integration
- Package manager: pnpm

## Architecture rules

- Keep frontend and backend clearly separated.
- Prefer a monorepo structure.
- Keep domain logic out of UI components.
- Put matching logic in backend/domain services, not in React components.
- Use strongly typed DTOs and schemas.
- Avoid overengineering. Start simple.

## Initial scope

Implement only the MVP below.

### MVP features

1. Landing page
   - explain the product briefly
   - CTA to sign in / connect Spotify

2. Authentication
   - app-level sign in
   - Spotify OAuth connection

3. Spotify sync
   - fetch current user's profile
   - fetch top artists
   - fetch top tracks
   - store synced data in PostgreSQL

4. My profile
   - display avatar, name, bio
   - display top artists
   - display top tracks

5. Match discovery
   - list similar users
   - show match score
   - show match reasons:
     - common artists
     - common tracks
     - common genres

6. User profile
   - view another user
   - show overlap details
   - follow / unfollow

## Explicit non-goals

Do not implement:

- chat
- comments
- timeline posts
- notifications
- Apple Music integration
- YouTube integration
- Netflix integration
- blockchain features
- AI chatbot features
- vector DB
- microservices
- realtime systems

## Matching algorithm v1

Start with a transparent, explainable scoring model.

Inputs:

- top artists overlap
- top tracks overlap
- genre overlap

Suggested weighting:

- artist overlap: 0.5
- track overlap: 0.3
- genre overlap: 0.2

Requirements:

- score must be explainable
- keep implementation simple
- save computed matches in DB
- avoid expensive realtime full-table recomputation on every request

Suggested approach:

- sync user Spotify data
- normalize artist / track / genre data
- compute matches in a service or background-safe command
- persist results in a `user_matches` table
- API reads from persisted match results

## Data model guidance

Create tables close to the following concepts:

- users
- spotify_accounts
- artists
- tracks
- genres
- user_top_artists
- user_top_tracks
- artist_genres
- follows
- user_matches

You may add practical tables if needed, but do not create unnecessary abstractions.

## API guidance

Prefer endpoints like:

- POST /auth/spotify/connect
- GET /me
- GET /me/profile
- POST /spotify/sync
- GET /matches
- GET /users/:id
- POST /users/:id/follow
- DELETE /users/:id/follow

Adjust naming if needed, but keep the API simple and consistent.

## UI guidance

Focus on clarity over decoration.
Important UI principle:
Users should understand _why_ they match.

Every match card should show:

- match percentage
- common artists
- common tracks count
- common genres

Do not hide the reasons behind the score.

## Code quality rules

- Use TypeScript strictly.
- Avoid `any`.
- Add Zod validation for request/response boundaries.
- Prefer small pure functions for domain logic.
- Write readable code over clever code.
- Keep files reasonably small.
- Add comments only where they improve clarity.
- Do not leave dead code.
- Do not leave TODOs unless unavoidable.

## Testing

Add tests for:

- matching score calculation
- overlap calculation
- follow/unfollow behavior
- Spotify data normalization if practical

Prefer lightweight tests over heavy framework setup.

## Delivery style

When asked to implement something:

1. inspect current repo structure
2. explain the plan briefly
3. make incremental changes
4. show changed files
5. mention how to run and verify

When requirements are ambiguous:

- choose the simplest path consistent with the MVP
- do not expand scope on your own

## Definition of done

A task is done only if:

- code builds
- types pass
- core happy path works
- changed behavior is easy to verify
- summary explains what changed and what remains

## Preferred repo structure

Suggested starting structure:

- apps/web
- apps/api
- packages/shared
- packages/config

You may adapt this if the existing repo already has a better structure.

## First build priority

If starting from scratch, prioritize in this order:

1. monorepo/app skeleton
2. database schema and migrations
3. backend auth and Spotify integration
4. sync and persistence
5. profile pages
6. matching engine
7. follow system
8. polish

## Important product philosophy

Taste is not "another social network with music features".
Taste is a taste-graph product.
The core is:

- user
- content
- preference strength
- similarity
- explanation

Always protect this core.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/haruocode) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
