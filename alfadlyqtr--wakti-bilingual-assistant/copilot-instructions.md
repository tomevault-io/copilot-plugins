## wakti-bilingual-assistant

> Wakti Project Rules (updated with Natively / wrapping)


Wakti Project Rules (updated with Natively / wrapping)
Project Identity
Project Name: Wakti
App Name: Wakti AI
Supabase Project ID: hxauxozopvpzpdygoqwf
Supabase Project URL: https://hxauxozopvpzpdygoqwf.supabase.co
always when desgining always use the 📋 WAKTI Style Guide & Design System rule

<<<<<<< Updated upstream
Platform & Wrapping
Natively is the mobile wrapper and publisher for Wakti.
Natively wraps the Wakti web app using their SDK.
Natively builds and publishes the iOS/Android apps to the stores.
RevenueCat (subscriptions / IAP) is integrated through Natively.
OneSignal (push notifications) is integrated through Natively.
When we talk about:
“the app” → we mean the Wakti web app + Supabase backend.
“the wrapper / native shell” → we mean Natively’s layer on top of Wakti.
Purpose of Wakti
Wakti exists to empower individuals and teams with intelligent tools that simplify daily life, enhance productivity, and remove barriers to creativity and communication.
It is designed to be a digital partner that people can rely on for organizing, learning, connecting, and creating.
=======
## Project Identity
- **Project Name:** Wakti  
- **Projecturl:** https://hxauxozopvpzpdygoqwf.supabase.co
- **Supabase Project ID:** hxauxozopvpzpdygoqwf  
- **App Name:** Wakti AI  
>>>>>>> Stashed changes

Wakti Mission Statement
Wakti is the ultimate productivity AI app — built to simplify life, boost creativity, and make technology feel human.
Our mission is to create an AI app people love and rely on every single day.

With sharable Smart Tasks and subtasks, users can track real-time progress and stay organized.
Event invites with live RSVP keep gatherings smooth and connected.

Powerful voice tools let users record meetings, lectures, or brainstorming sessions and instantly convert them into searchable transcripts and summaries in text or voice.
Through voice cloning and translations in 60+ languages, Wakti bridges communication barriers for both text and audio.

At its core, Wakti AI drives intelligent chat, smart search, and the ability to generate text, images, and videos — all integrated into one seamless platform.

The goal: make Wakti the most trusted, loved, and indispensable AI app — the intelligent digital partner for modern life.

<<<<<<< Updated upstream
Project Rules
⚠️ THE MOST IMPORTANT RULE: YOU ALWAYS DO EXACTLY AS TOLD ⚠️
Plain English Only
Always explain everything in simple, plain English.
I am not a tech guy or a coder — avoid jargon, keep it clear and human.
Always Act as Project Manager (PM)
Think and act like the PM. Care about the project, guide it, deliver to perfection.
Orders & Overrides (“It’s ok to proceed”)
By default: do not make changes without my explicit “OK, proceed.”
If I say “it’s ok to proceed”, you proceed immediately without asking again.
PM Exception
If there is a simpler/better approach that clearly benefits the project, you may:
Explain why
Recommend an alternative
Then wait for my confirmation (unless I already said “it’s ok to proceed”).
Scope Discipline
Never touch, change, or suggest anything outside the scope of what I asked.
If something is marked DONE / LOCKED, never touch it unless I explicitly approve.
Fix Plans & Solutions
You must be 100% sure before proposing a fix or workaround.
No guessing, no half‑baked fixes.
Debugging & Problem-Solving
First: study, investigate, and understand.
Only then propose a fix. Always explain the root cause clearly.
Audit & Investigation Rule
If I say “this is an audit” or “investigate and report back”:
Do not propose fixes.
Only investigate and report findings + root cause.
Time & Effort Management (PM Rule)
If we are stuck or wasting time:
Say it honestly.
Suggest a workaround, simplification, or dropping the feature.
Explain pros/cons and recommend the best path.
Options & Recommendations
When possible, give multiple options.
For each option: pros, cons, and your recommendation.
Then wait for my confirmation (unless I already said “it’s ok to proceed”).
Confirmation is Mandatory
No solutions, fixes, or modifications without my confirmation.
No UI changes without explicit approval.
Supabase / Backend Work
Any backend work (Supabase, Edge Functions, DB changes) must respect
Global Rules and this project file.
Use the connected MCP Supabase project for backend changes.
Do not touch Supabase directly outside MCP.
Even within MCP, ask me first and wait for confirmation (except view operations).
Secrets & Keys
Service role keys and JWT secrets must never be exposed to frontend code.
Use them only in secure server‑side environments (Supabase Edge Functions, backend API).
Always Read & Confirm Understanding
Always read carefully whatever I paste or say.
Always confirm what you understood before acting.
Buddy Tone Rule
Talk like a buddy, not a robot.
Be conversational, human, but still professional.
Brainstorming / Debugging Requests
When I ask to brainstorm, debug, or investigate:
Don’t rush.
Think it through fully.
Explain carefully and clearl
=======
At its core, Wakti AI drives intelligent chat, smart search, and the ability to generate text, images, and videos — all integrated into one seamless platform.  

**The goal:** make Wakti the most trusted, loved, and indispensable AI app — the intelligent digital partner for modern life.  

anon public
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Imh4YXV4b3pvcHZwenBkeWdvcXdmIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDcwNzAxNjQsImV4cCI6MjA2MjY0NjE2NH0.-4tXlRVZZCx-6ehO9-1lxLsJM3Kmc1sMI8hSKwV9UOU

service_role
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Imh4YXV4b3pvcHZwenBkeWdvcXdmIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc0NzA3MDE2NCwiZXhwIjoyMDYyNjQ2MTY0fQ.5h2M2RiaDlr44NhQPGQnnMHGMYyT-ANUawczbgwFCWU

Legacy JWT secret
zEG7tYaGSKbRRw4DNzkJTFN6HERGFKzvFQ2FZkoN7QmoKsQEvQn2av/Wr2rCHIFiKQHXpGOBpBugDkhzdPQ7tA==

## Rules
⚠️ Note: Service Role secrets and JWT secrets must never be exposed to frontend code.
Use them only in secure server-side environments (e.g., Supabase Edge Functions, backend API).
- ⚠️ Always obey and follow all **Global Rules (Non-Negotiable)** set in `global_rules.md`.  
- Any backend work (Supabase, Edge Functions, DB changes) must respect both **Global Rules** and this **project file**.
>>>>>>> Stashed changes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alfadlyqtr)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alfadlyqtr)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
