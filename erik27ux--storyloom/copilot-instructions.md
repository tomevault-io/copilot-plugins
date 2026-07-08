## storyloom

> These must be triggered manually at the right moments. Remind the user proactively.

# Storyloom — Claude Code Reference

## ⚠️ Security Checkpoints — DO NOT FORGET
These must be triggered manually at the right moments. Remind the user proactively.

1. **Before testing with real users / soft launch** → re-run full security review (`/security-review` or manual file audit). Confirm the Supabase SQL migration (`supabase_security_migration.sql`) has been run in production.
2. **Before adding RevenueCat / any payment system** → full security review with focus on: subscription tier write paths, webhook authentication, receipt validation, and ensuring `updateSubscriptionTier()` in AuthManager is replaced by a backend webhook (never called from client in production).
3. **Before App Store submission** → remove `#if DEBUG` dev tier overrides in `AuthManager.swift` (`devTierOverrides` dict), replace App Store placeholder ID `id000000000` in `join.html` and `index.html`, and run security review.
4. **After any Supabase schema change** → re-verify RLS policies cover the new tables/columns.
5. **When Apple Developer Program enrollment is complete ($99/year)** → upgrade push notifications from local to APNs:
   - Enable Push Notifications capability in Xcode → Signing & Capabilities (one checkbox)
   - Download an APNs Auth Key (.p8) from Apple Developer portal → Keys section
   - Run the Supabase DB migration to add `push_token text` column to `profiles` table (SQL in `NotificationManager.swift` comments)
   - Write a Supabase Edge Function that triggers on DB inserts (comments, questions, story publishes) and sends APNs pushes using the .p8 key
   - The `NotificationManager.uploadTokenToSupabase()` and all client-side code is already complete — no app changes needed beyond enabling the capability

---

## 🚀 Pre-Launch Checklist — Step-by-Step

### PHASE 1 — Apple Developer Program ($99/yr)
**When:** As soon as enrolled at developer.apple.com

**Xcode — one-time setup:**
- [ ] Add **Associated Domains** capability → `applinks:storyloom.live`
  - (AASA file already deployed at `storyloom.live/.well-known/apple-app-site-association` — no web change needed)
  - ⚠️ **Removed for simulator testing** — personal teams don't support this capability. Re-add it once enrolled in Apple Developer Program.
- [ ] Add **Push Notifications** capability (one checkbox in Signing & Capabilities)

**Apple Developer portal:**
- [ ] Download an **APNs Auth Key** (.p8) → Keys section of developer.apple.com
  - Save it somewhere safe — it cannot be re-downloaded

**Supabase — run this SQL migration:**
```sql
ALTER TABLE profiles ADD COLUMN push_token text;
```
  - (Full SQL in `NotificationManager.swift` comments)

**Supabase — write a new Edge Function:**
- Trigger: on INSERT into `story_entries`, `story_questions` (answer updates), or `comments`
- Action: call APNs using the .p8 key to send a push to the relevant user's `push_token`
- The client-side code (`NotificationManager.uploadTokenToSupabase()`) is already complete — no app changes needed

---

### PHASE 2 — RevenueCat Integration
**When:** Adding paid subscription tiers

**Security review first** — run `/security-review` with focus on:
- Subscription tier write paths
- Webhook authentication
- Receipt validation

**Files to update:**
- `AuthManager.swift` — find `updateSubscriptionTier()`:
  - Currently called from client-side (OK for dev/testing)
  - **Must be replaced by a RevenueCat webhook** before shipping — client must NEVER be able to write its own subscription tier in production
- `AuthManager.swift` — find `devTierOverrides` dict:
  - Already gated with `#if DEBUG` — ensure it stays that way until explicitly removed pre-launch

