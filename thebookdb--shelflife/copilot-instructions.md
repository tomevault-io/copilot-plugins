## shelflife

> **Canonical product vision and data model:** see `docs/shelflife.md` — that document is the single source of truth.

# CLAUDE.md

**Canonical product vision and data model:** see `docs/shelflife.md` — that document is the single source of truth.

## Development Commands

- `foreman start -f Procfile.dev` - Start all services (server, esbuild, tailwind, queue)
- `bin/rails test` - Run all tests (Minitest)
- `bin/rails test test/models/product_test.rb` - Run a single test file
- `bin/rails test test/models/product_test.rb:42` - Run a single test by line number
- `bin/rails db:seed` - Seed lookup tables (run after any fresh schema load)
- `bin/standardrb` - Ruby linter (Standard style)
- `bundle exec brakeman` - Security analysis
- MCP Gitea available for repo `dkam/shelf-life`

### Fresh database rebuild
```
rm db/schema.rb && bin/rails db:drop db:create db:migrate db:seed
```
**Do not use `db:drop db:create db:migrate` without deleting schema.rb first** — Rails detects the file and calls `db:schema:load` instead of running migrations, leaving stale state.

## Tech Stack

Rails 8 · Phlex 2 views · Stimulus + Turbo · Tailwind 4 · esbuild · SQLite (separate DBs for cache/queue/cable via Solid adapters) · html5-qrcode

**Phlex 2**: Use `plain "text"` when mixing text with HTML elements inside a block.

### Phlex component layout
All components live under `app/components/` with namespace `Components::`. Page-level components follow the `*_view.rb` naming convention and are rendered directly from controllers:
```ruby
render Components::Libraries::ShowView.new(library: @library, ...)
```
Reusable sub-components drop the `_view` suffix. Base class is `Components::Base`, which includes route helpers, form helpers, and Turbo element registration.

### Auth
Custom session-based auth (no Devise). `Authentication` concern in `ApplicationController`. Session model tracks user_agent and IP. Current user/library available via `Current.user` and `Current.library` (via `Current < ActiveSupport::CurrentAttributes`).

### Real-time updates
`ProductDataFetchJob` broadcasts product enrichment results via Turbo Streams on channel `"product_#{id}"`. Library changes broadcast on `"library_#{id}"`. Jobs use queue `:tbdb_api` with exponential backoff on rate limits/quota exhaustion.

## Product Vision

ShelfLife is a personal/small-group library app for cataloguing collections — books, board games, DVDs, whisky, vinyl, etc. Users scan barcodes to add items. Product data comes from TBDB (TheBookDB) API.

**The two core actions:**
1. **Scan something you have** → goes into your library, intent: `have`
2. **Scan something you want** → goes into your library, intent: `want`

Libraries are containers (like shelves). The have/want distinction lives on the item, not the library.

**Not yet:** social features, recommendations, marketplace, lending. Keep it simple.

**TBDB relationship:** ShelfLife consumes TBDB data (read-only). Product corrections will eventually flow back upstream. ShelfLife links to Booko for pricing.

## Routing

- `/` → Products index (recent additions dashboard)
- `/scanner` → Barcode scanning interface
- `/:ean13` → Product show (13-digit constraint)
- `/libraries`, `/library_items` → Collection management

## Data Models

### Product
Bibliographic data for a published work. Identified by `gtin` (EAN-13). Enriched from TBDB API (`tbdb_data` JSON, `product_type`, `title`, `author`, etc.). Auto-enrichment via `ProductDataFetchJob`. Products are shared across all users — one record per barcode.

### Library
A container for organising items. Think "shelves" — Home, Work, Mum's House.
Belongs to User (optional). Fields: `name`, `description`.
Has `visibility` enum: `ours` (0, private default), `anyone` (1, shareable via link), `everyone` (2, public). No UI for visibility yet.

### LibraryItem
Join between Library and Product — "this user has this product in this library". Key fields:
- `intent` (integer enum): `have` (0, default), `want` (1)
- Tracking: `added_by` / `updated_by` (User FKs)
- Condition: `condition` (FK), `condition_notes`, `last_condition_check`
- Acquisition: `acquisition_date`, `acquisition_source` (FK), `acquisition_price`, `copy_identifier`
- Status: `item_status` (FK) — seeded: Available, Checked Out, Missing, Damaged, In Repair, Retired
- Ownership: `ownership_status` (FK) — seeded: Owned, Borrowed, On Loan, Consignment
- Value: `replacement_cost`, `original_retail_price`, `current_market_value`
- Misc: `location`, `tags`, `is_favorite`, `notes`, `private_notes`

### Lookup tables (seeded)
`Condition`, `ItemStatus`, `OwnershipStatus`, `AcquisitionSource` — all populated by `db:seed`.

## Scanner Flow

1. User selects a library from the dropdown
2. Scanner POSTs `{ gtin:, library_id:, intent: }` to `/library_items`
3. Controller creates LibraryItem in the selected library with have/want intent
4. `set_library` session endpoint stores selection between scans

### Scanner UI (confirmed working portrait + landscape, iPhone 15 Pro)
- Single orientation-responsive layout using CSS `landscape:` variant — portrait stacks vertically, landscape splits 2/3 camera left / 1/3 controls right
- One Stimulus controller (`barcode_scanner_controller.js`), one scanner element — no dual portrait/landscape DOM
- Camera lens selector: slide-out drawer from left edge, uses `navigator.mediaDevices.enumerateDevices()` (not `Html5Qrcode.getCameras()` which hijacks the active stream on iOS)
- Active camera label shown via `recentItemTargetConnected` Stimulus callback
- Scan results prepend into the recent list via Turbo Stream with green highlight that fades after 2s
- 2-second per-barcode cooldown prevents duplicate scans
- Retired views (`adaptive_view.rb`, `horizontal_view.rb`) and controllers still exist in codebase but are unused

## TBDB Integration

Single shared `TbdbConnection` singleton for OAuth (not per-user). `Tbdb::Client` makes authenticated requests (lazy validation). `Tbdb::OauthService` handles OAuth lifecycle.

## Visual Language

Two distinct color palettes communicate whether you're looking at a **Product** (a thing that exists in the world) or a **Library Item** (your copy of that thing):

- **Owned (have)** — Blood orange (`orange-600`/`orange-100`). Personal, warm. Header: "Your Copy". Border accent: `border-l-4 border-orange-500`.
- **Not owned** — Slate blue (`slate-600`/`slate-100`). Neutral, informational. Covers: products not in any library, search results, and `want` (wishlist) items. Header: "About This Product" for products, "On Your Wishlist" for want items. Border accent: `border-l-4 border-slate-400`.

The colour follows **ownership**, not whether something is a library item. A `want` item is still a LibraryItem (with its own notes, tags, etc.) but gets slate blue because you don't have it yet.

These manifest as:
- A left-edge border on cards and detail views (bottom-edge on cover-art thumbnail cards)
- Coloured date spine on library item rows in library views
- Consistent header language ("Your Copy" vs "On Your Wishlist" vs "About This Product")
- Breadcrumbs: Product pages show `Products > Title`, Library item pages show `Library Name > Title`

**Navigation rule:** Clicking an item that is in a library (have or want) navigates to the **Library Item** detail view. Clicking a product not in any library navigates to the **Product** view.

## important-instruction-reminders
Do what has been asked; nothing more, nothing less.
NEVER create files unless they're absolutely necessary for achieving your goal.
ALWAYS prefer editing an existing file to creating a new one.
NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thebookdb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
