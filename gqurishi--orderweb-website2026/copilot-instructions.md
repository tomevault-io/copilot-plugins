## orderweb-website2026

> This is the OrderWeb marketing website (TanStack Start + React + Vite).

# OrderWeb Website — Agent Notes

This is the OrderWeb marketing website (TanStack Start + React + Vite).

- Keep the branch in a working state before pushing.
- Prefer npm scripts from `package.json` (`dev`, `build`, `preview`).
- Do not force-push or rewrite published git history unless explicitly requested.

## Admin CMS (Phase 0 locked)

- Admin URL: `/owadmin` — never add to public SiteNav / footer.
- Login: email + password; optional TOTP authenticator MFA (Account → Authenticator).
- Save behaviour: **Draft → Preview → Publish** (Revert to last published).
- Editable: words, images, links, prices, SEO, contact email settings.
- Not editable via admin: layout, fonts, animations, code.
- Stack: **Supabase** (Postgres) + **Cloudinary** (images) + **SMTP** → `mail@orderweb.co.uk`.
- Phase 1 pages: Home, About, Pricing, Contact.
- Phase 2 pages: Restaurant POS (`/restaurant-pos`), Website, Software.
- Legal / help pages (CMS): Privacy (`/privacy`), Terms (`/terms`), FAQ (`/faq`), DPA (`/dpa`).
- Phase 3: draft/publish, media meta, users/roles, activity log, SEO, hardening.
- Phase 4 extras: change password (`/owadmin/account`), forgot-password email reset, analytics snippets in Settings. Skipped: multi-language, blog, schedule publish, duplicate section.
- Full phase map: `docs/ADMIN_ROADMAP.md`. Training: `docs/HOW_TO_UPDATE_ORDERWEB.md`.
- Setup checklist: `docs/ADMIN_PHASE0_SETUP.md`
- Locked constants: `src/lib/admin/decisions.ts`
- Secrets: use `.env` from `.env.example` — never commit `.env` or service-role / API secrets.
- Admin: `/owadmin` (login, dashboard, page editors, media, social media, users, activity, account, settings).
- Social Media (`/owadmin/social`): Facebook / Instagram / YouTube / X URLs → footer icons (+ home Organization `sameAs`). Blank = hide icon.
- Content driver: local `.data/cms-db.json`. Supabase SQL ready in `supabase/schema.sql`.
- Bootstrap login: `VITE_ADMIN_EMAIL` + `ADMIN_BOOTSTRAP_PASSWORD` (creates first admin on first successful login).
- Users: max **2** active admins (same full Admin role — no Editor).
- Contact email: Settings → SMTP; inbox default `mail@orderweb.co.uk`. Env overrides: `SMTP_*`.
- Password reset emails need SMTP + `SITE_URL` (production origin).
- Analytics/SEO in Settings: GA4, GTM, Search Console, Bing, Meta Pixel, Clarity (+ custom head HTML).
- Phase 1 docs: `docs/ADMIN_PHASE1.md`, QA: `docs/ADMIN_PHASE1_QA.md`.
- Phase 2 docs: `docs/ADMIN_PHASE2.md`, QA: `docs/ADMIN_PHASE2_QA.md`.
- Phase 3 docs: `docs/ADMIN_PHASE3.md`, QA: `docs/ADMIN_PHASE3_QA.md`.
- Phase 4 docs: `docs/ADMIN_PHASE4.md`, QA: `docs/ADMIN_PHASE4_QA.md`.
- Backups: daily copy of `.data/cms-db.json` + `public/cms-uploads/`.

---
> Source: [gqurishi/orderweb-website2026](https://github.com/gqurishi/orderweb-website2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
