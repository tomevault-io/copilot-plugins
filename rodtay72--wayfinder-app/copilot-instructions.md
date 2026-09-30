## wayfinder-app

> Wayfinder is a privacy-safe, modular parent-child reflection and parent emotional development app.

# AGENTS.md - Wayfinder App Operating Rules

## Mission

Wayfinder is a privacy-safe, modular parent-child reflection and parent emotional development app.

The journal is a mechanism, not the whole product. Wayfinder helps parents recognise where their Cognition, Affect, and Behaviour (CAB) may not yet be aligned with the emerging needs of the child, then guides them toward awareness, growth, repair, and next action.

The app must always remain deployable and working. Partners may improve visuals, content, UX, and functional features, but every change must preserve authentication, privacy, dashboard data loading, journal save/read behaviour, and existing user records.

## Instruction Priority

If instructions appear to conflict, use this priority order:

1. Preserve privacy, security, authentication, RLS, profile integrity, parent/child identity, journal save/read behaviour, and deployability.
2. Preserve the Wayfinder ALIGN/CAB product canon.
3. Improve UX, UI, content, and functional features within the above boundaries.

Do not silently resolve conflicts. If a requested change would weaken privacy, auth, RLS, identity, existing records, or the ALIGN/CAB philosophy, stop and report the conflict.

## Must Read Before Editing

Before making product, UX, UI, copy, or implementation changes, read:

- docs/WAYFINDER_ALIGN_PRODUCT_CANON.md
- docs/app-architecture-navigation-review.md
- docs/auth-profile-flow.md
- docs/partner-collaboration-and-deployment-rules.md
- docs/profile-cleanup.sql

If `docs/WAYFINDER_ALIGN_PRODUCT_CANON.md` is missing, stop and ask for it to be added before making product/UX/copy changes.

## Product Canon: ALIGN/CAB

Wayfinder is not a child-diagnosis app, behaviour-labelling app, or generic Behaviour -> Advice parenting app.

Wayfinder helps parents recognise where their Cognition, Affect, and Behaviour are not yet aligned with the child's emerging need.

Core pathway:

Behaviour
-> Need
-> Parent CAB
-> Alignment Check
-> Awareness
-> Growth
-> Navigate / Next Action

Official ALIGN mapping:

- A = Awareness
- L = Locate
- I = Integrate
- G = Growth
- N = Navigate

CAB means:

- Cognition: what the parent thinks, assumes, expects, or interprets.
- Affect: what the parent feels emotionally and in the body.
- Behaviour: what the parent does in response.

The phrase "My child did something and I don't know why" must never mean "the child is the problem."

It means:

"The child's behaviour may be revealing a moment where the parent's CAB is not aligned with the child's emerging need."

## Product Framing Rules

Required framing:

- Behaviour is a signal, not a diagnosis.
- The child is not framed as the problem.
- The parent is not blamed or shamed.
- The product guides the parent to notice the possible child need, identify their own CAB, locate misalignment, integrate insight, grow capacity, and navigate a next action or repair step.
- Use cautious language: "may", "might", "possible", "may have", "let's explore".

Do not write or generate language such as:

- "Your child is avoidant."
- "Your child is controlling."
- "Your child is the problem."
- "This behaviour means X" as a certainty.
- "Fix this behaviour by..." as the primary flow.

Prefer language such as:

- "This behaviour may have been trying to solve something."
- "A possible need underneath this behaviour is..."
- "The behaviour may be a signal of..."
- "Let's get curious about what need was emerging."
- "Where might your CAB have moved out of alignment with this need?"

## Parent Development Profile Rules

Do not create fixed parent types, parent diagnoses, or child diagnoses.

If building profile or dashboard insights, create a living developmental reflection profile based on:

- recurring CAB patterns
- recurring child needs noticed
- recurring misalignments
- current growth edge
- emerging strengths
- current practice
- repair habits
- next action patterns

Example acceptable insight:

"Current Alignment Challenge: urgency vs child need for predictability."

Example unacceptable insight:

"You are a controlling parent" or "Your child is oppositional."

## Decode a Moment Build Rules

The planned entry point is:

Title: Decode a Moment
Subtitle: "My child did something and I don't know why."
Description: Explore what the behaviour may have been signalling, what happened in you, and how to respond with more alignment.
Button: Start Decode

When implementing this feature, preserve this flow:

1. Awareness - name what happened without judging the child.
2. Locate - locate the possible child need and the parent's internal response.
3. Integrate - connect child need with parent Cognition, Affect, and Behaviour.
4. Growth - identify the parent capacity being developed.
5. Navigate - choose one next action or repair step.

Suggested saved entry type:

entry_type = "behaviour_decode"

Use existing journal storage in Phase 1 if safe. Do not change Supabase schema unless explicitly approved.

## Core Architecture Rules

Stable identity chain:

Supabase auth user -> profiles.parent_id / Wayfinder ID -> dyads.child_id -> journal_entries

Use:

- Supabase user_id only as technical auth identity.
- parent_id / Wayfinder ID as the app identity.
- child_id as child identity.
- Dashboard data should load by parent_id first.

## Auth/Profile Rules

Required:

