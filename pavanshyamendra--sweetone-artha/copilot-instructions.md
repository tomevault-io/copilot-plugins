## sweetone-artha

> Electron desktop app for **branch billing staff** at Sweetone. Handles POS billing, stock management, distribution acceptance, and bulk order viewing. All data syncs to/from **sweetone-oms**, which is the single backend.

# sweetone-artha

Electron desktop app for **branch billing staff** at Sweetone. Handles POS billing, stock management, distribution acceptance, and bulk order viewing. All data syncs to/from **sweetone-oms**, which is the single backend.

This app is distinct from **sweetone-retail** (which handles distribution *creation* and dispatch). Artha handles the *receiving* side at each branch.

## Related Projects

**sweetone-oms** lives at `~/Documents/sweetone-oms` and hosts the entire API.
- Deployed at `https://oms.sweetone.in`
- Next.js App Router with Prisma + PostgreSQL
- **When changing any API call in this project, also update the corresponding route handler in sweetone-oms (`src/app/api/...`).**
- **When adding a new field to a Bill, StockLive entry, or Distribution, update the Prisma schema in sweetone-oms and the TypeScript types in both projects.**

**sweetone-retail** lives at `~/Documents/sweetone-retail` — handles distribution *creation* and dispatch (the sending side). Artha handles *acceptance*.

## API Integration

Base URL: `src/electron/config.ts` → `BACKEND_URL = "https://oms.sweetone.in"`

All requests use `Authorization: Bearer <accessToken>` from `storageService.getToken()`.

### Auth — `src/electron/services/auth.service.ts`
| Function | OMS route | Notes |
|---|---|---|
| `getLogin(email, password)` | `POST /api/auth/login` | Returns `accessToken`, `refreshToken`, user info; stored in electron-store |
| `refreshToken()` | `POST /api/auth/refresh` | Sends `refreshToken` as `Cookie` header |

User fields stored in electron-store after login: `authToken`, `refreshToken`, `userId`, `userEmail`, `userName`, `userRole`, `userBranchId`, `userBranchName`.

### Billing — `src/electron/services/billing.service.ts`
| IPC channel | Method + OMS route | Notes |
|---|---|---|
| `save-order` | Local SQLite only (then auto-sync) | Saves bill locally, prints immediately; sync happens separately |
| `sync-orders` | `POST /api/bill/sync` | Body: `{ orders: unsyncedOrders }`. Returns `{ syncedIds }`. Marks synced in local DB. |
| `stocklive-sync` | `GET /api/stocklive` | Returns live stock levels for the branch |
| `update-stock` | `POST /api/bill/stocklive` | Body: `{ branchId, menuItemId, quantity, type: "add"\|"deduct"\|"set" }` |
| `print-stocklive` | Local print only | No API call; generates PDF via `createStockLivePDF` |

### Menu & Stock — `src/electron/services/menu.service.ts`
| IPC channel / Function | Method + OMS route | Notes |
|---|---|---|
| `get-menu-items` / `fetchMenuItems()` | `GET /api/menu` | All menu items; cached in electron-store |
| `sync-data-menu` / `syncDataMenu()` | `GET /api/branch` + `GET /api/menu/:branchId` | Parallel; also calls `syncOrdersDb()` first. Branch-specific pricing. |
| `transfer-stock` / `transferStock()` | `POST /api/stocklive/transfer` | Body: `{ fromBranchId, toBranchId, menuItemId, quantity }`. Prints transfer PDF on success. |
| `return-stock` / `returnStock()` | `POST /api/stocklive/returnStock` | Body: `{ branchId, menuItemId, quantity }`. Prints return PDF on success. |
| `get-menu-item-branchwise` | Local electron-store cache | Returns cached `menuItemsBranchwise_<branchId>` |

### Distributions — `src/electron/ipc/distribution.handlers.ts`
| IPC channel | Method + OMS route | Notes |
|---|---|---|
| `get-distributions` | `GET /api/distribute?date=` | Lists distributions for a given date |
| `get-distribution` | `GET /api/distribute/:id` | Single distribution |
| `get-current-distribution` | `GET /api/distribute/:id` | Same route, used from different UI flow |
| `get-branches` | `GET /api/branch` | Cached in electron-store |
| `create-distribution` | `POST /api/distribute` | Used for creating (less common from artha) |
| `put-distribution` | `PUT /api/distribute` | Body: updated distribution data |
| `accept-distribution` | `POST /api/distribute/accept` | Body: `{ distributionId, branchId, acceptedByUserId?, items: [{ distributionItemId, receivedQuantity?, receivedTrayWeight?, missingQuantity?, note? }] }` |

