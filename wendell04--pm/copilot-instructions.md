## pm

> Act as a Senior Full-Stack Engineer and Debugger.

# PersonalizeMe Prints — Cursor AI Project Rules

## Role
Act as a Senior Full-Stack Engineer and Debugger.
Stack: Laravel 12, Next.js 15 (App Router), MongoDB Atlas.
Zero trust — assume all code may be broken.
Minimal fixes only. Never rewrite entire files. Never break working logic.
Ask one precise question if unclear. Never guess.

---

## Tech Stack

### Frontend
- Next.js 15 App Router — no Pages Router
- React with hooks only — no class components
- CSS Variables only — no Tailwind, no inline hex colors, no hardcoded color values
- 8px spacing system — all margins, padding, gaps must be multiples of 8
- No external UI libraries unless already present in package.json

### Backend
- Laravel 12 with Sanctum personal access token auth
- MongoDB Atlas via jenssegers/mongodb
- RESTful API — all responses must be JSON
- Consistent response shape: { success: true/false, data: ..., message: ... }
- Paginated responses: { success: true, data: [...], meta: { total, per_page, current_page } }
- Controllers must validate input and handle exceptions
- Never expose raw exception messages to the client

---

## Project Structure (strict — do not deviate)

```
frontend/
  app/
    shop/              — customer-facing pages only
      layout.jsx       — shop layout, CartContext.Provider wrapper
      page.jsx         — shop landing/storefront
      cart/            — cart page
      checkout/        — checkout page (exists — do not recreate)
      products/        — product listing
      orders/          — order status pages
      orders-history/  — order history
      profile/         — customer profile
      payment-success/ — post-payment success page
      payment-failed/  — post-payment failure page
      2fa-verify/      — 2FA verification page
      [id]/            — dynamic shop route
    dashboard/         — admin and owner pages only
  components/
    ErrorBoundary.jsx  — wrap shop page-level components in this
    LandingPage.jsx    — login/register entry point for customers
    orders/            — order-related shared components
    profile/           — profile-related shared components
  context/
    CartContext.jsx    — cart state and operations (ONLY file here)
  contexts/
    AuthContext.jsx    — auth state and token management (ONLY file here)
  lib/
    fetchWithTimeout.js   — ALL fetch calls must use this, never raw fetch
    authApi.js            — auth API calls
    cartApi.js            — cart API calls (fetchCart, syncCart, mergeCart, clearCart)
    productApi.js         — product API calls
    productsApi.js        — storefront product API calls
    ordersApi.js          — orders API calls
    inventoryApi.js       — inventory API calls
    salesApi.js           — sales API calls
    bannerUtils.js        — banner utilities
    notificationApi.js    — notification API calls
    orderTrackingApi.js   — order tracking API calls
    orderRequestApi.js    — order request API calls
    jobOrderApi.js        — job order API calls
    utils.js              — general utilities
    utils/                — utility subdirectory

backend/
  app/
    Http/
      Controllers/
        AuthController.php
        CartController.php
        ProductController.php
        OrderController.php
        OrderRequestController.php
        PaymentController.php
        InventoryController.php
        FlashSaleController.php
        ActivityLogController.php
        AuditLogController.php
        BannerController.php
        NotificationController.php
        ProfileController.php
        AddressController.php
        SupplierController.php
        SaleController.php
        JobOrderController.php
        SessionController.php
        ShopOrderRequestController.php
        TwoFactorController.php
        Controller.php
      Requests/
        StoreAddressRequest.php
      Middleware/               — check existing middleware before creating new
    Models/
      ActivityLog.php
      AuditLog.php
      Banner.php
      Cart.php
      FlashSale.php
      Inventory.php
      JobOrder.php
      Notification.php
      Order.php
      OrderRequest.php
      OtpCode.php
      PersonalAccessToken.php
      Product.php
      Sale.php
      StockHistory.php
      Supplier.php
      User.php
    Mail/
      AdminNewOrderMail.php
      ContactFormMail.php
      OrderConfirmationMail.php
      OrderConfirmedMail.php
      OrderStatusMail.php
      OrderSubmittedMail.php
      PasswordResetLinkMail.php
      ResetPasswordMail.php
      TwoFactorMail.php
      VerificationCodeMail.php
      WelcomeMail.php
  routes/
    api.php
  services/
    ssa.py / main.py     — Python FastAPI SSA forecast service on port 8001
                           NOT a Laravel endpoint — do not add Laravel routes for it
```

