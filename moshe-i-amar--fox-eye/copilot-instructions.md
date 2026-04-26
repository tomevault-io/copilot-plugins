## fox-eye

> Fox-Eye (GeoMap) is a real-time operational tracking and geofencing app for hierarchical military-style organizations. Users share live location on a Leaflet map; admins define Areas of Operations (AOs) as GeoJSON polygons; the system detects breaches and fires real-time alerts.

# Fox-Eye — Claude Code Context

## What This App Does
Fox-Eye (GeoMap) is a real-time operational tracking and geofencing app for hierarchical military-style organizations. Users share live location on a Leaflet map; admins define Areas of Operations (AOs) as GeoJSON polygons; the system detects breaches and fires real-time alerts.

## Monorepo Structure
```
fox-eye/
├── client/     React 18 + Vite + Tailwind + Leaflet (ESM)
├── server/     Node.js + Express + MongoDB + Socket.IO (CommonJS)
└── .claude/    Claude Code config and commands
```

---

## Server (server/src/)

### Stack
- Node.js, **CommonJS** (`require`/`module.exports` — never use `import/export`)
- Express 4.x, Mongoose 8.x, Socket.IO 4.x
- JWT auth (jsonwebtoken + bcryptjs), express-validator, helmet, express-rate-limit, **cookie-parser**
- **web-push** — VAPID-based Web Push notifications (requires `VAPID_PUBLIC_KEY`, `VAPID_PRIVATE_KEY`, `VAPID_SUBJECT` env vars)
- **HttpOnly session cookie** — JWT issued as `HttpOnly; Secure; SameSite=Strict` cookie on login/register; `POST /api/auth/logout` clears it server-side

### Patterns — always follow these
```js
// Controller — always wrap with asyncHandler
const asyncHandler = require('../utils/asyncHandler');
exports.doThing = asyncHandler(async (req, res) => {
  res.json({ success: true, data: { ... } });
});

// Error — use AppError, never throw plain Error
const { AppError } = require('../utils/errors');
throw new AppError('ERROR_CODE', 'Human message', 404);

// Route — auth first, then validators, then controller
router.get('/path', auth, validators, controller.method);

// Transactions — wrap multi-doc writes
const withTransaction = require('../utils/withTransaction');
await withTransaction(async (session) => { ... });

// Audit — log all sensitive admin actions
const { logAdminAction } = require('../services/adminAuditService');
await logAdminAction({ action: 'invite.create', actorUserId, targetType: 'invite', targetId, after: doc });
```

### Response shape
```js
// Success
{ success: true, data: { ... } }

// Error
{ success: false, error: { code: 'ERROR_CODE', message: '...' } }
```

