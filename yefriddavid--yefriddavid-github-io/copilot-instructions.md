## yefriddavid-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm start          # Dev server at http://localhost:3000
npm run build      # Production build (outputs to /build)
npm run serve      # Preview production build
npm run lint       # ESLint on src/**/*.js
npm run deploy     # Build + deploy to GitHub Pages
```

## Architecture

**React 18 SPA** using Vite, HashRouter, and CoreUI as the component framework. Built as a financial cash-flow management dashboard.

### State Management
Redux Toolkit + Redux-Saga + redux-act pattern:
- `src/actions/` — Action creators (redux-act style)
- `src/reducers/` — State slices (login, account, payment, paymentVaucher)
- `src/sagas/` — Async side effects via redux-saga
- `src/store/store.js` — Store config wiring it all together

Data flows: Component → dispatch action → saga intercepts → API call → reducer update → component re-renders.

### Routing & Auth
- **HashRouter** — all URLs are hash-based (`#/path`)
- **Auth guard** in `src/components/AppContent.js` — uses Firebase Auth `onAuthStateChanged`; shows spinner while resolving, redirects to `/login` when signed out
- Route definitions in `src/routes.js`; sidebar nav config in `src/_nav.js`

#### Firebase Authentication (refresh token)
Auth is handled by **Firebase Auth (email/password provider)** — no separate backend needed.

- **Service**: `src/services/auth/firebaseAuth.js`
- **Email convention**: users sign in with username; Firebase Auth account uses `${username}@cashflow.app` as synthetic email
- **Refresh token**: Firebase SDK stores it in IndexedDB automatically — session survives page reloads with no manual handling
- **Hybrid / lazy migration**: on first login, if the user has no Firebase Auth account yet, the system verifies via the legacy Firestore `passwordHash`, then auto-creates the Firebase Auth account transparently
- **Key functions**:
  - `signIn(username, password)` — hybrid login (Firebase Auth → legacy fallback → auto-migrate)
  - `signOut()` — Firebase signOut + clears localStorage
  - `getToken()` — returns a fresh ID token (auto-refreshes if expired)
  - `forceTokenRefresh()` — forces network refresh; call after a 401 response
  - `onAuthChange(cb)` — subscribes to auth state changes (used in AppContent)
  - `changePassword(username, currentPw, newPw)` — re-authenticates then updates password
- **Required Firebase console step**: enable **Email/Password** under Authentication → Sign-in method

#### Firestore Middleware
All Firestore operations should go through `src/services/providers/firebase/firebaseClient.js`:

```js
import { firestoreCall } from 'src/services/providers/firebase/firebaseClient'
const snap = await firestoreCall(() => getDocs(q))
```

- Refreshes the Firebase ID token before each call
- On `permission-denied` / `unauthenticated` → signs out and redirects to `/login`
- Retries transient errors (`unavailable`, `deadline-exceeded`) with exponential backoff
- Normalizes all Firebase error codes to Spanish user messages

### Backend / API
Dual backend:
1. **Google Apps Script** (primary) — POST requests with FormData (`action`, `token`, params). Base URL configured in `src/services/providers/api/utilApi.js`. A fresh Firebase ID token is injected automatically into every request via an axios request interceptor. On 401, the interceptor forces a token refresh and retries once.
2. **Firebase Firestore** — Primary data store for all modules. Config in `src/services/providers/firebase/settings.js`. Exports: `db` (Firestore), `auth` (Firebase Auth), `messaging` (FCM).

### Layout
`DefaultLayout` (`src/layout/DefaultLayout.js`) wraps all authenticated pages with `AppSidebar + AppHeader + AppContent + AppFooter`. Public pages (login, register, 404) render outside this layout.

### Styling
SCSS with CoreUI variables. Custom overrides in `src/scss/_custom.scss` and `src/scss/_variables.scss`. Prettier enforces: no semicolons, single quotes, trailing commas, 100-char line width, 2-space indent.

### i18n
`i18next` with Spanish (`es`) as default. Translation files loaded via HTTP backend. Use `useTranslation()` hook and `t('key')` in components.

### Key Libraries
- **DevExtreme / DevExpress** — DataGrid (`Accounts.js`) and reporting views
- **Chart.js + CoreUI Charts** — Dashboard charts
- **React Hook Form** — Form handling
- **Moment.js** — Date manipulation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yefriddavid) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
