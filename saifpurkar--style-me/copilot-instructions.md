## style-me

> - Next.js (App Router) + TypeScript + Tailwind CSS

# Style Me — Project Context

## Stack
- Next.js (App Router) + TypeScript + Tailwind CSS
- Supabase: Auth (magic link), Postgres DB, Storage (private `wardrobe` bucket, RLS scoped to user_id)
- Deployed on Vercel

## Existing tables
- profiles
- items
- outfits
- outfit_items

## Design system — follow exactly, ask before deviating
**Colors**
- `ink` #211F1C — primary text
- `paper` #F6F3EC — card/surface background
- `charcoal` #33302B — app background
- `denim` #33475B — primary accent (required-slot controls, selects, active states)
- `brass` #B08D57 — secondary accent (optional-slot controls, primary CTA buttons)

**Type**
- Display (headers only): Fraunces, semi-bold
- Body: Inter
- Utility (tags, labels, category names): IBM Plex Mono, uppercase, letter-spaced

**Layout**
- Sharp to near-sharp corners (2px radius max)
- Thin 1–1.5px rules / dashed borders instead of heavy shadows or big rounded cards
- Outfit screen uses a flat-lay arrangement (see below), not a generic stacked list

## Data model conventions
`items` table — columns to add/confirm:
- `category`: enum('top','bottom','footwear','accessory')
- `subcategory`: text, nullable — only used when category='accessory'. Values: 'headgear','eyewear','watch','bracelet','belt','bag','socks','other'
- `occasions`: text[] — allowed values: work, wfh, dining, date, casual_hangout, special_event, travel, outdoor, gym
- `formality`: text[] — allowed values: casual, smart_casual, business_casual, formal

`outfit_items` table: `outfit_id`, `item_id`, `slot` — slot is one of: headgear, eyewear, watch, top, bracelet, belt, bottom, footwear, bag

## Outfit screen — flat-lay layout order (top to bottom)
```
Headgear
Eyewear
(Watch)   Top   (Bracelet)
Belt
Bottom
(Footwear)   (Bag)
```
- Required slots (top, bottom, footwear): Swap button only, no dismiss.
- Optional/accessory slots (headgear, eyewear, watch, bracelet, belt, bag): Swap + dismiss (×).
- "Swap" on any slot offers two modes: Auto-Swap (cycles to next match automatically) or Choose from Closet (opens Closet screen filtered to that category/subcategory as a picker).
- A separate "Swap Entire Outfit" action regenerates all slots at once.
- If a *required* slot has no matching item, show which slot is missing rather than a blank or broken state. Accessory slots with no match are simply omitted, no message needed.

## Add Item flow
- Offer both "Take Photo" and "Upload from Library" — not one or the other.
- Tag fields (category, subcategory, occasion, formality) should eventually be AI-suggested and pre-filled, editable before save. Manual-only is fine until the AI tagging step is built.

## Working style
- Work in small, sequential steps. State the plan and which file(s) you'll touch before editing.
- Don't modify working features (auth, existing closet upload, navigation) unless the current task requires it.
- If something in this file conflicts with what I ask for in a session, point it out before proceeding rather than silently picking one.

---
> Source: [saifpurkar/style-me](https://github.com/saifpurkar/style-me) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