### Key files
| File | Purpose |
|------|---------|
| `src/app.js` | Express bootstrap, middleware, route mounting, request logging with token scrubbing |
| `src/config/db.js` | MongoDB connection with retry |
| `src/middleware/auth.js` | `auth`, `requireRole`, `requireOperationalRole` — reads token from `req.cookies.token` first, falls back to `Authorization: Bearer` header |
| `src/middleware/authorize.js` | `assertRoleEditAuthority`, `assertCompanyAccess`, `OPERATIONAL_ROLE_RANK`, `requireAdminOrCompanyCommander` |
| `src/middleware/errorHandler.js` | Global error handler |
| `src/utils/errors.js` | `AppError` class |
| `src/utils/asyncHandler.js` | Async wrapper for controllers |
| `src/utils/withTransaction.js` | MongoDB session/transaction helper |
| `src/utils/locationWriteBuffer.js` | `LocationWriteBuffer` — batches GPS location DB writes; `enqueue(userId, coords)` queues a write, `bulkWrite` fires every `LOCATION_WRITE_BUFFER_MS` ms (default 500); `shutdown()` drains on SIGTERM/SIGINT; `getStats()` for diagnostics |
| `src/utils/roles.js` | `OPERATIONAL_ROLES` array — source of truth for role enums |
| `src/utils/validators.js` | All express-validator chains: validateRegister (inviteToken required), validateCreateInvite, validateLogin, validateLocation, validateAOCreate/Update, validateFieldEventCreate, validateMobilizationCreate, validateMobilizationAdvance |
| `src/realtime/socket.js` | Socket.IO init, `getIO()` |
| `src/services/breachService.js` | Stateful AO breach detection |
| `src/services/locationService.js` | Location update handling; `updateUserLocation()` accepts optional `locationBuffer` — when provided, enqueues to `LocationWriteBuffer` instead of `User.save()` (broadcast still fires immediately); accepts optional `user` param to skip redundant `findById()` |
| `src/services/presenceService.js` | Connected user tracking |
| `src/services/scopeResolver.js` | Populates `req.scope` — what data the current user can see |
| `src/services/hierarchyService.js` | `resolveHierarchyPath()`, `ensureNoActiveChildren()`, `getHierarchyTree()` |
| `src/services/adminAuditService.js` | `logAdminAction()` — writes to AdminAuditLog collection |
| `src/controllers/inviteController.js` | `createInvite`, `validateInviteToken`, `listInvites`, `revokeInvite` |
| `src/controllers/eventController.js` | `createEvent`, `listEvents`, `acknowledgeEvent`, `resolveEvent` — fires push after create |
| `src/controllers/pushController.js` | `getVapidPublicKey`, `subscribe`, `unsubscribe` — Web Push subscription management |
| `src/models/InviteToken.js` | Invite token schema — token, roles, hierarchy, expiry, usedAt |
| `src/models/AdminAuditLog.js` | Audit trail schema — action, actorUserId, before/after |
| `src/models/FieldEvent.js` | Field event schema — eventType, senderId, GeoJSON Point coords, status lifecycle, denormalized hierarchy |
| `src/models/PushSubscription.js` | Web Push subscription schema — userId, endpoint, keys (p256dh/auth), denormalized role + hierarchy for scope targeting |
| `src/services/fieldEventService.js` | `createFieldEvent`, `listFieldEvents`, `acknowledgeFieldEvent`, `resolveFieldEvent`, `validateAntiSpoof` (velocity + timestamp skew) |
| `src/services/pushService.js` | `sendPushForFieldEvent(event)`, `sendPushForMobilization(mob)` — VAPID push to in-scope subscribers; auto-prunes expired endpoints (404/410) |
| `src/services/mobilizationService.js` | `createMobilization`, `getActiveMobilization`, `advanceMobilizationStatus`, `standDownMobilization`, `isUserInMobilizationScope` — full C2 mobilization lifecycle; enforces scope authority rules per operational rank |
| `src/controllers/mobilizationController.js` | `triggerMobilization`, `getActive`, `advanceStatus`, `standDown` — calls service, fires socket broadcast + push notification |
| `src/models/MobilizationEvent.js` | Mobilization schema — status (`ACTIVE`\|`TRANSIT`\|`PREPARATION`\|`DEPLOYMENT`\|`STOOD_DOWN`), triggeredBy, targetScope (`unitId` required; `companyId`/`teamId` narrow scope), rallyPoint (GeoJSON Point), rallyPointName, missionAOs, stoodDownAt/By; one active mobilization per unit enforced |
| `src/routes/eventRoutes.js` | Field event routes — POST rate-limited 10/hr per user ID |
| `src/routes/pushRoutes.js` | Push subscription routes — GET vapid-public-key (public), POST/DELETE /subscribe (auth) |
| `src/routes/mobilizationRoutes.js` | Mobilization routes — all auth-gated; POST validates with `validateMobilizationCreate` |

### Domain model
- **User**: location (GeoJSON Point, 2dsphere index), role (`admin`/`user`), operationalRole, unitId/companyId/teamId/squadId (all required), status (`active`/`pending`/`rejected`, default `active`), active (bool, default true), online (bool, default false), lastSeen (Date, default null) — `online`/`lastSeen` are maintained by `presenceManager` on socket connect/disconnect and set to `false`/`now` on logout
- **InviteToken**: token (32-byte hex, unique), createdBy, expiresAt, usedAt/usedBy, assignedRole, assignedOperationalRole, unitId/companyId/teamId/squadId, inviterName, active
- **AO**: GeoJSON Polygon, companyId, active, style
- **ViolationEvent**: type (`APPROACHING_BOUNDARY` | `BREACH` | `SUSTAINED_BREACH`), userId, aoId, coordinates, timestamp
- **AdminAuditLog**: action string (e.g. `invite.create`), actorUserId, targetType, targetId, before/after snapshots
- **FieldEvent**: eventType (`INJURED` | `AMBUSH` | `LINK_UP`), senderId, coordinates (GeoJSON Point, 2dsphere), status (`ACTIVE` | `ACKNOWLEDGED` | `RESOLVED`), acknowledgedBy/acknowledgedAt, resolvedBy/resolvedAt, unitId/companyId/teamId/squadId (denormalized from sender)
- **PushSubscription**: userId, endpoint (unique), keys.p256dh, keys.auth, role, unitId/companyId/teamId/squadId (denormalized for scope-based push targeting); upserted on subscribe, pruned on 404/410 response from push service
- **MobilizationEvent**: status (`ACTIVE` | `TRANSIT` | `PREPARATION` | `DEPLOYMENT` | `STOOD_DOWN`), triggeredBy (userId), incidentDescription, rallyPoint (GeoJSON Point), rallyPointName, targetScope (`{ unitId (required), companyId?, teamId? }` — narrowest scope wins), missionAOs ([{ aoId, assignedTeamId, objective }]), stoodDownAt/stoodDownBy; one active mobilization per unit enforced; indexes on `targetScope.unitId + status`
- **Hierarchy**: Unit → Company → Team → Squad (each has name, parentId, commanderId, active)
- **Coordinate convention**: MongoDB/GeoJSON = `[lng, lat]` · Leaflet = `[lat, lng]` ← common bug source