**RevenueCat setup steps:**
- [ ] Create RevenueCat project, link to App Store Connect
- [ ] Add RevenueCat SDK to Xcode project
- [ ] Configure entitlements/products in RevenueCat dashboard matching existing tier names (`free`, `family`)
- [ ] Write a Supabase Edge Function as the RevenueCat webhook endpoint (updates `profiles.subscription_tier` on purchase/renewal/cancellation)
- [ ] Verify webhook signature validation in the Edge Function
- [ ] Run Supabase RLS audit — confirm `subscription_tier` column is NOT writable by the user themselves (currently protected but verify after any schema changes)

---

### PHASE 3 — App Store Submission
**When:** Ready to submit

**Security review** — run `/security-review` (full pass, not just subscription paths)

**Files to update before submitting:**

1. `AuthManager.swift`
   - Remove the `devTierOverrides` dictionary entirely (or the whole `#if DEBUG` block that sets tier overrides)
   - Search for: `devTierOverrides`

2. `Storyloom web/join.html`
   - The "Coming soon to the App Store" badge needs to become a real App Store link
   - Replace the `.coming-soon` div with an `<a href="https://apps.apple.com/app/storyloom/id<REAL_ID>">` link
   - Also update the meta/OG tags if added later

3. `Storyloom web/index.html`
   - Same App Store link replacement as `join.html`

4. `Storyloom web/privacy.html`
   - Verify "Last updated" date is current
   - Add any new data processors introduced by RevenueCat

**App Store Connect setup:**
- [ ] Set Privacy Policy URL → `https://storyloom.live/privacy`
- [ ] Set Support URL → `https://storyloom.live` (or a contact page)
- [ ] Upload screenshots (all required device sizes)
- [ ] Write app description, keywords, subtitle
- [ ] Set age rating (likely 4+)
- [ ] Link to the correct RevenueCat in-app purchases

**TestFlight first:**
- [ ] Submit a TestFlight build with all `#if DEBUG` overrides removed
- [ ] Test invite flow end-to-end with a real share link (`storyloom.live/join/CODE`)
- [ ] Verify Universal Links open the app (not Safari) on a real device
- [ ] Verify push notifications fire correctly

---

### ALWAYS — After Any Supabase Schema Change
- Re-run RLS policy audit on any new tables or columns
- Check that new columns are not accidentally writable by the wrong role
- Confirm `supabase_security_migration.sql` is kept up to date

---

### Known Pending Verification
- **`FamilyView` column names** — `fetchReaders()` queries `story_access` using columns `user_id` and `date_granted`. Confirm these match the actual Supabase schema on first real-device test. If column names differ, update the query in `FamilyView.swift`.

---

### Current Website State (storyloom.live)
- `index.html` — "Coming soon to the App Store" badge + Privacy Policy footer link ✅
- `join.html` — "Coming soon" badge + invite code display + "Add Story Vault" hint ✅
- `privacy.html` — full privacy policy, live at `/privacy` ✅
- `.well-known/apple-app-site-association` — configured for `3BJ76TZ89X.erikfischer.Storyloom`, paths: `/join/*` ✅
- `vercel.json` — rewrites for `/join/:code` and `/privacy` ✅
- Deployment: `npx vercel --prod --token <cli-deploy token> --scope erik27uxs-projects`

---

## Project
Storyloom is an iOS app (SwiftUI + SwiftData + Supabase) for preserving and sharing family stories. Storytellers write, record narration, and attach images to stories; readers access them via invite codes.

**Stack:** SwiftUI · SwiftData · Supabase (auth, db, storage, realtime) · RevenueCat (planned) · Resend email · Vercel (storyloom.live)  
**Repo:** https://github.com/Erik27UX/StoryLoom · **Bundle ID:** erikfischer.Storyloom · **Team:** 3BJ76TZ89X

---

## 1. Workflow — GSD (Get Shit Done)
Reference: https://github.com/gsd-build/get-shit-done

Follow this loop for all non-trivial work:
1. **Discuss** → `/gsd-discuss-phase` — lock decisions before planning
2. **Plan** → `/gsd-plan-phase` — atomic tasks with verification conditions
3. **Execute** → `/gsd-execute-phase` — parallel where possible, one commit per task
4. **Verify** → `/gsd-verify-work` — test before shipping
5. **Ship** → `/gsd-ship`

