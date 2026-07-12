## gymly-app

> Gymly is a **multi-tenant SaaS gym management platform**. Every gym is an isolated

# AGENTS.md — Gymly Security & Safety Rules
# Project: gymly-app-06 · Stack: React 19 + Vite 5 + Firebase 12 + Tailwind 3
# Last updated: 2026-06-04
# READ THIS ENTIRE FILE BEFORE WRITING ANY CODE OR RULE

---

## 1. IDENTITY OF THIS PROJECT

Gymly is a **multi-tenant SaaS gym management platform**. Every gym is an isolated
tenant. A security mistake in one gym's data can expose every other gym's data.
There are currently live gym owners and members on production. No change may break
existing functionality or cause data loss.

Firebase project ID: `gymly-app-06`
Frontend deploy: Vercel (gymly.online)
Functions runtime: Node.js 18, CommonJS (.js), NOT TypeScript

---

## 2. NON-NEGOTIABLE RULES — NEVER VIOLATE THESE

These rules apply to every file, every prompt, every diff. No exceptions.

### 2.1 Firestore data model constraints

```
RULE: Staff and members are created with addDoc() — their Firestore doc IDs are
      auto-generated random strings. They are NOT equal to Firebase Auth UIDs.
      Only gym owner documents use setDoc(doc(db,'users',uid)) where uid matches Auth.

CONSEQUENCE: Never write Firestore rules or queries that assume
             doc.id == request.auth.uid for staff or member documents.
             Use gym_id scoping instead.

RULE: Membership plans are embedded arrays in gym.settings.plans.
      Do NOT create a separate /plans collection.

RULE: profile_photo must always be a Firebase Storage download URL string.
      Never store base64 image data in Firestore documents.
      Firestore document size limit is 1MB — base64 photos exceed this.

RULE: Never store undefined values in Firestore. Use null.

RULE: gym_id must be validated server-side on all sensitive collection writes.
      Client-supplied gymId cannot be trusted without a server-side ownership check.
```

### 2.2 Authentication and mock mode

```
RULE: The mockRole localStorage bypass MUST ONLY exist in development builds.
      Every mockRole check in any file must be wrapped in:
        if (import.meta.env.DEV) { ... }
      Vite tree-shakes this in production. Verify by checking that import.meta.env.DEV
      is the outermost condition before any localStorage.getItem('mockRole') call.

RULE: Never add new mockRole checks to any file without the DEV guard.

RULE: onAuthStateChanged must NEVER be inside a DEV guard.
      It must remain unconditional. If it is guarded, all production users are locked out.

AFFECTED FILES:
  src/context/AuthContext.jsx         — mockRole block wrapped in DEV guard ✓ DONE
  src/firebase/firestore.js           — isMock() prefixed with DEV guard ✓ DONE
  src/firebase/firestore-payments.js  — isMock() prefixed with DEV guard ✓ DONE
```

### 2.3 Firestore security rules

```
RULE: Deny by default. Every collection must have an explicit rule.
      Never rely on implicit denial — always write allow rules explicitly.

RULE: The /users collection read rule must ALWAYS require:
        request.auth != null
      AND either:
        request.auth.uid == uid  (own document)
      OR:
        caller's gym_id == resource's gym_id  (same-gym staff access)
      Public unauthenticated reads of /users are PERMANENTLY FORBIDDEN.

RULE: The /users collection update rule must ALWAYS block:
        'role' in request.resource.data
        'gym_id' in request.resource.data
      Role and gym_id are server-managed fields. No client write may set them.

RULE: The /users collection delete rule must ALWAYS be:
        allow delete: if false;
      Member deletion goes through the softDeleteMember Cloud Function only.

RULE: /gyms writes must require request.auth.uid == resource.data.owner_id.
      No cross-gym gym document writes are ever permitted.

RULE: /payments, /invoices, /attendance_logs, /whatsapp_logs, /invoice_counter,
      /message_retry_queue, /message_logs, /numbering_settings, /serial_counters
      must ALL have gym_id ownership checks. Use:
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.gym_id
          == resource.data.gym_id

RULE: /used_coupons allow write: if false — all writes go through Cloud Function.

RULE: /coupons allow read, write: if false — server-only via Admin SDK.

RULE: /deleted_members/{gymId}/bin/{memberId}
        allow read: owner and manager in same gym only
        allow write: if false — Cloud Function only

RULE: /audit_logs/{gymId}/events/{eventId}
        allow read: owner and manager in same gym only
        allow write: if false — append-only via Cloud Function, NEVER from client

RULE: /leads allow create with strict field validation only:
        keys must be subset of [name, phone, email, gym_id, created_at, message]
        name and phone must be strings under 100 characters

RULE: kiosk_devices allow update only if device_secret in request matches stored value.
      attendance_sessions writes must validate device_secret against paired kiosk doc.
      NEVER allow update: if true on kiosk_devices.
      NEVER allow read, write: if true on attendance_sessions.

RULE: After every firestore.rules change, run:
        npx firebase-tools rules:check --project gymly-app-06
      NEVER deploy rules that fail the check.
```

