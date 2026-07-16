## tunes

> Instructions for AI coding agents working in this repo. Keep this file lean — for domain language and rationale, follow the links out to `docs/`, don't duplicate them here.

# AGENTS.md

Instructions for AI coding agents working in this repo. Keep this file lean — for domain language and rationale, follow the links out to `docs/`, don't duplicate them here.

## Project overview

Tunes: a personal Next.js/Supabase app for tracking a solo musician's repertoire (Songs), personal notes on each (User Songs), and liked recordings — separate from casual playlists. Solo personal project, early stage, dormant between sessions. See [docs/domain-model.md](docs/domain-model.md) for why it exists and the domain vocabulary.

## Repo layout

```
src/app/            Next.js App Router pages (route = folder path)
  page.tsx            home
  login/, signup/      auth pages
  add-tune/            create a Song (route name predates the Song rename — see docs/direction/song-user-song-split.md)
  tune/[id]/           a Song's detail page
  tune/[id]/add-recording/   add a Recording to a Song
  recording/[id]/      a Recording's detail page
src/components/      shared React components (e.g. GlobalPlayer, the persistent player)
src/lib/             infra clients (supabaseClient, fonts)
src/types/           shared TS types
src/utils/           helpers (e.g. youtube.ts)
docs/                domain model, ADRs, direction notes (issues + ideas by subject) — see docs/README.md
```

## Tech stack

- Next.js 16 (App Router), React 19, TypeScript
- Tailwind CSS
- Supabase (`@supabase/supabase-js`) — auth + Postgres, client in `src/lib/supabaseClient.js`
- Persistent custom player for Recordings, backed by the YouTube IFrame API (`src/components/GlobalPlayer.tsx`, `src/utils/youtube.ts`) — see [docs/direction/music-player.md](docs/direction/music-player.md)

## Commands

```
npm run dev      # start dev server (localhost:3000)
npm run build    # production build
npm run start    # run production build
```

No test suite exists yet (see [docs/direction/testing.md](docs/direction/testing.md)).

## Rules and guardrails

- **Terminology**: use "Song" in new code/UI copy, not "Tune" — the domain was renamed ([ADR-0003](docs/adr/0003-song-canonical-user-song-personal.md)) but existing code/DB (`Tune` type, `public.tunes` table, `/tune`, `/add-tune` routes) hasn't caught up yet ([docs/direction/song-user-song-split.md](docs/direction/song-user-song-split.md)). Don't treat that existing code as the preferred pattern to copy; don't rename it unprompted either — it's a real migration, not a drive-by fix.
- **Song creation is not admin-gated**: any user can create a new Song on no search match; don't add approval/moderation gates here ([ADR-0003](docs/adr/0003-song-canonical-user-song-personal.md)).
- **Lead Sheets are private by default and publishing is admin-only**, never self-service or automatic — don't build a user-facing "publish" action ([ADR-0002](docs/adr/0002-lead-sheets-admin-gated-publishing.md)).
- **One email = one account** across auth methods (password + Google) — don't treat them as separate identities ([ADR-0001](docs/adr/0001-unique-email-account-linking.md)).
- Before "fixing" something you notice in passing, check [docs/direction/](docs/direction/) for a file on that subject — it may already be a known, deliberately-not-yet-fixed issue, or something the owner has other plans for. A hit there needs a conversation, not a silent fix.
- This is a solo hobby project the owner returns to after long gaps and is also using to learn broader dev practices — prefer clear, conventional patterns over clever ones, and explain non-obvious choices.

---
> Source: [jdyck/tunes](https://github.com/jdyck/tunes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
