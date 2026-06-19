## easier-to-deploy-storefronts

> Build a multilingual storefront + owner inbox + passwordless auth for a solo service business — one owner at the counter, many customers passing through. Convex (state) + Vercel (functions + static) + Resend (email) + OpenRouter (translation). Use this skill any time you start a new solo-operator project (pet-sitter, tutor, photographer, freelance editor, mobile mechanic) and need the same shape: editable storefront, single inbox, magic-link owner login, no developer in the loop after deploy.


# easier-to-deploy-storefronts — building a one-owner / many-customers stack

You're about to build a website for a solo operator — one human running a service business who needs a presence on the web but isn't a developer. They will edit prices on their phone, see customer messages in a single inbox, and sign in via email link. You will deploy once and never log into the production database again.

This skill is the **code-construction** playbook. For the **dev process around the code** — clarifying scenarios before you write anything, designing UX with [`parity-studio`](https://github.com/HomenShum/parity-studio), enforcing per-surface changelogs with [`easier-to-read-submissions`](https://github.com/HomenShum/easier-to-read-submissions), evaluating agents, and the scaling math past the free tier — read [`DEV_FLOW.md`](DEV_FLOW.md). It's the meta-process; this is the implementation.

Ten phases, in order. Each phase explains **what** to build, **why** that design (the calls that aren't obvious), and **how** (concrete code and commands). The phases are sequenced so you can stop after any phase and have something working — Phase 4 alone gives you a multilingual marketing site; adding Phase 5 makes the owner self-sufficient.

The whole thing runs on free tiers for steady-state operation. Estimated time to a deployed v1 with a non-technical owner: **6–8 hours** for someone new to Convex, **2–3 hours** for someone who's done it before.

---

## Phase 0 — Decide whether this skill applies

This skill fits a project with **all** of:

- One owner (not a multi-tenant SaaS, not a multi-vendor marketplace).
- Many anonymous customers who don't need accounts (they fill a form and get an email reply).
- Owner is non-technical or wants to stay that way.
- Content (prices, copy, photos, FAQs) changes more than once a quarter.
- Multilingual is a real requirement, not aspirational. (English-only? Skip Phase 8.)

If the project has a **second class of authenticated user** (employees, partners, vendors), stop and use a different skill — `easier-to-deploy-storefronts` is deliberately scoped to one-owner. Adding a second admin role doubles the auth surface; it's not a small change.

---

## Phase 1 — Provider setup (30 min)

Create accounts and grab keys. Don't paste keys into chat or anywhere except the env file.

| Provider | What for | Free tier covers |
|---|---|---|
| Vercel | Static hosting + serverless functions | Unlimited Hobby projects, 100GB bandwidth/mo, 100K function invocations/day |
| Convex | DB + file storage + cron + HTTP routes | 1GB storage, 1M function calls/mo, no credit card |
| Resend | Transactional email | 3000 emails/mo, 100/day, one verified domain free |
| OpenRouter | LLM router for translation + reply assist | Pay-as-you-go; ~$0.001–$0.01 per translation. Free fallback models available |

```bash
# Scaffold the project
npm create convex@latest <your-name>      # creates /convex with schema.ts + http.ts stubs
cd <your-name>
npx convex dev                            # one-time login + creates dev deployment

# Add Vercel
npm i -g vercel
vercel link                               # creates .vercel/, links to a Vercel project

# Project structure that emerges:
.
├── convex/                # backend (schema, functions, http routes, crons)
├── api/                   # Vercel serverless functions
├── index.html             # public landing page (English)
├── zh/index.html          # Chinese
├── es/index.html          # Spanish
├── inbox.html             # owner SPA
├── .env.example           # documented env vars
├── vercel.json            # rewrites: /admin → /inbox.html, /api/* → functions
└── package.json
```

`vercel.json` should contain at minimum:

```json
{
  "rewrites": [
    { "source": "/admin", "destination": "/inbox.html" },
    { "source": "/admin/", "destination": "/inbox.html" }
  ]
}
```

The `/admin → /inbox.html` rewrite is a small UX detail that matters: the owner remembers `your-site.com/admin`, not `/inbox.html`.

---

## Phase 2 — Convex schema (45 min)

Five tables. Define them all in `convex/schema.ts` upfront — even the ones you won't use until Phase 6 — so the schema is self-documenting.

```ts
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  // ============== Customer-facing ==============
  inquiries: defineTable({
    name: v.string(),
    contact: v.string(),                       // email or phone
    message: v.string(),
    dog: v.optional(v.string()),               // adapt: "child" / "vehicle" / "project" for your domain
    dates: v.optional(v.string()),
    service: v.optional(v.string()),
    lang: v.optional(v.string()),
    detectedLang: v.optional(v.string()),      // OpenRouter source-language detection
    translatedMessage: v.optional(v.string()), // translation for owner's reading language
    threadId: v.string(),
    type: v.literal("inquiry"),
    direction: v.literal("inbound"),
    status: v.string(),
    timestamp: v.number(),
    replied: v.boolean(),
    repliedAt: v.optional(v.number()),
    replyMessage: v.optional(v.string()),
    replyMessageOriginal: v.optional(v.string()),
  })
    .index("by_timestamp", ["timestamp"])
    .index("by_replied", ["replied"])
    .index("by_threadId", ["threadId"]),

  replies: defineTable({
    inquiryId: v.id("inquiries"),
    threadId: v.string(),
    message: v.string(),
    timestamp: v.number(),
    sentVia: v.optional(v.array(v.string())),
  })
    .index("by_inquiryId", ["inquiryId"])
    .index("by_threadId", ["threadId"]),

  bookings: defineTable({
    inquiryId: v.optional(v.id("inquiries")),
    startMs: v.number(),                       // start-of-day UTC ms
    endMs: v.number(),                         // end-day INCLUSIVE
    customerName: v.string(),
    customerContact: v.string(),
    service: v.optional(v.string()),
    status: v.string(),                        // "confirmed" | "tentative" | "cancelled"
    notes: v.optional(v.string()),
    priceTotal: v.optional(v.number()),
    depositPaid: v.optional(v.boolean()),
    createdFrom: v.string(),                   // "inquiry" | "manual"
    createdAt: v.number(),
    updatedAt: v.optional(v.number()),
  })
    .index("by_startMs", ["startMs"])
    .index("by_status", ["status"])
    .index("by_endMs", ["endMs"]),

  // ============== Owner-facing ==============
  // siteContent: ONE blob per language. The frontend owns the shape so adding
  // a new editable section never requires a migration.
  siteContent: defineTable({
    lang: v.string(),                          // "en" | "zh" | "es" | etc.
    data: v.any(),                             // { hero, host, why, how, pricing, faq, gallery, ... }
    updatedAt: v.number(),
    updatedBy: v.optional(v.string()),
  }).index("by_lang", ["lang"]),

  // adminAuth: passwordless email login. Two doc kinds in one table so a single
  // sweepExpired pass covers both. Tokens carry a 4-char prefix so requireAuth
  // can dispatch without a DB lookup on the legacy raw-key fast path.
  adminAuth: defineTable({
    kind: v.union(v.literal("link"), v.literal("session")),
    token: v.string(),                         // lnk_<hex> or ses_<hex>
    email: v.string(),
    expiresAt: v.number(),
    createdAt: v.number(),
    consumedAt: v.optional(v.number()),        // set when a link is verified (single-use guard)
  })
    .index("by_token", ["token"])
    .index("by_expires", ["expiresAt"]),
});
```

### Why one CMS blob per language, not normalized tables

The naive instinct is `pricing_tiers`, `faq_items`, `hero_text` as separate tables with foreign keys. Resist it. Every wording change becomes a migration, every new section becomes a schema review, and the owner can't add "a quick note about holiday hours" without your help.

The `data: v.any()` blob trades schema enforcement for **shape ownership at the frontend**. The HTML is the schema — `data-cms="hero.eyebrow"` attributes name the slots the owner can edit. Adding a slot is a one-line HTML change. The cost — "wrong word on a webpage" — is the right cost to absorb.

### Why the auth table mixes link + session kinds

Sweep them together (`sweepExpired`), audit them together ("what auth events happened last week" is one query), and keep the schema small. If you outgrow this and want OAuth or multi-admin, you'll be doing a real auth migration anyway — don't pre-optimize.

---

## Phase 3 — The auth state machine (90 min — the hardest phase)

This is the part most likely to ship subtly broken. Build it carefully and test the unhappy paths.

### What you're building

```
[email form] → POST /api/auth/request-link →
                 ↓
                 if email in ADMIN_EMAIL_ALLOWLIST:
                   mint kind="link" with lnk_<hex>, 15-min TTL
                   Resend email to owner
                 always return 200 {ok:true} (no enumeration)
                 ↓
[Owner clicks link in email] → GET /api/auth/verify?token=lnk_… →
                                 ↓
                                 consumeLink mutation:
                                   - row exists? not consumedAt? not expired?
                                   - patch consumedAt = now
                                   - insert kind="session" with ses_<hex>, 30-day TTL
                                 302 → /inbox?key=ses_<hex>
                                 ↓
[/inbox magic-link bootstrap] → grabs ?key=, stores in localStorage, scrubs URL
                                 ↓
[Every admin request] → x-api-key: ses_<hex> →
                          ↓
                          requireAuth dual-mode:
                            1. raw ADMIN_API_KEY → ok (server scripts)
                            2. starts with "ses_" → POST Convex /admin-auth/validate-session
                            3. else → 401
```

### Convex side

Create `convex/admin/auth.ts` with `internalMutation`s for `createLink`, `consumeLink`, `revokeSession`, `revokeSessionById`, `revokeAllForEmail`, `sweepExpired`, plus an `internalQuery` for `validateSession`. Tokens generated with `crypto.getRandomValues(new Uint8Array(32))` then hex-encoded, prefixed `lnk_` or `ses_`.

Critical correctness gotcha: when revoking a session that the owner picked from a UI list, **revoke by `_id`**, not by token. The list response only carries `tokenSuffix` (last 6 hex chars) for privacy — if you have the UI synthesize a token from suffix and call `revokeSession`, Convex's exact-token lookup misses and you get a silent no-op that returns 200. The shape that works:

```ts
export const revokeSessionById = internalMutation({
  args: { sessionId: v.id("adminAuth"), scopeEmail: v.string() },
  handler: async (ctx, args) => {
    const row = await ctx.db.get(args.sessionId);
    if (!row || row.kind !== "session") return { ok: false, reason: "not_found" };
    if (row.email !== args.scopeEmail) return { ok: false, reason: "out_of_scope" };
    await ctx.db.delete(row._id);
    return { ok: true };
  },
});
```

The `scopeEmail` check is the one server-side gate that prevents a session caller from revoking someone else's session even with a guessed `_id`.

Expose all of these via `httpAction` routes in `convex/http.ts` under `/admin-auth/*`, **all admin-key gated** — Vercel is the only legitimate caller, the owner-facing tokens (`lnk_`, `ses_`) live in request bodies.

### Vercel side

Three Vercel routes at minimum:

- `POST /api/auth/request-link` — checks `ADMIN_EMAIL_ALLOWLIST`, mints link via Convex, sends Resend email. **Always returns 200** for any email shape — no enumeration.
- `GET /api/auth/verify?token=lnk_…` — exchanges link for session via Convex, 302 redirects to `/inbox?key=ses_…`. Set `Cache-Control: no-store` and `Referrer-Policy: no-referrer` on this response. Render a bilingual error HTML page (not a JSON 4xx) on failure — the user is in their email client, not a script.
- `GET/POST /api/auth/sessions` — list / revoke devices. The `requireAuth` here must capture the caller's email so revocations stay scoped.

The `requireAuth` helper goes in **every** admin Vercel route and must be `async`:

```js
async function requireAuth(req) {
  const sent = String(req.headers['x-api-key'] || '').trim();
  if (!sent) return false;
  const validKey = String(process.env.ADMIN_API_KEY || '').trim();
  if (validKey && sent === validKey) return true;       // server-side fast path
  if (sent.startsWith('ses_')) {
    try {
      const r = await fetch(`${CONVEX_SITE_URL}/admin-auth/validate-session`, {
        method: 'POST',
        headers: { 'content-type': 'application/json', 'x-api-key': validKey },
        body: JSON.stringify({ token: sent }),
      });
      if (!r.ok) return false;
      const j = await r.json().catch(() => ({}));
      return Boolean(j?.ok);
    } catch (_) {
      return false;                                     // fail-closed on Convex outage
    }
  }
  return false;
}
```

The `ses_` prefix check is what keeps the legacy raw-key path sync — only the magic-link path pays the Convex round-trip.

### Frontend side

The `/inbox` login overlay has the email field as the **primary** action and the raw-key password input collapsed under a `<details>` disclosure (only useful for dev / break-glass). The magic-link bootstrap script at the top of the page reads `?key=` from URL, stuffs it into `localStorage["pawpaw_inbox_key"]` (rename for your project), and scrubs the URL via `history.replaceState`.

For the public landing page edit-in-place FAB (Phase 4): never show the FAB based on localStorage alone. **Server-validate first** — call `/api/site-content?action=verify-key` with the stored token, only `.classList.add('show')` on 200. On 401, clear the localStorage key (it's stale or revoked).

### Test the unhappy paths before moving on

Mint a `lnk_` via the admin-key-gated Convex route, then verify all of these:

1. Click verify with valid `lnk_` → 302 to `/inbox?key=ses_…` ✓
2. Click verify with same `lnk_` again → bilingual error page (single-use enforcement) ✓
3. Probe `/api/site-content?action=verify-key` with the resulting `ses_` → 200 ✓
4. Probe with bogus `ses_` → 401 ✓
5. Revoke the session → re-probe → 401 ✓
6. `/api/auth/request-link` with a non-allowlist email → 200 `{queued:false}`, no email sent ✓
7. Race: two clicks on the same `lnk_` arriving in parallel → one succeeds, one returns `already_used` ✓ (Convex serializes mutations per-doc)

If any of these fail, fix it now — Phase 5 onward assumes they work.

---

## Phase 4 — Public landing page with edit-in-place CMS (90 min)

The shape: static HTML/CSS, hydrated with dynamic copy from `siteContent` on page load, owner edits in place when authenticated.

### Markup pattern

Every editable string carries a `data-cms` attribute naming its slot:

```html
<header>
  <p class="kicker" data-cms="hero.eyebrow">Castro Valley pet boarding</p>
  <h1 data-cms="hero.headline">Cage-free home for your dog while you're away.</h1>
  <p data-cms="hero.sub">$55/night, capped at 4 dogs, treated like family.</p>
</header>

<section class="pricing">
  <table>
    <tr>
      <td data-cms="pricing.0.service">Adult overnight</td>
      <td data-cms="pricing.0.note">any size</td>
      <td><strong data-cms="pricing.0.rate">$55</strong><span data-cms="pricing.0.suffix">/night</span></td>
    </tr>
    <!-- repeat for pricing.1, pricing.2, ... -->
  </table>
</section>
```

Slot naming uses dotted paths (`hero.eyebrow`, `pricing.0.rate`) that map onto the `siteContent.data` blob. The frontend's hydration script walks `[data-cms]` elements and resolves each path against the blob.

### Hydration script

```js
(async function () {
  const lang = (document.documentElement.lang || 'en').toLowerCase();
  const r = await fetch(`/api/site-content?lang=${lang}`).catch(() => null);
  if (!r || !r.ok) return;                              // static fallback wins
  const { data } = await r.json();
  if (!data) return;
  document.querySelectorAll('[data-cms]').forEach((el) => {
    const path = el.dataset.cms.split('.');
    let cursor = data;
    for (const k of path) {
      if (cursor == null) return;
      cursor = cursor[k];
    }
    if (typeof cursor === 'string') el.textContent = cursor;
  });
})();
```

Run this **once on load**, after the static markup parses. Edge-cache the GET endpoint for 60s — owner edits propagate within a minute, fast enough for any solo operator.

### Edit-in-place FAB

A floating button that appears only for authenticated owners. Click → flips every `[data-cms]` element to `contenteditable=true`, shows a save bar. Save POSTs the diff (or the whole blob — small payloads either way) to `/api/site-content`.

The visibility gate: server-side validate the localStorage key on every page load (described in Phase 3). Anything else — checking only that `localStorage` has *some* value — is a leak vector.

---

## Phase 5 — Owner inbox SPA (90 min)

A single-page app at `/inbox.html` with four tabs: Inquiries / Calendar / Stats / CMS Editor. Plus a sessions modal triggered by a 🚪 button in the header.

### Login overlay structure (post-Phase 3)

Email form is primary, password input is collapsed:

```html
<div class="login-overlay" id="login-overlay">
  <div class="login-card">
    <div class="logo">🐾</div>
    <h1>Your Brand</h1>
    <p>Sign in to admin</p>
    <input type="email" id="login-email-input" placeholder="owner@example.com">
    <button onclick="requestMagicLink()">📧 Send login link</button>
    <p id="login-magic-msg"></p>
    <details><summary>Advanced sign-in</summary>
      <input type="password" id="api-key-input" placeholder="Admin key">
      <button onclick="login()" style="background:#666">Sign in with key</button>
    </details>
  </div>
</div>
```

### Sessions modal

Triggered by 🚪. Lists active sessions with the current device tagged. Each row has an ✕ revoke button (passes Convex `_id`, not synthesized token). Footer has "Sign out this device" and "Sign out everywhere."

The current-device detection: compare last 6 chars of `localStorage` token against each row's `tokenSuffix`. The full token never leaves the server.

### Inquiry list pattern

For every inquiry from the customer's contact form (Phase 6), display:

- Customer name + contact (email or phone)
- The original message
- The translated message (if `detectedLang` differs from owner's reading language) — show both side-by-side
- Status (replied / pending) and timestamp
- Quick-action buttons (reply, mark replied, archive)

The "Assist" mode generates a reply suggestion via OpenRouter; "Autopilot" mode auto-sends after a delay. Default to Assist — solo operators want control.

---

## Phase 6 — Customer intake (45 min)

The contact form on the public landing pages POSTs to `/api/contact`. Three things happen on the backend:

1. **Detect language + translate** via OpenRouter. If the customer wrote in language A and the owner reads language B, store both.
2. **Persist to `inquiries`** via Convex.
3. **Email the owner** via Resend (subject includes detected language flag and a short summary).

### Translation prompt

A single OpenRouter call does detection + translation + summarization in one round-trip:

```
You are a translator for a small business owner.
The owner reads {OWNER_LANG}.
Translate the message below to {OWNER_LANG} if it isn't already, and detect its source language.
Return JSON: { "detectedLang": "<iso-639-1>", "translatedMessage": "<translation or original>", "summary": "<≤15 words>" }

Message: """{MESSAGE}"""
```

Chain a free fallback: `kimi-k2-0905` (cheap, strong at zh↔en) → `qwen/qwen3-max` → `deepseek/deepseek-chat-v3.1:free`. Wrap each in a 15s timeout.

### Resend payload

The new-inquiry email to the owner should be **dense and skim-friendly**:

```
Subject: 🐾 New inquiry from <name> [<detected-lang>]

<summary>

— Translated to <owner-lang> —
<translated message>

— Original (<detected-lang>) —
<original message>

Reply: <reply-link to /inbox?inquiryId=xxx>
Customer: <contact>
Submitted: <timestamp>
```

The "Reply" deep-link drops the owner straight into the inquiry's reply view — fewer clicks than navigating from the inbox tab.

---

## Phase 7 — Photo gallery (45 min)

Owner uploads photos from their phone, the public site shows the most recent N. Implementation:

```ts
// convex/gallery/store.ts
export const addPhoto = mutation({
  args: { storageId: v.id("_storage"), caption: v.optional(v.string()) },
  handler: async (ctx, args) => {
    const url = await ctx.storage.getUrl(args.storageId);
    if (!url) throw new Error("storage_not_found");
    // Add to siteContent.data.gallery for ALL languages — the gallery is shared
    for (const lang of ["en", "zh", "es"]) {
      const existing = await ctx.db.query("siteContent").withIndex("by_lang", (q) => q.eq("lang", lang)).first();
      const data = existing?.data || {};
      const gallery = Array.isArray(data.gallery) ? [...data.gallery] : [];
      gallery.unshift({ url, storageId: String(args.storageId), caption: args.caption || "", ts: Date.now() });
      if (gallery.length > 48) gallery.length = 48;     // cap; owner can ✕-remove via UI
      const newData = { ...data, gallery };
      if (existing) await ctx.db.patch(existing._id, { data: newData, updatedAt: Date.now() });
      else await ctx.db.insert("siteContent", { lang, data: newData, updatedAt: Date.now(), updatedBy: "gallery-add" });
    }
    return { ok: true, url };
  },
});
```

Client side: compress before upload (`max 1280px wide, JPEG quality 85`). Use Convex's `generateUploadUrl()` so the file goes directly to Convex storage — Vercel function never sees the bytes, sidestepping the 4.5MB function body limit.

For historical archive imports, ship a `scripts/bulk-upload-photos.py` (or `.mjs`) that walks a folder and runs the same three-step pipeline (upload-url → blob upload → commit).

### Public-side render

Hydrate the gallery by reading `data.gallery` in the same hydration call from Phase 4. Render the most recent 6 photos in a strip; "see all" links to a fuller view if you want one.

---

## Phase 8 — Multilingual structure (60 min)

Three files: `index.html` (English), `zh/index.html`, `es/index.html`. Same skeleton, **divergent content per language** (don't auto-translate; let the owner write each).

### What each file owns

- Identical `data-cms` namespace (`hero.eyebrow`, `pricing.0.rate`, ...)
- Same hydration script
- Different default text content (the static fallback when Convex is unreachable)
- Language switcher in the header that links to the other two

### Why divergent (not auto-translated)

The Chinese version of "Castro Valley pet boarding" might emphasize a Chinese-speaking owner; the Spanish version might emphasize Hayward & Castro Valley local Spanish-speaking community ties. Auto-translating English is technically possible but loses these. Let the owner write each — they know their customers.

### Adding a fourth language

1. Copy `zh/` → `vi/`
2. Replace static fallback content
3. Set `<html lang="vi">`
4. Convex `siteContent` accepts the new `lang` value automatically (no schema change)

---

## Phase 9 — Cron + cleanup (15 min)

`convex/crons.ts`:

```ts
import { cronJobs } from "convex/server";
import { internal } from "./_generated/api";

const crons = cronJobs();
crons.daily(
  "adminAuth: sweep expired tokens",
  { hourUTC: 9, minuteUTC: 0 },
  internal["admin/auth"].sweepExpired,
);
export default crons;
```

`sweepExpired` walks `adminAuth` `by_expires` index, deletes anything with `expiresAt < now`, capped at 200/call. In steady state you'll generate <10 expired rows/day; the cap exists for catastrophe recovery (mass invalidation, schema migration).

---

## Phase 10 — Deploy + verify (30 min)

```bash
# 1. Deploy Convex (registers schema, functions, http routes, crons)
CONVEX_DEPLOYMENT=prod:<your-deployment> npx convex deploy

# 2. Set Vercel env vars (Production + Preview):
#    ADMIN_API_KEY, ADMIN_EMAIL_ALLOWLIST, RESEND_API_KEY, OPENROUTER_API_KEY,
#    CONVEX_SITE_URL, PUBLIC_BASE_URL, MOM_EMAIL, OWNER_EMAIL, EMAIL_FROM
vercel env add ADMIN_API_KEY production    # repeat for each var

# 3. Deploy Vercel
npx vercel --prod
```

### Live verification protocol (do not skip)

After every deploy, **fetch the live URL and grep for a content signal** before saying "shipped." Build success ≠ deployed.

```bash
# 1. Public hydration works
curl -s https://<your-app>.vercel.app/ | grep -E 'data-cms|hero\.eyebrow' && echo OK

# 2. Auth verify-key with no header → 401
curl -s -o /dev/null -w "%{http_code}" -X POST https://<your-app>.vercel.app/api/site-content?action=verify-key
# expect: 401

# 3. Magic-link round-trip (use the admin key you just set)
TOKEN=$(curl -s https://<your-deployment>.convex.site/admin-auth/create-link \
  -H "x-api-key: $ADMIN_API_KEY" \
  -d '{"email":"<allowlisted-email>"}' | jq -r .token)
curl -s -i "https://<your-app>.vercel.app/api/auth/verify?token=$TOKEN" | head -3
# expect: HTTP/2 302  /  location: /inbox?key=ses_...
```

If any of those fail, the deploy is not done — even if Vercel shows green.

### Hand-off to the owner

The owner's complete sign-in instructions:

1. Go to `https://<your-app>.vercel.app/admin`
2. Type your email → click **Send login link**
3. Open your email, click the green button
4. Bookmark the page that loads (it'll be your inbox from now on)

If they get locked out: text you. You can either rotate `ADMIN_API_KEY` on Vercel (logs everyone out) or have them request a fresh link (preferred — that's the design).

---

## What this skill is NOT for

- **Multi-tenant SaaS.** One owner, one deployment. If you want N owners, you want a different shape (per-owner Convex deployments? RLS in shared deployment? both have tradeoffs).
- **Marketplaces with peer-to-peer trading.** "Many owners, many customers" is a real marketplace. `easier-to-deploy-storefronts` is "one + many," much simpler.
- **High-volume e-commerce.** Stripe + cart flow + inventory aren't in this skill. Add them as Phase 11+ if you need them.
- **Customer accounts.** Customers fill a form and get an email. Adding customer auth doubles the auth surface; if you need it, build it deliberately.

## What you can swap without rebuilding

- **Convex → Postgres + Hono.** Move `siteContent`, `inquiries`, `bookings`, `adminAuth` into Postgres tables. Rewrite `convex/http.ts` as Hono routes. Vercel layer doesn't change. ~1 day.
- **Resend → Postmark / SendGrid.** Single env-var change (`from:` address) + replace the API call helper. ~30 min.
- **OpenRouter → direct provider.** Bypass the router; call Anthropic / OpenAI directly. Lose the free fallback chain. ~30 min.
- **Vercel → Cloudflare Workers / Netlify.** Function shape is portable — wrappers differ but the logic is identical. ~half a day.

The lock-in is in your *integration code*, not the providers. That's the right level to absorb lock-in.

---

## Skill checklist (run before saying done)

- [ ] All five Convex tables in `schema.ts` with indexes
- [ ] `convex/admin/auth.ts` has `createLink`, `consumeLink`, `validateSession`, `revokeSessionById`, `revokeAllForEmail`, `sweepExpired`
- [ ] `convex/http.ts` registers admin-key-gated routes for every auth action
- [ ] `convex/crons.ts` schedules daily `sweepExpired`
- [ ] `api/auth/request-link.js` returns 200 for non-allowlist emails (no enumeration)
- [ ] `api/auth/verify.js` 302s on success, renders bilingual HTML on failure
- [ ] `api/auth/sessions.js` enforces email-scope server-side (Convex layer, not just Vercel)
- [ ] `requireAuth` is `async`, accepts both raw key and `ses_<hex>` tokens
- [ ] Public landing pages have `data-cms` attributes on every editable string
- [ ] FAB on landing pages does **server-side** key validation before showing
- [ ] `/inbox.html` login overlay has email primary, password under `<details>`
- [ ] `/inbox.html` sessions modal lists devices and revokes by `_id` (not synthesized token)
- [ ] `.env.example` documents every variable, defaults are fail-closed empty
- [ ] No hardcoded secrets / personal info anywhere outside `.env.example`
- [ ] Live verification: `curl` the deployed URL and grep for a content signal
- [ ] Owner can sign in via email magic-link from a fresh browser

If a checkbox is unchecked, the skill is incomplete — finish it before handoff.

---
> Source: [HomenShum/easier-to-deploy-storefronts](https://github.com/HomenShum/easier-to-deploy-storefronts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
