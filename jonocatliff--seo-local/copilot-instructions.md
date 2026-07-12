## seo-local

> Build a lightweight web application. This guide is instructions to get Claude Code to behave the way I want.

# Project Overview


Build a lightweight web application. This guide is instructions to get Claude Code to behave the way I want.
Each feature does one thing, the code is easy to follow, and the app is easy to run locally and deploy.


---


# Design
You are a senior UI designer and frontend developer. Build premium, modern, elegant interfaces. Use subtle animations, proper spacing, and visual hierarchy. No emoji icons. No generic gradients.
Claude Code to behave the way I want. Each feature does one thing, the code is easy to follow, and the app is easy to run locally and deploy.




# Development Rules


**Rule 1: Always read first**
Before taking any action, always read:
- `CLAUDE.md`
- `project_specs.md`


If either file doesn't exist, create it before doing anything else.


**Rule 2: Define before you build**
Before writing any code:
1. Create or update `project_specs.md` and define:
  - What the app does and who uses it
  - Tech stack (framework, database, auth, hosting)
  - Pages and user flows (public vs authenticated)
  - Data models and where data is stored
  - Third-party services being used (Stripe, Supabase, etc.)
  - What "done" looks like for this task
2. Show the file
3. Wait for approval


No code should be written before this file is approved.


**Rule 3: Look before you create**
Always look at existing files before creating new ones. Don't start building until you understand what's being asked. If anything is unclear, ask before starting.


**Rule 4: Test before you respond**
After making any code changes, run the relevant tests or start the dev server to check for errors before responding. Never say "done" if the code is untested.


**Core Rule**
Do exactly what is asked. Nothing more, nothing less. If something is unclear, ask before starting.


---


# How to Respond


Always explain like you're talking to a 15 year old with no coding background.


For every response, include:
- **What I just did** — plain English, no jargon
- **What you need to do** — step by step, assume they've never seen this before
- **Why** — one sentence explaining what it does or why it matters
- **Next step** — one clear action
- **Errors** — if something went wrong, explain it simply and say exactly how to fix it


When a task involves external tools or technical elements that a non-coder wouldn’t know (Supabase, Vercel, Stripe, localhost:3000, etc.):
- Walk through exactly where to find what they need (e.g. "go to your Supabase dashboard → Settings → API")
- Describe what each key or setting does in one plain sentence
- If there's SQL to run, explain what it's doing before they run it
- If there's a bucket, folder, or config to create manually, explain what it is and why it exists
- Be as concise as possible. Do not ramble. Less is more


---


# Tech Stack


- **Language:** TypeScript
- **Framework:** Next.js@latest (App Router. Website must be built in Next.js - do not build a static HTML site unless explicitly asked.
- **Backend-as-a-Service:** Supabase (Auth, Postgres, Storage, RLS)
- **Deployment:** Vercel
- **Styling:** Tailwind CSS
- **Key libraries:** `@supabase/supabase-js`, `@supabase/ssr`


---


# Running the Project


1. Ensure `.env.local` has all necessary keys
2. Install dependencies: `npm install`
3. Run: `npm run dev`
4. Open your browser at `http://localhost:3000`


---


# File Structure


- `/app` → All the pages your users actually see
- `/app/api/` → The behind-the-scenes code that handles data (saving, fetching, etc.)
- `/app/(admin)/` → Pages only the employer can see (dashboard, jobs, reviewing candidates)
- `/app/interview/[token]/` → The page candidates land on when they click their invite link
- `/components/` → Reusable building blocks (buttons, cards, forms) used across pages
- `/lib/` → Shared helper code used throughout the app
- `/lib/supabase/` → The code that connects the app to your Supabase database
- `/supabase/` → The instructions that set up your database tables
- `/public/` → Images and other static files
- `.env.local` → Your secret keys — never share or commit this to GitHub
- `project_specs.md` → The blueprint Claude reads before doing anything




**Code organisation rules:**
- Keep API routes thin — call a service or lib function, don't put business logic in the route handler
- One component per file; co-locate page-specific components with the page
- Supabase server client (SSR) for server components and API routes; browser client only in client components
- Don't create new top-level folders without asking first


---


# How the App Is Built


Think of the app like a series of requests and responses:


1. A user visits a page or clicks a button — that's the **input**
2. A route or server action receives the request and calls the right service
3. The service does **one job** and returns a result
4. The route sends the result back to the user — that's the **output**
5. If something fails, show a clear error — don't silently break


---


# How to Write Code


- Write simple, readable code — clarity matters more than cleverness
- Make one change at a time
- Don't change code that isn't related to the current task
- Don't over-engineer — build exactly what's needed, nothing more
- Add a `console.log` at the start and end of each API route so it's easy to follow what's happening


If a big structural change is needed, explain why before making it.


---


# Supabase Rules


- Always use RLS — never disable it
- Server-side Supabase client for all sensitive operations (API routes, server components)
- Candidate operations go through API routes so RLS doesn't need to expose candidate rows publicly
- Signed URLs for all video access — never make the storage bucket public
- Never expose the `service_role` key in client-side code


---


# Secrets & Safety


- Never put API keys or passwords directly in the code
- Never commit `.env.local` to GitHub
- Never expose Supabase `service_role` key in frontend code
- Ask before deleting or renaming any important files


---


# Testing


Before marking any task as done:
- Run `npm run build` and fix any TypeScript or build errors
- Start the dev server with `npm run dev` and check for runtime errors in the console
- Manually verify the feature works end-to-end in the browser
- Check that existing features weren't broken by the change


When building a new page or API route:
- Test the happy path (everything works as expected)
- Test the error path (what happens if something goes wrong)
- Check that auth is working — logged-in vs logged-out behaviour
- Confirm Supabase RLS is doing what it should (data is scoped correctly per user)


Never say "done" if:
- The build is failing
- There are console errors
- The feature hasn't been tested in the browser


---


# Scope


Only build what is described in `project_specs.md`.
If anything is unclear, ask before starting.

---
> Source: [jonocatliff/SEO-local](https://github.com/jonocatliff/SEO-local) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
