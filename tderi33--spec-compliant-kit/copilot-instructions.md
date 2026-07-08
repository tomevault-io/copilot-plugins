## spec-compliant-kit

> Rising Tide OS — a React + Vite + TypeScript SPA for car wash operations management, backed by a hosted Supabase instance (auth, database, edge functions). There is no local backend; all server-side logic runs on Supabase cloud.

# AGENTS.md

## Cursor Cloud specific instructions

### Project overview

Rising Tide OS — a React + Vite + TypeScript SPA for car wash operations management, backed by a hosted Supabase instance (auth, database, edge functions). There is no local backend; all server-side logic runs on Supabase cloud.

### Development commands

| Task | Command |
|------|---------|
| Dev server | `npm run dev` (port 8080) |
| Lint | `npm run lint` |
| Test | `npm run test` |
| Build | `npm run build` |

### Key caveats

- **Authentication**: The app uses Google OAuth via Lovable's auth proxy, restricted to `@risingtidecarwash.com` accounts. The `/login` page (`src/pages/Login.tsx`) **also has an email/password form** that calls `supabase.auth.signInWithPassword` — for local dev/testing you can sign in directly with the `RISING_TIDE_TEST_EMAIL` / `RISING_TIDE_TEST_PASSWORD` credentials (this is the simplest path and what TestSprite uses). Alternatively, you can bypass auth entirely by running the following in the browser console once the dev server is running at `http://localhost:8080`:
  ```js
  const { createClient } = await import('https://esm.sh/@supabase/supabase-js@2');
  const sb = createClient(
    'https://kusikfglatxtgcruquks.supabase.co',
    'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Imt1c2lrZmdsYXR4dGdjcnVxdWtzIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzU4MDgyMzUsImV4cCI6MjA5MTM4NDIzNX0.U_uf-zzFfLrVbUnSqACdUddzNiB9EjExEy41vMS1DTA'
  );
  const { data, error } = await sb.auth.signInWithPassword({
    email: '<TEST_EMAIL>', password: '<TEST_PASSWORD>'
  });
  if (data?.session) {
    localStorage.setItem('sb-kusikfglatxtgcruquks-auth-token', JSON.stringify(data.session));
    location.reload();
  }
  ```
  Replace `<TEST_EMAIL>` and `<TEST_PASSWORD>` with the values from the `RISING_TIDE_TEST_EMAIL` and `RISING_TIDE_TEST_PASSWORD` secrets. After reload, the app will recognize the session and show authenticated pages.
- **Localhost OAuth does not work**: The Lovable auth proxy only allows redirects to the production domain. Clicking "Sign in with Google" on `localhost:8080` will 404 on the redirect. Use the email/password form on the login page (or the console-based bypass above) for local testing.
- **Test user must have an admin role**: The `useRoleAccess` hook returns `false` for all pages when the user has no role (`role` is null). Without a row in the `user_roles` table, dashboard pages render blank. The test account must have `role = 'admin'` in the `user_roles` table to access all pages.
- **Pre-existing lint errors**: The codebase has ~109 existing ESLint errors (mostly `@typescript-eslint/no-explicit-any`). ESLint exits with code 1 due to these pre-existing issues. Do not attempt to fix them unless specifically asked.
- **Supabase backend is remote**: The `.env` file contains the Supabase project URL and anon key. No local Supabase setup or Docker is needed for frontend development.
- **Multiple lockfiles**: The repo contains both `package-lock.json` and `bun.lock`/`bun.lockb`. Use `npm install` (matching `package-lock.json`) for dependency installation.
- **Vite dev server binds to `::` (all interfaces)** on port 8080 with HMR overlay disabled.

### Member Amenities module (PRD Part B)

The Member Amenities system is built slice-by-slice from `docs/amenities/Member Amenities System - PRD.md` (Part B) plus the `Cursor Build Kit/` execution plan, prompt playbook, and verification checklist. When touching anything amenities-related, read these first:

- **Guardrails live in `.cursor/rules/`** and load automatically: `00-rt-os-conventions.mdc` (always-on repo conventions), `10-amenities-module.mdc` (amenities specifics, scoped by glob), `20-open-holds.mdc` (always-on hard stops), `30-firmware.mdc` (scoped to `firmware/**`).
- **Three open holds must NOT be resolved in code without Tom's sign-off** (see `20-open-holds.mdc`): (1) the leaderboard blended-score weights/tie-break, (2) ADA / member accessibility dimensions and masked-entry UX, (3) the §8 verification sampling minimum-n / 100%-review policy. Build the surrounding structure and leave a flagged `// HOLD:` stub instead of inventing a value.
- **Feature flag**: gate all amenities nav/Settings entries behind `featureFlag: "member-amenities"`. The flag is registered in `src/lib/pageRegistry.ts` but is intentionally **default-off** (not in `DEFAULT_FLAGS` in `src/hooks/useFeatureFlags.ts`). Enable it locally with `VITE_FEATURE_FLAGS="member-amenities"`.
- **Firmware**: the ESP32-S3 post-controller lives at `firmware/amenities-post-controller/` (C++/PlatformIO, not the web stack). It builds in its own CI job (`.github/workflows/firmware-build.yml`, gated on `firmware/**`) — do not mix firmware deps into the web manifests or vice-versa. `docs/server-contract.md` inside it is the shared device↔server contract; any device-endpoint change is one atomic PR touching the contract, the edge function(s), and the firmware. `secrets.h` is git-ignored (only `secrets.example.h` is committed).

### ESP32-S3 amenities firmware (PlatformIO) — build/test

Run all commands from `firmware/amenities-post-controller/`:

| Task | Command |
|------|---------|
| Build device firmware | `pio run` (default env `esp32s3_n16r8`) |
| Host unit tests | `pio test -e native` |

- **PlatformIO (`pio`) is a Python tool**, installed via `pip install platformio` into `~/.local/bin`. That dir is on `PATH` for login shells via `~/.profile` (also added to `~/.bashrc`). If `pio` is "command not found" in a non-login shell, run `export PATH="$HOME/.local/bin:$PATH"`.
- **`secrets.h` is git-ignored** and is auto-created from `secrets.example.h` by the startup update script (the committed template's placeholder values are all the compiler needs — no real credentials). If it is ever missing, recreate it with `cp src/secrets.example.h src/secrets.h`.
- The first `pio run` downloads the `espressif32 @ 6.9.0` toolchain (~minutes, one-time); it is cached under `~/.platformio` afterward. The `native` env only needs the host compiler + Unity.
- The `esp32s3_n16r8` platform is **pinned** (`espressif32 @ 6.9.0` = Arduino core 2.0.17) on purpose — see the note in `platformio.ini` before bumping it.

---
> Source: [tderi33/spec-compliant-kit](https://github.com/tderi33/spec-compliant-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
