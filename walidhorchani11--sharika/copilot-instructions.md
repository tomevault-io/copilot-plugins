## project-learnings

> - When a phone number is entered, check for existing orders

# Project Learnings

## Critical Implementation Paths

### 1. Order Form Auto-fill Pattern
- When a phone number is entered, check for existing orders
- Auto-fill customer information (name, alt phone, address, governorate, delegation)
- Clear auto-filled data when phone number changes or becomes invalid
- Display customer statistics (active orders, total orders, last order details)
- Use `lastAutoFilledPhone` state to manage clearing logic

### 2. Real-time Validation Pattern
- Use `fieldErrors` state object to track validation errors per field
- `validateField` function provides immediate feedback as user types
- Display errors with visual indicators (red border, error message)
- Validate on blur and on change for better UX

### 3. Excel/CSV Import Pattern
- Always use `xlsx` library for parsing Excel files
- For CSV files, implement custom parser to handle quoted values correctly
- Always include preview step before applying updates
- Use multi-criteria search when exact match fails (tracking_id → phone+name → phone only)
- Implement scoring system for ambiguous matches (price, date, tracking ID, company)
- Always allow manual selection/validation before applying updates
- Display comprehensive statistics and details in preview tables

### 4. Payment Resolution Pattern
- When payment not found by tracking_id, search by customer name
- Use multi-criteria scoring system:
  - Tracking ID missing (+20 points)
  - Tracking ID different (+15 points)
  - Price matching exactly (+10) or close (+5)
  - Date matching exactly (+10) or close (+5)
  - Already Intigo (+5)
- Minimum score of 10 required for automatic resolution
- Always show preview before applying updates
- Update tracking_id and delivery company to Intigo if needed

### 5. Server-Side Pagination Pattern
- Always use server-side pagination for large lists (100+ items)
- Implement debouncing for search terms (300ms delay)
- Use `range()` for pagination in Supabase queries
- Use `count: 'exact'` to get total count
- Pass pagination metadata (page, limit, totalCount, totalPages) to client

### 6. Search Pattern
- Use Supabase `or()` with `ilike` for case-insensitive partial matching
- Support multiple fields: `customer_name.ilike.%term%,customer_phone.ilike.%term%,order_number.ilike.%term%,delivery_tracking_id.ilike.%term%`
- Always update placeholder to indicate all searchable fields

## User Preferences & Workflow

### Framework Version
- **ALWAYS use Next.js 16** - Never use older versions
- Use Next.js 16 App Router features (Server Components, Client Components, useSearchParams)
- User explicitly requested this preference

### Package Manager
- **ALWAYS use `pnpm`** - Never use `npm` or `yarn`
- User explicitly requested this preference

### Build & Testing
- Always run `pnpm run build` after significant changes
- Fix linter errors immediately
- Test build before considering task complete

### Code Style
- Use TypeScript strictly
- Prefer functional components with hooks
- Use Tailwind CSS for styling
- Mobile-first responsive design
- **Accessibility contrast rule (user-requested): never use near-white text colors on light backgrounds**
  - Forbidden text color range on light surfaces: `#EFEFEF` to `#FFFFFF` (includes `#F1F1F1`)
  - Minimum readable text tone on light backgrounds: `#374151` or darker
  - If a utility class maps to very light text (`text-gray-100/200/300`), override to readable contrast

## Project-Specific Patterns

### Product Price Calculation
- Prices stored in `price_config` JSONB field
- Structure varies by product:
  - Simple: `{ "Montre": 50 }`
  - Nested: `{ "argile": { "4": 160, "6": 230 } }`
  - Size-based: `{ "grand": 92, "petit": 65 }`
- Always check product name first, then extract specific fields
- Use `useEffect` to recalculate when product or specific fields change

### Delivery Provider Integration
- All providers implement `DeliveryProvider` interface
- Use Strategy Pattern for different APIs
- Always update `delivery_tracking_id` after successful shipment
- Cosmos/X Delivery: Don't change status on shipment (only on scan)
- Intigo: Change status to `en_livraison` on shipment
- Always create proxy routes for PDF downloads (security)

### Status Hierarchy
- Implement status levels to prevent downgrades:
  - nouvelle: 1
  - a_confirmer: 2
  - en_preparation: 3
  - prete: 4
  - en_pickup: 5
  - en_depot: 6
  - en_livraison: 7
  - livree: 8
  - retour: 9
  - retour_recu: 10
  - annulee: 0 (can always be set)