### Operational roles (descending authority)
`HQ` > `UNIT_COMMANDER` > `COMPANY_COMMANDER` > `TEAM_COMMANDER` > `SQUAD_COMMANDER`

Rank values (from `authorize.js`): HQ=5, UNIT_COMMANDER=4, COMPANY_COMMANDER=3, TEAM_COMMANDER=2, SQUAD_COMMANDER=1

### Account status lifecycle
Users have a `status` field: `active` | `pending` | `rejected`. The `auth` middleware throws 403 with specific error codes:

| Status | Error code | Meaning |
|--------|-----------|---------|
| `pending` | `AUTH_PENDING` | Account awaiting approval |
| `rejected` | `AUTH_REJECTED` | Account was rejected |
| `active=false` | `AUTH_INACTIVE` | Account deactivated |

### Invite system
Registration is invite-gated. Flow: admin creates invite → shares link → user visits `/register?token=...` → Register page validates token (`GET /api/auth/invite/:token`) → user submits form with inviteToken → server marks invite used and creates user with pre-assigned role/hierarchy.

**Invite rules:**
- Only HQ/UNIT_COMMANDER/COMPANY_COMMANDER can create invites
- Only HQ can create admin-role invites
- Actor rank must be > target operationalRole rank
- COMPANY_COMMANDER can only revoke their own invites
- Tokens expire after N days (1–30, default 7), single-use

### API endpoints
```
Auth (rate limited: 5/15min for login/register, 20/15min for invite):
  POST   /api/auth/register            # Invite-gated registration; sets HttpOnly session cookie
  POST   /api/auth/login               # Email/password login; sets HttpOnly session cookie
  POST   /api/auth/logout              # Clears session cookie + marks user online=false in DB (no auth required)
  GET    /api/auth/me                  # Current user profile (auth required)
  GET    /api/auth/invite/:token       # Validate invite token (public)

Users:
  GET    /api/users                    # All users (admin only)
  GET    /api/users/near               # Nearby users in scope (auth)
  GET    /api/users/:id                # Single user (admin)
  PUT    /api/users/me/location        # Update own location (auth)

Hierarchy (read-only):
  GET    /api/hierarchy/tree           # Full tree
  GET    /api/hierarchy/units|companies|teams|squads

AOs:
  GET    /api/aos                      # List AOs in scope (auth)
  POST   /api/aos                      # Create AO
  PUT    /api/aos/:id                  # Update AO
  PATCH  /api/aos/:id/active           # Toggle active
  DELETE /api/aos/:id                  # Delete AO

Violations:
  GET    /api/violations               # List in scope (auth)
  GET    /api/violations/:id           # Single violation

Field Events (rate limited: 10/hr per user for POST):
  GET    /api/events                   # List field events in scope (auth); ?status=&eventType=&page=&limit=
  POST   /api/events                   # Report field event — INJURED/AMBUSH/LINK_UP (auth); also fires Web Push
  PATCH  /api/events/:id/acknowledge   # Acknowledge active event (auth)
  PATCH  /api/events/:id/resolve       # Resolve event (auth)

Push Notifications:
  GET    /api/push/vapid-public-key    # VAPID public key (public — no auth)
  POST   /api/push/subscribe           # Save push subscription for current user (auth)
  DELETE /api/push/subscribe           # Remove push subscription (auth); body: { endpoint }

Mobilization (auth; TEAM_COMMANDER rank or above to write):
  GET    /api/mobilization/active      # Active mobilization(s) in caller's scope
  POST   /api/mobilization             # Trigger mobilization; body: { targetScope, incidentDescription?, rallyPoint?, rallyPointName? }
  PATCH  /api/mobilization/:id/status      # Advance phase: ACTIVE→TRANSIT→PREPARATION→DEPLOYMENT; body: { status }
  PATCH  /api/mobilization/:id/stand-down  # Stand down (triggerer, HQ, or UNIT_COMMANDER only)

Admin (auth + admin role + HQ/UC/CC operationalRole):
  GET    /api/admin/hierarchy/tree
  POST   /api/admin/companies|teams|squads
  PUT    /api/admin/companies|teams|squads/:id
  DELETE /api/admin/companies|teams|squads/:id
  POST   /api/admin/users              # Create user
  PUT    /api/admin/users/:id          # Update user
  PATCH  /api/admin/users/:id/active   # Toggle active
  PATCH  /api/admin/users/:id/roles    # Change roles
  POST   /api/admin/invites            # Create invite
  GET    /api/admin/invites?status=active|used|expired
  DELETE /api/admin/invites/:id        # Revoke invite

System:
  GET    /api/health
```