### 2.4 Cloud Functions

```
RULE: All Cloud Functions are CommonJS (.js) files.
      The functions directory has NO TypeScript build pipeline.
      Never create .ts files in functions/src/ without first adding tsconfig.json
      and a tsc build step to functions/package.json.

RULE: Every HTTPS callable function must verify context.auth first:
        if (!context.auth) throw new functions.https.HttpsError('unauthenticated', ...)

RULE: Every function that writes to a gym's data must re-read the caller's
      gym_id and role from Firestore server-side. Never trust client-supplied role.

RULE: Financial writes (payments, subscriptions, coupon redemption) must NEVER
      be written directly from the client to Firestore.
      They must always go through a Cloud Function that re-computes amounts
      server-side from the gym's plan data.

RULE: Use db.batch() when a function performs 3 or more Firestore writes that
      must all succeed or all fail together. Never use separate awaits for
      writes that are logically atomic.

RULE: When using batch.set() for a document that needs an auto-generated ID,
      use db.collection(...).doc() with no arguments to generate the ref,
      then batch.set(ref, data). Firestore batch does NOT support .add().

RULE: Scheduled functions that use collectionGroup() queries MUST have the
      corresponding index in firestore.indexes.json with queryScope: COLLECTION_GROUP.
      Deploy the index BEFORE deploying the function.

RULE: Use if (!admin.apps.length) admin.initializeApp() at the top of every
      new functions/src/*.js file.

RULE: New functions must be exported from functions/index.js. Add at the bottom:
        const module = require('./src/filename');
        exports.functionName = module.functionName;
      Never move or modify existing exports in index.js.
```

### 2.5 Frontend security

```
RULE: firebase-admin must NEVER appear in the root package.json dependencies.
      It is a Node.js server package. It belongs only in functions/package.json.

RULE: No coupon codes, API secrets, or service account credentials may appear
      in any file inside src/. These belong in Firestore (server-read only)
      or in environment variables never shipped to the client.

RULE: GYMLY_COUPONS.txt must never be committed to version control.
      Add it to .gitignore if not already present.

RULE: vercel.json must include a headers block with at minimum:
        Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
        X-Frame-Options: DENY
        X-Content-Type-Options: nosniff
        Referrer-Policy: strict-origin-when-cross-origin
        Content-Security-Policy: (scoped to self + Firebase + reCAPTCHA domains)

RULE: console.error and console.warn in src/firebase/config.js must be wrapped
      in if (import.meta.env.DEV) so they are removed from production bundles.

RULE: File uploads must validate MIME type client-side before sending to Storage:
        if (!file.type.startsWith('image/')) throw error
      Storage rules must enforce: request.resource.contentType.matches('image/.*')
      and request.resource.size < 5 * 1024 * 1024 (5MB limit).
```

### 2.6 Data safety — no data loss

```
RULE: NEVER run firebase firestore:delete on any collection in production.

RULE: NEVER restructure or rename existing Firestore collections.
      All schema changes must be additive (new fields, new collections).

RULE: NEVER delete a /users document directly from the client or from a
      migration script. Use softDeleteMember Cloud Function.

RULE: Before any bulk migration script runs:
        gcloud firestore export gs://gymly-app-06-backups/$(date +%Y%m%d)
      Verify the export completed before running the migration.

RULE: Migration scripts that modify existing documents must be idempotent.
      Running them twice must produce the same result as running them once.

RULE: The base64 photo migration (migrateBase64Photos function) is safe to
      run live — it only replaces base64 strings with Storage URLs.
      The worst case if it fails mid-run is that some docs still have base64.
      It does not delete any document.
```

