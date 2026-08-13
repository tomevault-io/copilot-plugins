## portfolio-tracker

> After EVERY code change, BEFORE committing:

# CLAUDE.md — Rota App Development Rules

## MANDATORY POST-CHANGE VERIFICATION

After EVERY code change, BEFORE committing:

1. **TypeScript check**: `cd rota-app && npx tsc --noEmit` — 0 errors
2. **Verify script**: `cd rota-app && bash scripts/verify.sh` — ALL CHECKS PASSED
3. **API test**: `curl` the affected endpoint on production — verify response
4. **Visual test**: Take screenshot via Chrome MCP or ask user to verify on device
5. **Cross-reference**: Search the web for how the feature works in real apps (Google Maps, Strava, Instagram) — compare with our implementation
6. **Data flow check**: Trace the data from API response → mobile state → UI render. Verify field names match at every step.

## NEVER DO

- Never commit code without running verify.sh
- Never assume an API response shape without checking the actual backend route
- Never use fake/hardcoded coordinates — always verify with Google Maps
- Never add a feature without testing it end-to-end on production
- Never skip the visual check — if it's a UI change, SEE it working
- Never use emoji in Mapbox GL symbol layers — they don't render in WebView
- Never trust that "it should work" — TEST IT

## MAP RULES

- Mapbox GL JS in WebView: ONLY use circle layers + text symbol layers
- Emoji/Unicode in `text-field` DOES NOT RENDER in React Native WebView
- Use colored circles for markers, text labels for names
- Always test zoom in/out behavior after map changes
- Water filter removed — trust create-event reverse geocoding instead

## FIELD NAME CONSISTENCY

- Backend events: `meetingLat`, `meetingLng` (not `lat`, `lng`)
- Frontend map: normalize with `e.lat || e.meetingLat`
- Categories: `eventType` field contains new category names (spor, kosu, kafe, etc.)
- Always check `resolveCat()` in map.html handles the eventType

## TECH STACK

- Mobile: React Native + Expo Router + TypeScript
- Backend: Next.js App Router + Drizzle ORM + Neon PostgreSQL
- Map: Mapbox GL JS v3.4.0 via WebView bridge
- Deploy: Vercel (auto-deploy on push to main)
- Test: Expo Go on physical device (no browser preview for RN)

---
> Source: [Cugutayy/portfolio-tracker](https://github.com/Cugutayy/portfolio-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