### Socket events
`location:update`, `location:request`, `presence:subscribe`, `viewport:subscribe`

`location:update` payload includes optional transient fields `heading` (0–360°) and `speed` (m/s) when provided by the device Geolocation API — not persisted to DB, broadcast-only.

Field event broadcasts (server → in-scope sockets):
`field:event:new` — payload `{ event, timestamp }` where `event` includes `senderName` (denormalized from sender doc at broadcast time)
`field:event:acknowledged`, `field:event:resolved`

Mobilization broadcasts (server → sockets in `targetScope`; admins always included):
`mobilization:activated` — payload `{ mobilization, timestamp }` — fired on POST /api/mobilization
`mobilization:status_changed` — payload `{ mobilization, timestamp }` — fired on phase advance
`mobilization:stood_down` — payload `{ mobilization, timestamp }` — fired on stand-down

Socket connect now accepts optional session type: `socketService.connect(token, { sessionType: 'MOBILE' | 'WEB' })` — stored as `socket.sessionType` on the server (transient, not persisted).

---

## Client (client/src/)

### Stack
- React 18 with hooks only (no class components)
- Vite 5 + **vite-plugin-pwa** (`injectManifest` strategy) — custom `src/sw.js` service worker with Workbox caching + Web Push handler; generated as `dist/sw.js`
- Tailwind CSS 3.x with custom dark theme
- Leaflet 1.9 + react-leaflet 4.x + leaflet-draw
- Axios with `withCredentials: true` (session cookie sent automatically) via `services/api.js` — no `Authorization` header needed
- Socket.IO client via `services/socketService.js` — connects with `withCredentials: true`; auth handled by session cookie in WS handshake
- Use `import.meta.env.VITE_*` (never `process.env`)

### Auth context — always use this, never read localStorage directly
```jsx
import { useAuth } from '../context/AuthContext';

const { user, authReady, login, logout, isAuthenticated } = useAuth();
// authReady = false while bootstrapping (show spinner, not redirect)
// login(userData) — stores user in state; server cookie already set by login/register API call
// logout() — async; calls POST /api/auth/logout to clear cookie server-side, then clears state
```
`AuthProvider` wraps the entire app in `App.jsx`. `useAuth()` must be called inside `AuthProvider`.

### Route protection — use RouteGuard, not the deleted route files
```jsx
// ProtectedRoute / AdminRoute / AdminManagementRoute are DELETED
// Use RouteGuard with props instead:
<RouteGuard requireRole="admin" requireOperationalRoles={['HQ','UNIT_COMMANDER']} fallback="/dashboard">
  <AdminManagement />
</RouteGuard>
```

### api.js account-state redirects
On 401 → redirects to `/login?reason=session-expired`
On 403 + account state code → redirects to `/login?reason=AUTH_PENDING|AUTH_REJECTED|AUTH_INACTIVE`
Login page reads `?reason=` and shows color-coded notice (gold=info, amber=warning, red=error).

### Design system — always match this
```
Backgrounds:   bg-jet (#0a0a0a)  bg-charcoal (#1a1a1a)  bg-slate-dark (#2d2d2d)
Primary:       text-gold (#C7A76C)   bg-gradient-to-r from-gold to-gold-light
Borders:       border-gold/20  border-gold/40
Effects:       shadow-gold-glow   backdrop-blur-glass
Animations:    animate-fade-in   animate-slide-up
Text:          text-white (primary)  text-gray-400 (secondary)  text-red-400 (error)
```

### Existing UI components — use these, don't recreate
```
components/ui/Button.jsx              — variants: primary | secondary | outline | ghost
components/ui/Card.jsx                — glass-card with gold border
components/ui/Input.jsx               — dark styled input
components/ui/Modal.jsx               — overlay modal
components/ui/AlertBanner.jsx         — contextual message bar; props: message, tone (warning|error|success|info), onDismiss, action ({ label, onClick, disabled })
components/ui/NotificationPrompt.jsx  — inline push-notification opt-in/opt-out banner; self-contained (uses useNotifications hook); hides if unsupported or denied
```