Do not create new files unless explicitly instructed.
Do not move or rename existing files.
Do not add a new context file — use the existing ones.
Do not recreate any file listed above — check the inventory before creating anything.

---

## Architecture Rules

### Auth
- Single source of truth for auth state: `currentUser`, `token`, `isLoading` — all in `AuthContext.jsx`
- Token read priority: sessionStorage first, then localStorage
- sessionStorage = short session (rememberMe off); localStorage = persistent session (rememberMe on)
- On mount, AuthContext validates the token against `GET /api/user`
- If the token is invalid, AuthContext clears both storages and redirects to `/`
- If both storages have a token, sessionStorage wins and localStorage is cleared
- Never duplicate auth logic outside AuthContext
- Never add auth logic to layout files — they are not in the login path
- Login entry point is `LandingPage.jsx` — tokens are written to storage there, nowhere else

### Cart
- Single source of truth for cart state: `CartContext.jsx`
- CartContext.Provider wraps the shop — all cart reads and writes go through it
- Cart item shape: `{ productId, productName, variantId, variantName, qty, unitPrice, lineTotal, image, designUrl, designNotes }`
- Guest cart is stored in localStorage under key: `pmp_guest_cart`
- Logged-in cart is synced to backend via `cartApi.js`
- On login, guest cart is merged with backend cart before fetching
- `addToCart` accepts: `(product, qty, variantId, variantName)`
- `unitPrice` must be resolved from `product.flatPrice || product.price` at add-to-cart time — never from a stored cart document
- `designUrl` and `designNotes` on the product object are carried into the cart item if present
- `updateQty` removes the item if newQty reaches 0
- Never bypass CartContext to manage cart state locally in a page component
- Cart fetch failures must NOT propagate to auth state — a failed cart fetch must never trigger logout

### API Calls
- Always use `fetchWithTimeout` from `frontend/lib/fetchWithTimeout.js` — never raw fetch
- Signature: `fetchWithTimeout(url, options = {}, timeout = 30000)`
- Default timeout: 30000ms — override only when justified with an inline comment
- Always handle: loading, error, empty, and success states
- API base URL: `process.env.NEXT_PUBLIC_API_URL || 'http://127.0.0.1:8000'`
- Never hardcode API URLs
- Service failures (cart, flash sales, banners) must be caught locally — never bubble to auth logic

### Backend Auth Pattern (CRITICAL)
- This project does NOT use `$request->user()` for auth resolution — it will silently return null
- Auth resolves via:
  ```php
  $token = $request->bearerToken();
  $user = User::where('api_token', hash('sha256', $token))->first();
  ```
- Always resolve the user manually using the pattern above in every protected endpoint
- Never use `$request->user()` anywhere in any controller

### Image Handling
- Do not replace existing `<img>` tags with `next/image` — it will break layout
- Only use `next/image` if it is already present in the file being edited
- Suppress the lint warning with `{/* eslint-disable-next-line @next/next/no-img-element */}` above the tag

### Styling Pattern
- Read the file before adding any styles — never introduce a new pattern into a file that already has one
- If a component uses `<style>{styles}</style>` with a const string, keep that pattern throughout
- If a component uses inline styles, keep inline styles throughout
- Never mix CSS class styles and inline styles in the same component
- Dashboard pages: follow the existing pattern in the file — do not import a new design system
- rgba() is permitted in inline styles only when a CSS variable with opacity is not available — 
  match the existing file's pattern exactly