---

## 3. CURRENT SECURITY STATUS (as of 2026-06-04)

### 3.1 Fixed and deployed ✅

| ID | Issue | Fix |
|----|-------|-----|
| C-1 | Role escalation via /users update | Rules: uid ownership + role/gym_id field block deployed |
| C-2 | mockRole bypass in production | DEV guard in AuthContext.jsx, firestore.js, firestore-payments.js |
| C-3 | Coupon codes in JS bundle | redeemCoupon Cloud Function, gymlyCodesData.js deleted |
| H-2 | /users publicly readable | Auth + gym-scoped read rule deployed |
| Trainer fix | Landing page query broke after H-2 | auth.currentUser guard in GymLandingPage.jsx |
| Delete | Client-side deleteDoc on /users | softDeleteMember + restoreMember + permanentlyDeleteExpired deployed |

### 3.2 Pending — work in order ⏳

| Priority | ID | Issue | Blocker |
|----------|----|-------|---------|
| 🔴 NOW | C-3 seed | /coupons collection empty — coupon activation broken in prod | Run seedCoupons.js after firebase-tools login + ADC credentials |
| 🔴 NOW | C-3 deploy | redeemCoupon function not deployed | firebase deploy --only functions after seeding |
| 🟡 TODAY | M-2 | /used_coupons still client-writable | Add allow write: if false to rules — one line |
| 🟡 TODAY | H-6 | No CSP/HSTS/X-Frame-Options in vercel.json | Add headers block to vercel.json |
| 🟠 DAY 2 | H-3 | /gyms writable by any authenticated user | Add owner_id ownership check to rules |
| 🟠 DAY 2 | H-4 | 9 collections have no gym isolation | Add gym_id ownership checks — one rules deploy |
| 🟠 DAY 2 | H-5 | Kiosk + attendance fully open | device_secret validation in rules |
| 🟠 DAY 2 | H-7 | Storage wildcard cross-gym file access | Scope storage.rules by gymId |
| 🟠 DAY 3 | H-1 | No Firebase App Check | initializeAppCheck in config.js + Console enforcement |
| 🟡 DAY 3 | M-1 | firebase-admin in client package.json | Remove from root, confirm in functions/ only |
| 🟡 DAY 3 | M-3 | Admin gate client-side only | Check /admins/{uid} server-side in AdminDashboard.jsx |
| ⚪ DAY 5 | L-1 | html2canvas XSS surface | Audit usage, replace with jsPDF native APIs |
| ⚪ DAY 5 | L-2 | /leads open spam writes | Add field validation to create rule |
| ⚪ DAY 5 | L-3 | console.error leaks in production | Wrap in import.meta.env.DEV |

---

## 4. KNOWN ARCHITECTURE DECISIONS

```
DECISION: Firestore proxy pattern exists for dev/demo mode.
  src/firebase/firestore.js checks isMock() and routes to either
  firestore_real.js (production) or mockFirestore.js (dev).
  The isMock() check is DEV-guarded. Never remove this guard.

DECISION: Member and staff documents use addDoc() — auto-generated IDs.
  Only owner documents use the Firebase Auth UID as the doc ID.
  All rules and queries must account for this.

DECISION: Plans are embedded in gym.settings.plans as an array.
  Do not create a /plans collection. Updates write the full array.

DECISION: Kiosk routes (/kiosk/entry, /kiosk/exit) are intentionally
  unauthenticated at the Firebase Auth level. Security is enforced via
  device_secret token validation in Firestore rules.

DECISION: Vite is pinned at 5.x. Do NOT upgrade to Vite 6+ or 8+.
  Vite 8's rolldown bundler breaks Firebase 12 ESM imports.
  Check Firebase 12 compatibility before any Vite version bump.

DECISION: Firestore is initialized with persistentLocalCache for offline
  support on gym reception tablets with poor connectivity.

DECISION: Member deletion is soft-only from the client.
  Hard deletion only occurs via permanentlyDeleteExpired scheduled function
  after 30 days. Audit log is written for every delete and restore.
```

