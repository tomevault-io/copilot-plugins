## nodeform

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Development server with Turbopack
npm run build    # Production build
npm start        # Start production server
npm run lint     # ESLint via next lint
```

There are no tests in this project.

## Architecture Overview

**NodeForm / SurveyFlow** is a visual survey builder with conditional flow. Workspace owners (admins) build surveys in a React Flow canvas; respondents fill them out via a separate runtime experience.

### Stack

- **Next.js 15** App Router with Turbopack, **React 19**
- **Firebase Firestore** (via Admin SDK, no ORM) as the sole database
- **Firebase Storage** for file uploads
- **NextAuth v4** for admin authentication (JWT strategy)
- **Zustand** for client state; `@xyflow/react` for the visual editor
- **Stripe** for subscriptions and addons
- **OpenAI** (`gpt-4o-mini`) for AI survey generation
- **Resend** for transactional email
- **ShadCN/UI** + Tailwind CSS; i18n via a custom React context (`pt` default, `en`)

### Two distinct user types

| Type | Auth mechanism | Session |
|---|---|---|
| **Admin** (workspace owner) | NextAuth — Google OAuth or e-mail OTP | JWT cookie `next-auth.session-token` |
| **Respondent** (survey taker) | Custom OTP or SSO token | httpOnly cookie `respondent_session` (24 h) |

Workspace identity is the admin's `userId` — the same ID is used as `workspaceId` everywhere.

### Firestore collections

| Collection | Description |
|---|---|
| `users` | Admin accounts; subscription status, addons, AI credits |
| `surveys` | Survey documents (nodes + edges as JSON arrays) |
| `surveys/{id}/responses` | Subcollection; one doc per completed response |
| `respondents` | Respondent accounts, scoped to `workspaceId` |
| `respondentSessions` | Token → respondent mapping, 24 h TTL |
| `surveyParticipations` | Tracks who participated in which survey; bonus status |
| `surveyProgress` | Partial save; doc ID `{respondentId}_{surveyId}` |
| `workspaceApiKeys` | API keys stored as SHA-256 hash; prefix `nfk_` |
| `ssoTokens` | One-time SSO tokens; 5 min TTL, single-use |

### Survey data model

Surveys are stored as React Flow graphs: `nodes: SurveyNode[]` and `edges: SurveyEdge[]`. Node types: `presentation`, `singleChoice`, `multipleChoice`, `rating`, `textInput`, `endScreen`.

- The first node at runtime is the one with **no incoming edges**.
- `singleChoice` edges are per-option (`edge.data.optionId`); all other types use a generic `source` handle.
- Eligibility rules (`EligibilityRule[]`) can be set at the survey level (blocks ineligible respondents entirely) or on individual nodes (those nodes are auto-skipped at runtime).
- All eligibility rules use **AND logic** (`evaluateEligibility` in [lib/utils/eligibility.ts](lib/utils/eligibility.ts)).

### Access control flow

1. **Middleware** ([middleware.ts](middleware.ts)) protects `/dashboard/:path*`, `/editor/:path*`, `/onboarding`, `/upgrade`. It handles the onboarding gate and passes `x-pathname` as a header.
2. **Subscription gate** lives in [app/dashboard/layout.tsx](app/dashboard/layout.tsx) — it reads Firestore directly (not the JWT) to check `subscriptionStatus` or `trialEnd`. The path `/dashboard/settings` is exempt.
3. **API routes** use `resolveWorkspace()` ([lib/services/resolve-workspace.ts](lib/services/resolve-workspace.ts)), which accepts either a `Bearer nfk_...` API key or a NextAuth session.

### Addons system

Two purchasable addons stored under `users.addons`:

- **`respondents`**: unlocks respondent management, SSO, API key creation, participation tracking.
- **`surveyProgress`**: unlocks saving and restoring partial survey progress.

Check addon status via `hasAddon(workspaceId, addonId)` ([lib/services/addons.ts](lib/services/addons.ts)).

### SSO flow (server-to-server)

External platforms call `POST /api/sso/token` with a `Bearer nfk_...` API key and respondent data. NodeForm returns a one-time token (5 min TTL). The client then redirects the user to `/survey/[id]?sso_token=[token]`, which exchanges it for a respondent session cookie.

The survey page (`app/survey/[id]/page.tsx`) detects `?sso_token=` on load, calls `GET /api/respondent/auth/sso?token=...&surveyId=...` client-side, removes the token from the URL via `history.replaceState`, and starts the survey authenticated — the respondent never sees a login screen.

**Embed / iframe mode:** Add `?embed=true` to the survey URL to suppress the header and enable cross-origin cookie support. In embed mode the auth route sets `SameSite=None; Secure` on the `respondent-session` cookie instead of `lax`, which is required for the cookie to be accepted inside a cross-origin iframe. `SameSite=None` requires HTTPS — it will not work on plain HTTP (localhost). The SSO token URL for embed: `/survey/[id]?sso_token=[token]&embed=true`.

If the SSO token is invalid or expired, the page falls back to `checkRespondentAuth` — which either finds an existing session cookie or shows the manual OTP login gate.

### AI credits

10 free credits per month per workspace (auto-reset on the 1st of each month). Consumed transactionally in [lib/credits.ts](lib/credits.ts) before each call to `POST /api/surveys/generate`.

### Client-side state

| Store | Persistence | Purpose |
|---|---|---|
| `useEditorStore` | `localStorage` (`surveyflow-editor-storage`) | Editor state: nodes, edges, survey metadata |
| `useRuntimeStore` | None (in-memory) | Survey answering session: current node, answers, score |

### i18n

`useI18n()` returns `{ t, locale, setLocale }`. Locale preference is persisted to `localStorage` key `surveyflow-locale`. All UI strings should come from `lib/i18n/pt.ts` / `lib/i18n/en.ts`.

### Environment variables

See [.env.example](.env.example) for all required variables: `NEXTAUTH_*`, `GOOGLE_CLIENT_*`, `FIREBASE_*`, `STRIPE_*`, `OPENAI_API_KEY`, `NEXT_PUBLIC_APP_URL`.

The Firebase private key in `.env` uses literal `\n` escapes; [lib/firebase-admin.ts](lib/firebase-admin.ts) replaces them with real newlines on init.

---
> Source: [diogopaesdev/nodeform](https://github.com/diogopaesdev/nodeform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