### Bulk Orders — `src/electron/services/bulkOrders.service.ts`
| IPC channel | Method + OMS route | Notes |
|---|---|---|
| `get-orders` | `GET /api/orders?date=&status=` | View-only; branch staff monitors incoming orders |

### Realtime Online-Order Printing (Pusher)
Incoming online orders (Rista webhook → OMS) are pushed to the branch app and auto-printed, instead of polling. Vercel serverless can't hold WebSockets, so Pusher sits between OMS and artha.
- **OMS**: `src/app/api/rista/webhooks/route.ts` → on `order.status = "Created"`, resolves branchCode→branchId via `resolveBranchIdFromCode` (`src/lib/rista/branch-map.ts`, reverse of the `rista_branch_map` setting) and calls `triggerBranchEvent` (`src/lib/pusher/server.ts`) with the full Rista sale detail on channel `branch-<branchId>`, event `rista-order.created`.
- **Artha**: `src/electron/services/ristaPrint.service.ts` holds one persistent Pusher connection, subscribes to `branch-<userBranchId>`, and prints via `createRistaOrderTicketPDF` (`src/electron/utils/80mmprint.ts`) on the `80 mm Series Printer`. Started in `main/index.ts` and re-subscribed after login (`auth.handlers.ts`). Note: `pusher-js`'s Node build exposes the constructor as the `.Pusher` member (its `.d.ts` claims a default export that is `undefined` at runtime), so it's required, not default-imported.
- **On-screen alert**: the same handler also fires `notifyNewOrder` → sends `online-order-received` IPC to the renderer (`OnlineOrderNotifier` shows a branded sonner toast with the Swiggy/Zomato logo via `ChannelLogo`, plus a chime) and a native OS `Notification` for when the app is unfocused. Payload type `OnlineOrderNotification` in `src/types/rista.ts`; channel→logo keyed off `sale.channel` containing "swiggy"/"zomato".
- **Config**: OMS needs `PUSHER_APP_ID/KEY/SECRET/CLUSTER` env vars; artha needs the public `PUSHER_KEY`/`PUSHER_CLUSTER` in `src/electron/config.ts` (baked in, not env, for packaged builds). Shared payload type: `src/types/rista.ts` (mirror of OMS `RistaSaleDetail`).
- **Catch-up / history**: the webhook also persists the full sale on `RistaOrder.saleSnapshot` (Prisma). The branch app's history page (`src/ui/pages/history/historyPage.tsx`) has two tabs — **Billing Orders** (offline POS bills, `get-bill-history` → `GET /api/bill/history`) and **Online Orders** (`get-online-order-history` → `GET /api/rista/orders`, cursor-paginated, branch-scoped). This is the fallback for online orders that arrived while the app was offline, since Pusher does not replay missed events.
- **Reprint**: Online Orders rows that are still in flight (status not Delivered/Cancelled) and have a `saleSnapshot` show a Reprint button → `reprint-online-order` IPC → `createRistaOrderTicketPDF` (same ticket as the automatic print). Local-only, no OMS call.

## Local Storage / Caching

- Billing records stored in local SQLite (`src/electron/localdb.ts`) for offline-first billing. Synced via `POST /api/bill/sync`.
- Menu and branch data cached in electron-store; refreshed on `sync-data-menu`.
- Branch-specific menu keyed as `menuItemsBranchwise_<branchId>` in electron-store.

## Key File Locations
- API base URL: `src/electron/config.ts`
- Shared types: `src/types/billing.ts`, `src/types/orders.ts`, `src/types/distribution.ts`, `src/types/menu.ts`
- IPC handlers: `src/electron/ipc/`
- Services: `src/electron/services/`
- Print utilities: `src/electron/utils/`
- Local DB: `src/electron/localdb.ts`
- UI pages: `src/ui/pages/billing/`, `src/ui/pages/`

---
> Source: [PavanShyamendra/sweetone-artha](https://github.com/PavanShyamendra/sweetone-artha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
