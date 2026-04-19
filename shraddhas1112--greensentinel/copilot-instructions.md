## greensentinel

> Agricultural monitoring PWA for Indian farmers. Production app targeting ₹40L investor grade.

# GreenSentinel — Claude Standing Instructions

## Project Context
Agricultural monitoring PWA for Indian farmers. Production app targeting ₹40L investor grade.
Stack: React + TypeScript + Vite + Tailwind + Framer Motion | AWS CDK + Lambda + Bedrock | S3 + CloudFront

---

## DEPLOY — Always Follow This Process

**Never run ad-hoc `aws s3 sync` or `aws s3 cp` commands to deploy.**
Always use the deploy script which enforces correct Cache-Control headers.

```bash
cd frontend
npm run deploy
```

This runs `deploy.sh` which does:
1. `npm run build`
2. Upload `dist/assets/` → `Cache-Control: public, max-age=31536000, immutable`
3. Upload `dist/` (entry points) → `Cache-Control: no-cache, no-store, must-revalidate`
4. CloudFront invalidation on `E7IN6ZTW0EXV5`

**Why:** If `index.html` is cached by the browser, users never receive new SW/JS. The two-step upload is mandatory every single deploy.

**Profile:** `green-sentinel` | **Region:** `ap-south-1`
**Bucket:** `green-sentinel-dev-frontend-938881281454`
**CloudFront:** `E7IN6ZTW0EXV5`

---

## DATA INTEGRITY — Never Fabricate Values

Do not add any hardcoded, estimated, or assumed values in the UI without a real data source:
- No hardcoded satellite pass timings (Sentinel-2 revisit is ~5 days, not hours)
- No estimated savings figures (e.g. ₹6,000 × alerts)
- No derived "AI scores" using arbitrary formulas (e.g. `95 - alerts * 5`)
- If real data is unavailable, show `--` or omit the metric entirely

---

## AI LABELLING — Only Label Real AI

Only use "AI" in labels/copy where AWS Bedrock (Claude Vision) is actually called:
- **Yes:** Disease Scanner (Bedrock), Threat Detection Camera (Bedrock)
- **No:** NDVI score, health score, weather forecast, alert counts

---

## MOBILE FIRST

All UI changes must work at 375px viewport width without horizontal scroll.
- Tooltips: use `createPortal` into `document.body` with `getBoundingClientRect()` clamping
- No fixed-width elements wider than `calc(100vw - 2rem)`

---

## BUILD CHECK

Always run `npm run build` inside `frontend/` after any code change and confirm it passes before finishing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ShraddhaS1112) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