### Status Transition Control
- **CRITICAL**: Some statuses can ONLY be set through specific actions:
  - `en_pickup`: ONLY via adding order to a pickup (scan or selection)
  - `en_depot`: ONLY via Excel synchronization with delivery companies
  - Manual changes to these statuses are BLOCKED server-side
- `prete` can transition to `en_livraison` (direct delivery by driver)
- Always validate status transitions server-side in `/api/orders/[id]/status`
- Remove manual transitions from `StatusStepper.tsx` for controlled statuses

### Excel Export Pattern
- Always export only filtered results
- Include comprehensive columns (22+ columns)
- Use `xlsx` library with proper column widths
- Generate filename with date and result count
- Format dates and prices appropriately

### Dashboard Widget Pattern
- Create reusable widget components for tracking different order categories
- Use Recharts (`recharts`) for data visualization (PieChart, DonutChart)
- Widget structure:
  - Title with icon/emoji
  - Clickable counters linking to filtered lists
  - Donut chart showing distribution
  - Success message when count is 0
  - Visual alert (ring) when count > 0
- Always use URL parameters for filtering (`?status=...&payment=...`)
- Support combined filters in `OrderList.tsx` using `useSearchParams` (Next.js 16)
- Initialize filter state from URL parameters
- Synchronize filter state with URL changes via `useEffect`
- Widget categories:
  - Preparation: `nouvelle` + `en_preparation`
  - Transit: `en_pickup` + `en_depot` + `en_livraison`
  - Payment: `livree` + `payment_status = 'non_paye'`
  - Returns: `retour`

## Known Challenges

### 1. Hydration Errors
- Never nest `<a>` tags inside `Next/Link` components
- Use `<button>` with `window.location.href` for phone links inside Links
- Use `e.stopPropagation()` to prevent parent Link navigation

### 2. Turbopack Build Issues
- Avoid using `continue` in for loops with await (use if/else instead)
- Use `JSON.stringify()` for generating TypeScript files to avoid quote issues
- Always use double quotes in generated TypeScript files

### 3. TypeScript Array Access
- Supabase relations return arrays, not single objects
- Cast to `any` when accessing nested properties: `(order.delivery_company as any)?.name`
- Or use array access: `order.delivery_company[0]?.name`

### 4. CSV Parsing
- Implement custom parser to handle quoted values correctly
- Handle multi-line values in CSV
- Always trim and clean values after parsing

## Evolution of Project Decisions

### LocalStorage Draft Saving
- Initially implemented automatic draft saving every 2 seconds
- Changed to automatic restoration without modal
- **FINAL DECISION**: Completely removed as per user request
- User prefers no automatic saving

### Animation Preferences
- User requested removal of pulse animations
- User requested removal of channel badge
- Keep animations subtle and purposeful

### Statistics Display
- Total amount statistic hidden (admin-only data)
- User requested to either set to 0 or hide it

### Performance Optimization
- Moved from client-side filtering to server-side pagination
- Implemented debouncing for search
- Removed initial data fetching from server components

## Tool Usage Patterns

### File Operations
- Always use absolute paths when possible
- Prefer `read_file` over `grep` for understanding context
- Use `codebase_search` for semantic understanding
- Use `grep` for exact string/pattern matching

### Build & Linting
- Always run `read_lints` after editing files
- Run `pnpm run build` before considering task complete
- Fix TypeScript errors immediately

### Memory Bank Updates
- Update when user explicitly requests "update memory bank"
- Focus on `activeContext.md` and `progress.md` for current state
- Review ALL files during memory bank updates
- Document new patterns in `systemPatterns.md`
- Always document new widgets, components, and features in Recent Changes### Pickup Order Validation
- **CRITICAL**: Never allow cancelled orders (`order_status = 'annulee'`) to be added to pickups
- Validate server-side in all pickup endpoints:
  - `/api/pickups/[id]/add-orders`
  - `/api/pickups/[id]/scan`
  - `/api/pickups/[id]/import-excel`
- Filter cancelled orders client-side in pickup detail page
- Return clear error messages when attempting to add cancelled orders

---
> Source: [walidhorchani11/sharika](https://github.com/walidhorchani11/sharika) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