1. User signs up/signs in with Supabase email/password.
2. Email must be verified before app access.
3. Profile creation/retrieval goes through ensure_profile.
4. ensure_profile is called via explicit /rest/v1/rpc/ensure_profile fetch.
5. Request must send Authorization: Bearer <access_token>.
6. Browser code must never directly insert/upsert into profiles.

Never add browser-side:

profiles.insert
profiles.upsert

## Dashboard Read Rules

Dashboard reads are protected by Supabase RLS.

For dyads and journal_entries:

1. Use the verified callback auth session where available.
2. Send explicit Authorization: Bearer <access_token> for protected reads.
3. Load by parent_id first.
4. Use user_id only as fallback.
5. Do not rely only on sb.auth.getSession(), because in this static app it may return no session after the auth callback.

Expected working reads:

DB.getAllDyads(user.id, profile.parent_id, authSession)
DB.getEntries(user.id, profile.parent_id, authSession)

## PDPA / Privacy Rules

Normal user-facing UI must not show:

- parent email
- Supabase UUID
- child names
- JWTs
- refresh tokens
- anon keys
- service keys
- raw auth identifiers

Normal user-facing UI may show:

- generated Parent ID / Wayfinder ID
- generated Child ID
- parent age
- parent gender
- child age
- child gender
- role
- non-identifying activity/reflection content
- non-identifying ALIGN/CAB reflection content

Debug UI may show technical IDs only when hidden behind:

localStorage.getItem('wayfinder_debug_auth') === '1'

Never show tokens or secrets, even in debug UI.

## Date/Time Rules

Any value written to Supabase timestamp/timestamptz columns must use ISO format:

new Date().toISOString()

Never write locale strings such as:

28/05/2026, 00:23:02

Locale formatting is allowed only for display.

## Dashboard Must Keep Working

After verified login, Dashboard must show:

- Parent ID / Wayfinder ID
- role
- parent age/gender if available
- Past activities
- DISC insight if available
- Your children
- Child ID
- child age/gender
- recent reflections
- My journal trail
- This week, lean into

If no records exist, show friendly empty states. Do not break the Dashboard just because one data fetch fails.

Future dashboard enhancements may add "Your ALIGN Journey" and "Decode a Moment", but these must not break existing Dashboard data loading.

## Modular Collaboration Rules

Low-risk edits:

- styles.css
- content.js
- docs
- UI-only layout sections in app.js

Medium-risk edits:

- dashboard widgets
- journal views
- activity history views
- read-only DB display methods
- behaviour decode display cards using existing journal storage
- ALIGN/CAB visual summaries using already-loaded journal data

High-risk edits requiring extra review:

- supabase.js
- SQL/RLS files
- auth/profile logic
- ensure_profile
- email verification gate
- Parent ID / Child ID generation
- data writes/deletes
- Vercel/Supabase config
- new database tables or schema changes
- changes to journal save/read payload compatibility

## Required Checks Before Commit

Run:

```bash
git diff --check
node --check supabase.js
```

On Windows PowerShell, also search:

```powershell
Select-String -Path .\*.js, .\api\*.js -SimpleMatch -Pattern "profiles.insert","profiles.upsert","ensure_profile","Authorization","Bearer"
```

Cross-platform alternative:

```bash
rg -n "profiles\.(insert|upsert)" app.js supabase.js api/*.js
rg -n "ensure_profile|Authorization|Bearer" supabase.js app.js api/*.js
```

Confirm:

- no browser-side profiles.insert
- no browser-side profiles.upsert
- ensure_profile still uses explicit Bearer fetch
- dashboard reads still use verified session / Bearer token
- normal UI does not show parent email, Supabase UUID, child names, or tokens
- ALIGN/CAB wording remains non-diagnostic and non-blaming
- behaviour decode remains Behaviour -> Need -> Parent CAB -> Alignment Check -> Awareness -> Growth -> Navigate / Next Action

If available, also run:

```powershell
pwsh -NoProfile -ExecutionPolicy Bypass -File ./scripts/verify-wayfinder.ps1
```

or the Windows PowerShell equivalent.

## Manual Smoke Test

Before deploying or after Vercel deploys:

1. Verified user can log in.
2. Dashboard appears.
3. Same Parent ID is reused.
4. Child ID appears if records exist.
5. Past activities appear if records exist.
6. Recent reflections appear.
7. Sign out works.
8. Unverified email is blocked.
9. No parent email appears in normal UI.
10. No Supabase UUID appears in normal UI.
11. Incognito and normal browser show the same dashboard records.
12. Journal entry can be saved and appears in Journal Trail.
13. If Decode a Moment exists, behaviour_decode can be saved and appears in Journal Trail without breaking activity journals.
14. Desktop and mobile layouts remain usable.

Useful console checks:

```js
await DB.getAllDyads("USER_ID", "PARENT_ID")
await DB.getEntries("USER_ID", "PARENT_ID")
```

Expected:

- dyads return existing child records
- entries return existing journal records

## Commit Rule

Do not commit if the app is knowingly broken.

If a check fails, stop and report:

- what failed
- suspected cause
- proposed fix

Before reporting completion, summarise:

- files changed
- product/ALIGN impact
- technical risk level
- checks run
- manual tests still needed

---
> Source: [rodtay72/wayfinder-app](https://github.com/rodtay72/wayfinder-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