### Key files
| File | Purpose |
|------|---------|
| `src/App.jsx` | Routing wrapped in `<AuthProvider>`; uses `<RouteGuard>` for protected routes |
| `src/context/AuthContext.jsx` | `AuthProvider` + `useAuth()` hook — single source of truth for auth state |
| `src/components/layout/RouteGuard.jsx` | Composable route guard (props: requireRole, requireOperationalRoles, fallback) |
| `src/pages/Dashboard.jsx` | Orchestrator shell — composes DashboardMap + DashboardSidebar + AOModal + UserDetailModal + MobilizationModal; delegates socket events to useDashboardSocket, GPS to useLocationTracking, hierarchy to useHierarchyData, AO CRUD to useAOHandlers, mobilization lifecycle to useMobilization; computes `canMobilize` from role/operationalRole |
| `src/pages/dashboard/DashboardMap.jsx` | Leaflet map panel with MapController, viewport subscriber, user/AO/field-event markers; props: `center`, `users`, `userLocation`, `liveUpdateIds`, `aos`, `fieldEvents`, `canManageAOs`, `getCompanyIdentity`, `onViewportChange`, `onUserClick`, `onEventClick`, `onEventRespond`, `respondingIds`, AO CRUD callbacks; co-located events are grouped and rendered via `EventClusterSpider`; includes layer-switcher FAB (bottom-right, above locate-me FAB) toggling between CartoDB dark and Esri World Imagery + labels hybrid |
| `src/pages/dashboard/EventClusterSpider.jsx` | Field event cluster + spider component — renders a count badge when 2+ events share the same location; click to spider-expand with radial dashed lines and per-event ACK/RESOLVE cards (portal-based overlay); props: `events`, `position`, `onRespond`, `respondingIds` |
| `src/pages/dashboard/DashboardSidebar.jsx` | Composes MobilizationPanel (top) + AOPanel + FieldEventsPanel + ViolationsPanel + user list + location controls |
| `src/pages/dashboard/AOPanel.jsx` | AO list with edit/enable/disable; uses `isImageUrl` from markerUtils |
| `src/pages/dashboard/FieldEventsPanel.jsx` | Events list with ACK/RESOLVE buttons |
| `src/pages/dashboard/ViolationsPanel.jsx` | Violations list with severity/company/date filters |
| `src/pages/dashboard/AOModal.jsx` | AO create/edit form modal |
| `src/pages/dashboard/UserDetailModal.jsx` | User detail popup |
| `src/pages/dashboard/useAOHandlers.js` | AO CRUD state machine — `aoDraft`, `aoModalMode`, `aoForm`, `handleAOCreate/Edit/Delete/Select/Cancel/Submit`, `handleToggleAOActive` |
| `src/pages/dashboard/useHierarchyData.js` | Loads hierarchy tree, returns `{ hierarchyMap, companyOptions, teamOptions }` — `teamOptions` is the raw teams array (each has `_id`, `name`, `parentId`) used to filter teams by company in MobilizationModal |
| `src/pages/dashboard/MobilizationPanel.jsx` | Sidebar widget — shows pulsing red MOBILIZE button when no active mobilization (visible only when `canMobilize`); shows active mobilization status card with phase progress dots, phase-advance button, and stand-down (with tap-to-confirm safety); props: `canMobilize`, `activeMobilization`, `mobilizationLoading`, `mobilizationSending`, `mobilizationError`, `onOpenModal`, `onAdvance`, `onStandDown` |
| `src/pages/dashboard/MobilizationModal.jsx` | Slide-to-confirm mobilization trigger modal; internal `SlideToConfirm` uses pointer-capture drag (92% threshold locks, 5-second countdown fires `onConfirmed`); role-based scope form: TC=both locked, CC=company locked, UC/HQ=free company+team selection; exports `MOBILIZE_ROLES` array and default `MobilizationModal` |
| `src/pages/dashboard/useLocationTracking.js` | GPS watchPosition + throttled location send; returns `{ userLocation, locationLoading, locationError }`; calls `onFirstLocation` once on first fix; captures `coords.heading` + `coords.speed` from Geolocation API and forwards them to `socketService.updateLocation()` |
| `src/pages/dashboard/useDashboardSocket.js` | All Dashboard socket event handlers (location, presence, field events, AO realtime); returns `{ liveUpdateIds, onlineUsers }`; merges `heading` + `speed` from `location:update` payload into user objects |
| `src/pages/Admin.jsx` | AO management, violations list; uses `useSocketConnection` hook |
| `src/pages/admin/AdminUserDetailModal.jsx` | Full user details modal for Admin page |
| `src/pages/admin/BreachAlertPanel.jsx` | Breach alerts list with auto-dismiss |
| `src/pages/AdminManagement.jsx` | Thin shell (~100 lines) — destructures `useAdminManagement()` and renders tabs + modals |
| `src/pages/admin-management/useAdminManagement.js` | All AdminManagement state + handlers in one hook |
| `src/pages/admin-management/HierarchyModals.jsx` | Exports `CompanyModal`, `TeamModal`, `SquadModal` |
| `src/pages/admin-management/UserModals.jsx` | Exports `UserModal`, `RolesModal`, `ConfirmModal` |
| `src/pages/admin-management/InviteModal.jsx` | Invite link generation modal |
| `src/pages/admin-management/HierarchyTree.jsx` | Hierarchy tree visualization (Unit → Company → Team → Squad) |
| `src/pages/Register.jsx` | Invite-gated registration — requires `?token=` URL param |
| `src/pages/Login.jsx` | Login + account-status notice display (reason codes in URL) |
| `src/services/api.js` | Axios instance with JWT interceptor + account-state redirect logic |
| `src/services/authApi.js` | `login()`, `register()`, `getMe()`, `validateInvite(token)` |
| `src/services/adminApi.js` | Admin CRUD + `createInvite()`, `listInvites()`, `revokeInvite()` |
| `src/services/eventApi.js` | `createEvent()`, `getEvents()`, `acknowledgeEvent()`, `resolveEvent()` |
| `src/services/mobilizationApi.js` | `mobilizationApi.getActive()`, `.trigger(payload)`, `.advanceStatus(id, status)`, `.standDown(id)` — C2 mobilization lifecycle calls |
| `src/services/socketService.js` | Socket.IO client singleton; `connect(token, { sessionType })`; `updateLocation(coordinates, { heading, speed })` — bearing fields optional |
| `src/services/pushService.js` | Client push helpers: `subscribeToPush()`, `unsubscribeFromPush()`, `getSubscription()`, `requestNotificationPermission()` |
| `src/hooks/useFieldEvents.js` | Fetch + live socket subscription for field events; returns `{ events, loading, error, refetch, updateEvent }`; `updateEvent(id, patch)` applies optimistic local update; socket ACK/RESOLVE handlers merge onto existing entry to preserve populated `senderId` |
| `src/hooks/useOfflineQueue.js` | Persist + flush field events queued while offline; returns `{ queue, enqueue, flush }` — backed by `localStorage` |
| `src/hooks/useNotifications.js` | Push notification state — `{ supported, permission, subscribed, loading, error, enable, disable }` |
| `src/hooks/useMobilization.js` | Mobilization state hook; params `{ realtimeEnabled }`; returns `{ activeMobilization, loading, sending, error, trigger, advanceStatus, standDown, clearError }`; loads active mobilization on mount; subscribes to `mobilization:activated/status_changed/stood_down` socket events when `realtimeEnabled` |
| `src/hooks/useSocketConnection.js` | Shared socket lifecycle hook used by Dashboard and Admin; params `{ navigate, onConnect }`; returns `{ realtimeEnabled, realtimeStatus, realtimeNotice, realtimeNoticeTone, clearRealtimeNotice }` |
| `src/utils/markerUtils.js` | All SVG marker builders + Leaflet Icon factories + event configs; exports `sanitizeColor`, `isImageUrl`, `isValidIconValue`, `snapHeading`, `buildBearingArrow`, `buildUserPinSvg` (accepts `heading`/`speed` for directional arrow), `buildEventMarkerSvg`, `buildClusterMarkerSvg`, `createUserMarkerIcon` (accepts `heading`/`speed`), `createEventMarkerIcon`, `createClusterMarkerIcon`, `getEventIconCached` (module-level cache), `EVENT_TYPE_CONFIG`, `EVENT_STATUS_OPACITY` |
| `src/utils/mapGeometry.js` | Map geometry helpers: `isPointInPolygon`, `findAoForPoint`, `toGeoPolygon`, `toLatLngs`, `calculateDistance`, `normalizeCoords` |
| `src/utils/roleUtils.js` | `OPERATIONAL_ROLE_RANK` map and `operationalRoles` array — shared between client pages and admin logic |
| `src/utils/formatUtils.js` | `formatTimestamp`, `formatViolationType` — shared formatting helpers |
| `src/sw.js` | Custom service worker source (injectManifest mode) — Workbox precache + runtime caching + `push` event handler + `notificationclick` handler |
| `src/pages/mobile/MobileFieldView.jsx` | Mobile page — GPS watcher, socket MOBILE session, Wake Lock, network-status detection, offline queue flush; uses `useFieldEvents(25)`, `useAOs`, `useViolations`, `useAuth`; derives composite `connectionStatus` from socket state + AO error; passes `feedSlot`/`notificationSlot`/`user` to MobileLayout; `handleShowOnMap` sets `mapFocusCoords` only (no tab switch — map always visible); owns `respondingIds` Set + `handleEventRespond` (calls `eventApi`) and passes both to `MobileFieldMap`; GPS watcher forwards `heading`/`speed` to `socketService.updateLocation()`; route `/mobile` |
| `src/pages/mobile/MobileLayout.jsx` | Phase 2 mobile shell — `100dvh`, map as `absolute inset-0 z-0`, TopNav at `z-[500]`, BottomReportingSheet at `z-[400]`; props: `mapSlot`, `user`, `connectionStatus`, `gpsStatus`, `wakeLockStatus`, `violations` (unused by TopNav, reserved), `userCoordinates`, `onQueueEvent`, `queuedCount`, `feedSlot`, `notificationSlot`, `onBack` |
| `src/pages/mobile/components/TopNav.jsx` | Glassmorphism 3-column header — left: fox SVG logo; center: inline `NetworkDot` (green=connected, red=reconnecting/disconnected); right: burger button that opens `MenuDrawer` slide-in panel; `MenuDrawer` shows user profile, GPS status, wake-lock state, Back to Dashboard; props: `connectionStatus`, `gpsStatus`, `wakeLockStatus`, `user`, `onBack` |
| `src/pages/mobile/components/LiveStatusIndicator.jsx` | Memoized tri-state dot — green+pulse=connected, amber+pulse=reconnecting, red=disconnected; optional `violations` badge; isolated so status changes never re-render the map layer; no longer used by TopNav (replaced by inline `NetworkDot`) |
| `src/pages/mobile/components/BottomReportingSheet.jsx` | Draggable action sheet (`z-[400]`); collapsed=handle+chevron only (~2.75rem); expanded=two tabs: "Report" (ReportingGrid) and "Events" (feedSlot); swipe-up/down + tap-to-toggle; props: `userCoordinates`, `onQueueEvent`, `queuedCount`, `feedSlot`, `notificationSlot` |
| `src/pages/mobile/components/ReportingGrid.jsx` | Memoized 2×2 action grid; top row=SOS (INJURED red, AMBUSH amber), bottom row=Contact (LINK_UP green) + EVENTS shortcut; min-h-[80px] cells; haptic feedback; uses `SignalConfirmModal`; offline queuing; props: `userCoordinates`, `onQueueEvent`, `queuedCount`, `onViewEvents`, `disabled` |
| `src/pages/mobile/components/SignalConfirmModal.jsx` | Confirm dialog for ReportingGrid signal sends; props: `pending`, `sending`, `noGps`, `onClose`, `onConfirm` |
| `src/pages/mobile/MobileFieldMap.jsx` | Hierarchy-scoped Leaflet map; imports `getEventIconCached`, `EVENT_TYPE_CONFIG`, `createClusterMarkerIcon`, `createUserMarkerIcon`, `createSelfMarkerIcon`, `snapHeading` from markerUtils; props: `userCoordinates`, `events`, `focusCoords`, `onFocusConsumed`, `initialZoom`, `showZoomControl`, `onEventRespond`, `respondingIds`; groups co-located teammates into cluster markers AND co-located events into `EventClusterSpider`; teammate markers are bearing-aware (gold, uses `heading`/`speed` from presence data); flies to `focusCoords`; includes center-on-me FAB and layer-switcher FAB (OSM dark ↔ Esri satellite hybrid) |
| `src/pages/mobile/MobileEventFeed.jsx` | Live field events list; delegates each row to `EventCard`; props: `events`, `loading`, `error`, `refetch`, `onShowOnMap`, `onEventUpdate` |
| `src/pages/mobile/EventCard.jsx` | Single event row — type/status badges, sender info, show-on-map button, ACK/RESOLVE buttons; props: `event`, `isPending`, `onShowOnMap`, `onRespond` |