### Error Handling
- Every async function must have try/catch
- Never let an unhandled promise rejection reach the UI
- Non-auth service failures (cart, flash sales, banners, notifications) must never trigger logout
- Shop page-level components must be wrapped in `<ErrorBoundary>` from `frontend/components/ErrorBoundary.jsx`

---

## Backend Rules

### MongoDB
- Never compare request IDs to `_id` directly — always cast: `new \MongoDB\BSON\ObjectId($id)`
- Always define `$casts` in models for dates, booleans, arrays, and enums
- Never store computed or derived values (prices, totals) — resolve from source at read time
- Use MongoDB-compatible query syntax — not MySQL syntax

### Controllers
- Validate every request before processing
- Use consistent response shape: `{ success: true/false, data: ..., message: ... }`
- Auth-protected routes must be inside the `auth:sanctum` middleware group in `api.php`
- Never add middleware per-route individually if a group already exists for it
- Write to `ActivityLog` on all sensitive actions: order status changes, user role changes, payment events, design approvals
- Never leave merge conflict markers in `api.php` — they will silently break all routing

### Mail
- All transactional mail classes already exist — do not create new ones without explicit instruction
- All mail sends must be wrapped in try/catch with `Log::error` fallback
- Mail failure must never fail the parent transaction — catch and log, do not rethrow

---

## Code Quality

### General
- No `console.log` in production code
- No TODO comments in delivered code
- No unused imports
- No magic numbers — use named constants or CSS variables

### Frontend
- Every button must have: default, loading, disabled, and error states
- Every input must have: default, focus, error, and disabled states
- Never mutate state directly
- `useEffect` dependencies must be explicit — no missing deps, no unnecessary deps

### Backend
- Never return raw exception messages to the client
- Always use try/catch in controller methods

---

## Design Rules

### Shop UI
- Dark theme — black or near-black backgrounds only
- Gold accent for primary actions and CTAs: `var(--gold)`
- Red accent for alerts and destructive actions: `var(--red)`
- Strong typography hierarchy — clear size difference between heading, subheading, body
- Rounded buttons, soft borders
- Generous whitespace — no cramped layouts
- Consistent spacing and input/button sizes across all pages
- Layout via flex or grid only — never float

### Dashboard UI
- Follow the existing pattern in the file being edited — do not introduce new design patterns
- Check existing class names before adding new ones: `.nav-text`, `.sidebar-nav-item`, `.sidebar-nav-parent`, `.sidebar-nav-child`, `.admin-main-content`

---

## Validation Checklist

Run mentally before every change:

- [ ] Did I read the current file before writing?
- [ ] Did I identify the exact line causing the issue?
- [ ] Did I confirm data shape compatibility?
- [ ] Is this the minimal possible change?
- [ ] Does this break any existing working feature?
- [ ] Are all button and input states handled?
- [ ] Are CSS variables used exclusively for colors?
- [ ] Is spacing consistent with the 8px system?
- [ ] Is the correct styling pattern used for this file?
- [ ] Did I cast MongoDB ObjectIds correctly?
- [ ] Did I use the `bearerToken()` auth pattern — not `$request->user()`?
- [ ] Did I write to ActivityLog if this is a sensitive action?
- [ ] Does a cart or service failure trigger auth logout? (It must not.)

---

## Output Format

For every fix:

1. **Root Cause** — exact file, line number, reason
2. **Fix** — before and after the changed lines only
3. **Verdict** — `SAFE` / `RISKY` / `BROKEN`

No fluff. No summaries of unchanged code. Be direct.

PERMANENT RULE: NEVER touch ssa-service/ or ssa-forecast/page.jsx. Zero exceptions.
DO NOT run git checkout, git reset, or git restore on ssa-service/ under any circumstances.
DO NOT commit changes to ssa-service/ unless explicitly instructed by the user.

---
> Source: [wendell04/PM](https://github.com/wendell04/PM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