---

## 5. DEPLOY CHECKLIST

Run through this before every production deploy.

### Firestore rules deploy
```
[ ] Run: npx firebase-tools rules:check --project gymly-app-06
[ ] Zero errors in rules:check output
[ ] Deploy: firebase deploy --only firestore:rules --project gymly-app-06
[ ] Verify in Firebase Console → Firestore → Rules that new rules are live
[ ] Run Rules Playground tests for any changed collection
```

### Cloud Functions deploy
```
[ ] If new collectionGroup() query added: deploy indexes FIRST
      firebase deploy --only firestore:indexes --project gymly-app-06
[ ] Wait for index green checkmark in Firebase Console → Firestore → Indexes
[ ] Deploy functions: firebase deploy --only functions --project gymly-app-06
[ ] Verify all expected function names appear in deploy output
```

### Frontend deploy
```
[ ] Run: npm run build — must complete with zero errors
[ ] Chunk size warnings are acceptable — do not block on them
[ ] git push to main — Vercel auto-deploys
[ ] Verify Vercel deploy succeeded in Vercel dashboard
[ ] Test login as owner on production URL after deploy
```

### Emergency rollback
```
[ ] Rules rollback: revert firestore.rules in git, re-deploy --only firestore:rules
[ ] Functions rollback: firebase functions:delete <functionName> --project gymly-app-06
[ ] Frontend rollback: Vercel dashboard → Deployments → Redeploy previous deployment
```

---

## 6. SENSITIVE FILES — NEVER COMMIT THESE

```
functions/.env
functions/serviceAccount.json
GYMLY_COUPONS.txt
.env.local
.env.production
Any file containing GOOGLE_APPLICATION_CREDENTIALS path or contents
Any file containing a Firebase private key
```

If any of these are detected in a diff, STOP and do not apply the change.

---

## 7. AGENT BEHAVIOUR RULES

When working as an AI agent on this codebase:

```
BEFORE writing any code:
  1. Read this file completely
  2. Check the current security status table (Section 3.2)
  3. Confirm the task does not violate any rule in Section 2

BEFORE modifying firestore.rules:
  1. Read the entire current firestore.rules file
  2. Show the exact diff — old lines with -, new lines with +, line numbers
  3. Wait for explicit confirmation before applying
  4. Run rules:check after applying, before deploying

BEFORE modifying any file in functions/src/:
  1. Read functions/index.js to understand export pattern
  2. Read functions/src/invoicing.js to understand CJS conventions
  3. Write .js not .ts
  4. Include the admin.apps.length guard
  5. Add exports to functions/index.js

BEFORE deploying:
  1. Follow the deploy checklist in Section 5 in order
  2. Indexes before functions — never the other way around
  3. Rules before frontend — the UI depends on rules being live first

NEVER do these without explicit written confirmation from the project owner:
  - Delete any Firestore collection or document in production
  - Rename any collection
  - Deploy a migration that modifies more than 100 existing documents
  - Change the Firestore cache mode
  - Upgrade firebase, vite, or react major versions

IF a diff would break an existing feature:
  1. Stop and flag the breakage clearly before applying
  2. Provide a fix for the breakage in the same diff
  3. Never ship a known breakage

IF any write involves role, gym_id, subscription_valid_until, or owner_id:
  1. It must go through a Cloud Function with server-side validation
  2. Never write these fields from client-side Firestore SDK calls
```

---

## 8. QUICK REFERENCE — COLLECTION OWNERSHIP PATTERN

Use this exact pattern for gym-scoped Firestore rules:

```javascript
// Standard gym-scoped read rule for any collection
allow read: if request.auth != null
  && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.gym_id
     == resource.data.gym_id;

// Standard gym-scoped write rule for any collection
allow write: if request.auth != null
  && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.gym_id
     == resource.data.gym_id
  && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role
     in ['owner', 'manager'];

// Server-only collection (Cloud Function writes only)
allow read, write: if false;
```

---

*This file is the single source of truth for security and safety rules on the Gymly project.
Any agent, developer, or Claude Code session working on this codebase must read and follow it.*

---
> Source: [gravitycomedia-droid/gymly-app](https://github.com/gravitycomedia-droid/gymly-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