### Map patterns
- `MapContainer` parent div must have explicit height (`h-screen`, `h-[calc(...)]`, or `h-full` inside a flex container)
- **Mobile map pattern — always use `absolute inset-0`**: wrap `MapContainer` in `<div className="absolute inset-0">` inside a `relative flex-1 min-h-0` shell. `height:100%` chains through flex-item computed sizes fail on mobile Safari; `absolute inset-0` gives Leaflet explicit pixel bounds that always resolve correctly.
- Use `useMap()` hook for imperative control (`setView`, `flyTo`)
- Leaflet coordinates are `[lat, lng]` — opposite of GeoJSON

---

## Common Bug Patterns

| Symptom | Likely cause |
|---------|-------------|
| `useAuth` crash / context undefined | `useAuth()` called outside `<AuthProvider>` — check component tree in App.jsx |
| Register shows "Invite Required" modal | No `?token=` in URL — must use invite link |
| 403 AUTH_PENDING on login | `user.status === 'pending'` in DB — account not yet activated |
| Socket 401 after login | Session cookie not present or expired — ensure `withCredentials: true` on socket connect and CORS `credentials: true` on server |
| Login redirects to `/login?reason=session-expired` immediately | `AuthContext` bootstrap `getMe()` 401 is triggering the interceptor — the 401 interceptor must skip `/api/auth/me` calls (see `api.js`) |
| Server crashes on startup with "invalid directive value for connect-src" | `CLIENT_ORIGIN` env var contains comma-separated origins — CSP builder must `.split(',')` and spread them individually |
| AO breach false positives | `AO_BREACH_GRACE_MS` env var too low |
| Wrong marker position | Coordinate swap — passing `[lng, lat]` to Leaflet |
| Map invisible / zero height | Missing height on `MapContainer` parent div |
| Vite env var undefined | Using `process.env` instead of `import.meta.env` |
| React infinite re-render | Missing `useCallback`/`useMemo` in Dashboard deps |
| Scope error non-admin | `scopeResolver.js` not populating `req.scope` correctly |
| Invite creation fails with rank error | Actor operationalRole rank ≤ target rank — only higher ranks can invite lower |
| Mobile map invisible | MobileLayout Phase 2 uses `absolute inset-0 z-0` for the map inside a `relative overflow-hidden` root — ensure `MobileFieldMap`'s root div is `absolute inset-0 z-0` and `MapContainer` is wrapped in `<div className="absolute inset-0">` |
| ReportingGrid button disabled unexpectedly | `disabled={false}` is hardcoded in `BottomReportingSheet` — buttons are never gated by socket state; if appearing disabled check the `disabled` prop passed to `ReportingGrid` |
| Field event POST 422 velocity | Two rapid events with large coordinate delta — `validateAntiSpoof` in `fieldEventService.js` |
| Panic send silently queued instead of sent | `navigator.onLine === false` or no `err.status` → `useOfflineQueue.enqueue()` was called; check `queuedCount` badge on ReportingGrid |
| Wake lock not acquired | Browser requires a user gesture before `wakeLock.request()` on some versions; also unavailable in non-secure (non-HTTPS) contexts |
| Push subscription silently fails | VAPID keys missing from server `.env` — `GET /api/push/vapid-public-key` returns 503; generate keys with `npx web-push generate-vapid-keys` |
| Push prompt never appears | Browser blocked notifications (permission=`denied`) or non-HTTPS context — `NotificationPrompt` hides itself in both cases |
| SW push handler missing after build | `vite.config.js` must use `strategies: 'injectManifest'` pointing to `src/sw.js`; do not switch back to `generateSW` mode |
| Field event markers not showing on map | `ev.coordinates.coordinates` is `[lng, lat]` (GeoJSON) — Leaflet needs `[lat, lng]`; also check that `useFieldEvents` is mounted and events have valid coords |
| Offline/logged-out users still visible on map | `GET /api/users/near` only returns users where `online: true` OR `lastSeen` within 1 hour — if a user appears stale, check `presenceManager` flushed `online=false` to DB; server crash leaves stale `online:true` cleared by `PresenceManager.resetStalePresence()` on startup |
| Bearing arrow never shows even when moving | Browser/device returns `null` for `GeolocationCoordinates.heading` (common on desktop and some Android); arrow is intentionally suppressed when heading is null or `speed < 0.5 m/s` — this is correct fallback behaviour, not a bug |
| Satellite tiles blocked / blank in hybrid mode | Server CSP `img-src` must include `https://server.arcgisonline.com` — add it in `server/src/app.js` Helmet config alongside the OSM origin |

