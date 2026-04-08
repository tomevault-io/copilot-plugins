## propria

> **Propria** (Latin: "one's own") — the operating system for Community Land Trusts.

# Propria — Lovable Handoff Guide

## What This Is
**Propria** (Latin: "one's own") — the operating system for Community Land Trusts.
Tagline: "Properly yours." Domain: **propria.app**

Powered by **CROS™** (Compass + Relational Operating System) and **NRI** (Natural Relational Intelligence) — the same ambient intelligence architecture that powers The Schola, adapted for the CLT world.

## Stack
- **Frontend**: React + TypeScript + Vite (this repo)
- **Backend** (to build in Lovable): Supabase + Edge Functions
- **AI**: Claude via Lovable AI Gateway (same pattern as The Schola)
- **Payments**: Stripe Connect
- **Email**: Gmail API (emails come from staff's own Gmail)

## Brand Identity

### Fonts
- **Display**: Playfair Display (serif) — headings, large text
- **Body**: Source Serif 4 — body copy
- **UI**: DM Sans — buttons, labels, nav, forms

### Colors
```css
:root {
  --forest: #1B3A2D;      /* Primary — sidebar, headers */
  --forest-mid: #2C5A45;
  --forest-light: #3D7A5F;
  --terra: #B85C38;        /* Accent — CTAs, "Propria." dot */
  --terra-light: #D4784F;
  --gold: #C4963A;         /* Secondary accent — NRI, compass */
  --parchment: #F5F0E8;    /* Light background */
  --cream: #FAF7F2;        /* Screen background */
  --ink: #1A1A14;          /* Primary text */
}
```

### Sacred Rules
- "Propria." always ends with a period in terra/orange
- Forest green sidebar is sacred — never change it
- Cards: `border: 0.5px solid var(--border)`, `border-radius: 12px`
- Buttons: `border-radius: 8px`
- NRI elements use `--gold` for dots, labels, accents

## CROS™ Architecture

### The Four Directions
| Direction | Latin | Domain | Color |
|-----------|-------|--------|-------|
| **Itiner** | Journey | Pipeline/applicant advancement | Blue |
| **Custodia** | Stewardship | Active homeowner care | Green |
| **Cura** | Care | Relational warmth | Gold |
| **Reconciliatio** | Reconciliation | Re-engagement | Coral |

### Signal → Direction Map
See `/src/cros/signalMap.ts` — 32 signal types mapped to directions.

### CROS Hooks (already built, in `/src/cros/`)
| Hook | What It Does |
|------|-------------|
| `useCompassPosture` | Infers current direction from signals + route fallback |
| `useCompassSessionEngine` | 8 parallel evaluations → max 3 nudges sorted by weight × confidence |
| `useCompassGlow` | FAB pulses gold on new signals (2min duration, 3min cooldown) |
| `useCompassAutoOpen` | Opens compass drawer once per session after 2.5s |

### For Lovable: Replacing Demo Data with Supabase
The hooks currently use in-memory arrays (`eventStream.ts`). Replace with:
```typescript
// eventStream.ts → supabase
supabase.from('propria_signals').select('*').gte('detected_at', cutoff)
supabase.from('app_event_stream').insert({ ... })
supabase.from('compass_user_state').select('*').eq('user_id', userId)
```
Hook signatures and logic stay identical.

## Data Model

### Demo Data Files (`/data/`)
10 JSON files ready to seed Supabase:
- `homeowners.json` — 5 records with full profile data
- `applicants.json` — 4 records with pipeline stages, documents, education
- `properties.json` — 5 properties with maintenance, inspection data
- `clts.json` — Rondo CLT org record (the tenant)
- `counselors.json` — 6 counselors with certs, languages, availability
- `education_modules.json` — 6 modules with lesson counts, thresholds
- `signal_kind_map.json` — 16 signal types → CROS directions
- `nudge_templates.json` — 8 nudge message patterns
- `users.json` — 4 personas (staff, homeowner, applicant, counselor)
- `board_members.json` — 4 board members with roles and terms

### Core Tables (from handoff doc)
```sql
clts (id, name, city, state, formula_type, ground_lease_fee, home_count)
users (id, clt_id, name, email, role, avatar_initials)
homeowners (id, clt_id, user_id, address, purchase_price, equity_built, nri_direction, attention_score)
applicants (id, clt_id, name, stage, education_progress, assigned_counselor_id, nri_flag)
properties (id, clt_id, homeowner_id, address, status, maintenance_requests)
counselors (id, name, org, certifications, languages, availability)
sessions (id, counselor_id, applicant_id, type, format, readai_summary)
education_progress (id, user_id, module_id, quiz_score, cert_issued_at)
propria_signals (id, clt_id, user_id, signal_type, direction, detected_at)
compass_user_state (user_id, dismissed_nudge_ids, last_auto_open_date, guide_sections_seen)
app_event_stream (id, user_id, event_type, event_data, route, created_at)
```

## Views / Routes

### Persona-Level Routes
| Route | Persona | View |
|-------|---------|------|
| `/` | Public | Marketing site |
| `/#/app` | CLT Staff (Sarah Chen) | Full CLT App — 13 screens |
| `/#/homeowner` | Homeowner (Maria Torres) | 5-screen portal |
| `/#/applicant` | Applicant (Keisha Johnson) | Pipeline tracker |
| `/#/gardener` | Network admin | 5-zone Gardener Console |
| `/#/education` | Applicant | 6-module course |
| `/#/certification` | Counselor | 4-module cert |
| `/#/counselor` | Counselor (Sarah Wilkins) | Session dashboard |
| `/#/directory` | Public | Counselor listing |

### CLT App Screens (inside `/#/app`)
Dashboard, Stewardship (4 tabs), Pipeline, Praeco (3 tabs), Resale, Assets, Maintenance, Governance (4 tabs), Finances (2 tabs incl. HUD-9902), Settings, Account, Magic Import, Ground Lease Generator

### Key Features
- **Magic Import**: 8-step AI wizard (Upload→Scan→Map→Dedup→Validate→Review→Import→Done)
- **HUD-9902 Generator**: Auto-populated quarterly report with field validation
- **Site Builder**: Browser-chrome preview + NRI content assistant + block editor
- **Resale Calculator**: Interactive formula calculator (3 formula types)
- **NRI Compass**: CROS-powered ambient intelligence drawer

## Demo Personas
| Name | Role | Key Data |
|------|------|----------|
| Sarah Chen | CLT coordinator, Rondo CLT | 47 families, 94% lease collected |
| Maria Torres | Homeowner, 14 Oak St. | 6.2 yrs owned, $14.2k equity |
| Keisha Johnson | Applicant, Stage 4 | 60% education, docs expired |
| Sarah Wilkins | HUD counselor, LSS Financial | 5 active clients |

## Architecture Decisions
1. **CLT as tenant** — multi-tenant, each CLT has isolated data
2. **CROS as ambient layer** — FAB in authenticated layout, not a separate page
3. **Signals, not polling** — emit signals when things happen, score them on drawer open
4. **Always confirm before executing** — NRI suggests, human decides
5. **Role-scoped everything** — queries filtered by role at source, never post-filtered
6. **Silent failure always** — event logging never blocks user operations
7. **One auto-open per session** — compass never auto-opens more than once

## Patterns from The Schola to Replicate
1. **Lovable AI Gateway** for all Claude calls
2. **Deterministic action engine** (no hallucination) for nudges
3. **Automation gating** with cooldowns for scheduled tasks
4. **Feature flags** per tenant from `tenant_feature_flags` table
5. **Real-time subscriptions** for membership/profile changes
6. **Human-centric terminology** — "stewardship" not "management", "families" not "accounts"

## Running Locally
```bash
npm install
npm run dev
```
Deployed to GitHub Pages on push to `claude/rebuild-design-project-ZCd5o`.
Live at: https://jeromydarling.github.io/propria/

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jeromydarling)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jeromydarling)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