**Quick fixes** (bugs, copy edits, small tweaks): use `/gsd-quick "task"` — no full phase machinery needed.  
**Start of session:** run `/gsd-progress` to see where we left off.  
**Lost context:** run `/gsd-resume-work`.

---

## 2. Coding Principles — Karpathy Guidelines
Reference: https://github.com/forrestchang/andrej-karpathy-skills

### Think Before Coding
Don't assume. Don't hide confusion. Surface tradeoffs.
- State assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### Simplicity First
Minimum code that solves the problem. Nothing speculative.
- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" that wasn't requested.
- If you write 200 lines and it could be 50, rewrite it.

### Surgical Changes
Touch only what you must. Clean up only your own mess.
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- Every changed line should trace directly to the user's request.

### Goal-Driven Execution
Define success criteria. Loop until verified.
- Transform tasks into verifiable goals before starting.
- For multi-step tasks, state a brief plan with verification checkpoints.

---

## 3. Swift Rules — Everything Claude Code
Reference: https://github.com/affaan-m/everything-claude-code (rules/swift/)

### Coding Style
- Prefer `let` over `var`; default to `struct`, use `class` only for reference semantics
- Follow Apple API Design Guidelines — clarity at point of use, name by role not type
- Enable Swift 6 strict concurrency checking; favour `Sendable` value types
- Use actors for shared mutable state instead of locks or dispatch queues
- Use Swift 6 typed throws with pattern matching for error handling

### Patterns
- Protocol-oriented design: focused protocols with extensions for shared defaults
- Value types for data models; enums with associated values for distinct states
- Actor pattern for shared mutable state (no manual synchronisation needed)
- Dependency injection: pass protocol-typed dependencies with default implementations

### Security
- Store tokens and passwords in **Keychain**, never `UserDefaults`
- Never hardcode secrets in source — decompilation tools extract them trivially
- Don't disable App Transport Security (ATS)
- Validate and sanitise all user input, deep link data, and clipboard content before processing
- Use `URL(string:)` with proper validation — never force-unwrap URL initialisation

---

## 4. UI/UX Principles
Reference: https://github.com/nextlevelbuilder/ui-ux-pro-max-skill

- Minimum text contrast **4.5:1** in light mode
- Smooth transitions **150–300ms** for state changes
- Touch targets sized for finger interaction (minimum 44×44pt)
- Micro-interactions for user feedback on actions
- Soft shadows and subtle depth for premium feel
- Avoid harsh animations, neon colours, or "AI gradient" aesthetics
- Design is industry-specific — Storyloom is warm, personal, family-oriented (cream/brown palette, serif-adjacent type)

---

## 5. Development Skills — Superpowers
Reference: https://github.com/obra/superpowers

Install in Claude Code: `/plugin install superpowers@claude-plugins-official`

Key workflow additions:
- **Test-first**: write failing test → watch it fail → write minimal code → pass → commit
- **YAGNI + DRY**: no speculative features, no premature abstraction
- Use `/plan` before executing complex changes
- Use `/code-review` after execution, before shipping

---

## 6. Memory — claude-mem
Reference: https://github.com/thedotmack/claude-mem  
**Status: Installed** — memory builds passively from each session.

- Worker must be running: `npx claude-mem start`
- Web viewer: http://localhost:37701
- Search past sessions: use the `mem-search` skill in any prompt
- Tag sensitive content with `<private>` to exclude from memory
- Memory injection starts from the second session onward
- To run a full codebase ingest: `/learn-codebase` (optional, ~5 min)

**Disabling temporarily:** set `CLAUDE_MEM_WELCOME_HINT_ENABLED=false` or stop the worker.

---

## How to disable any of these
To stop following a reference, comment it out or delete its section from this file. Takes effect next session. For claude-mem specifically: `npx claude-mem stop` stops the worker; to fully uninstall run `npx claude-mem uninstall`.

---
> Source: [Erik27UX/StoryLoom](https://github.com/Erik27UX/StoryLoom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
