## hobby-connect

> Act as a Senior Software Engineer building a high-grade university system. Prioritize clean layered architecture, secure data flow, and modularity.

# Developer Instructions for Cursor

## Role
Act as a Senior Software Engineer building a high-grade university system. Prioritize clean layered architecture, secure data flow, and modularity.

## Tech & Architecture Rules
- Use Next.js App Router and Supabase SSR (Server-Side Rendering) auth packages.
- Use Supabase Postgres for the database.
- Create modular React components (e.g., separate files for ChatTab, EventTab, Sidebar).

## UI & Theme Rules
- **Theme Enforcement:** Exclusively use the defined Purple and White color palette. 
- **Dark Mode:** Use Tailwind's `dark:` modifier on every component to ensure full Dark Mode support. Backgrounds should be Deep Charcoal, not pure black.
- **Layout:** Group layouts MUST use a Tabbed interface to separate chat from official events. 

## Business Logic & Security Rules (Strict)
1. **Email Validation:** The registration form MUST strictly validate that the email ends exactly with `@rsu.ac.th` on both the client and server. Block any other domains.
2. **Password Validation:** Ensure 'Password' and 'Confirm Password' match before submitting to Supabase.
3. **Frontend Role-Based Access (RBAC):** Use conditional rendering in the Events Tab. If `user.role !== 'moderator'`, completely hide the "Create Event", "Edit", and "Delete" buttons.
4. **Backend Security (RLS):** Implement Postgres Row Level Security (RLS) policies. 
   - `messages` table: `INSERT` allowed for any authenticated user in the group.
   - `events` table: `INSERT`, `UPDATE`, `DELETE` allowed strictly for users where `role === 'moderator'`. 
5. **Conflict Checking:** Before saving a new event, query the database to ensure the "Campus Location" and "Date/Time" do not overlap with an existing approved event.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kelvinakp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