---

## Security Architecture

### Auth: HttpOnly Cookie (not localStorage)
- Login/register set `token` as `HttpOnly; Secure; SameSite=Strict` cookie — JWT is never accessible to JavaScript
- `POST /api/auth/logout` clears the cookie server-side (browser sends cookie, server sets `Max-Age=0`)
- `auth.js` middleware reads `req.cookies.token` first, falls back to `Authorization: Bearer` for tooling
- Socket.IO auth reads the cookie from `socket.handshake.headers.cookie` via `parseCookieToken()` in `socketAuth.js`
- Client: axios uses `withCredentials: true`; no `Authorization` header interceptor; `socketService.connect()` uses `withCredentials: true`

### Content Security Policy
Helmet is configured with an explicit CSP in `server/src/app.js`:
- `script-src 'self'` — no inline scripts, no foreign script loads
- `style-src 'self' 'unsafe-inline'` — Leaflet requires inline styles
- `img-src` — `data:`, `blob:`, OpenStreetMap tile origins, and `https://server.arcgisonline.com` (Esri satellite layer) explicitly allowlisted
- `connect-src` — API server + WebSocket origins derived from `CLIENT_ORIGIN` env var
- **`CLIENT_ORIGIN` is comma-separated** — always `.split(',')` before spreading into CSP directives; passing the raw string crashes Helmet

### SVG Injection Prevention
Map marker SVGs interpolate colors from DB (company color field). Two defenses in `src/utils/markerUtils.js`:
- `sanitizeColor(value)` — strict hex (`#abc` / `#aabbcc`) and `rgb()`/`rgba()` allowlist; falls back to `DEFAULT_AO_COLOR`
- `isImageUrl(value)` — explicitly blocks `data:image/svg+xml` to prevent SVG-in-SVG script execution; allows `data:image/<non-svg>`, `https://`, `blob:`, `/path`

## Dev Commands
```bash
# Server
cd server && npm run dev      # nodemon on port 5000

# Client
cd client && npm run dev      # Vite on port 5173
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Moshe-I-Amar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
