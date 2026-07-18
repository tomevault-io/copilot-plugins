## ritual-engine

> You are working on Ritual Engine, a white-label toolkit that turns

# CLAUDE.md : Ritual Engine

You are working on Ritual Engine, a white-label toolkit that turns
cultural sites into walkable ceremonies: NFC/QR/GPS/compass-triggered
web experiences organized by cosmological structures. Read PLAN.md for
the product definition, domain model (section 4), file map (section 5),
and the current build phase.

## Standing conventions (never relax these)

1. **No em-dashes, ever.** Not in code comments, copy, docs, or commit
   messages. Use a colon, a comma, or a middle dot. The lint script
   enforces this; do not disable it.
2. **Config-driven, always.** Client-specific behavior lives in
   sites/<client>/site.json and tokens.json only. If a feature requires
   touching engine code per client, redesign the feature.
3. **No app, no install, no accounts.** Everything must run in a stock
   mobile browser over https. If a capability needs more, it does not
   ship.
4. **Every gate has studio mode.** All triggers (gps, compass, time)
   render a visible "studio mode: continue anyway" bypass. Removing it
   is a per-site curatorial decision made by a human, not a default.
5. **Graceful degradation is part of the artwork.** No camera, no
   sensor, desktop, denied permission: each case shows the content in a
   reduced but dignified form with copy that treats the visitor as a
   guest, not an error.
6. **Copy is bilingual-ready.** All visitor-facing strings go through
   the locale table in site.json; never hardcode display text in the
   engine. English fallback is acceptable, hardcoding is not.
7. **Privacy floor.** localStorage for journey state only. Optional
   Supabase analytics stores station id + timestamp, nothing else. No
   fingerprinting, no third-party trackers, ever. Participation
   submissions render only where approved = true.
8. **Design tokens only.** Engine CSS uses CSS variables exclusively,
   loaded from the site's tokens.json. Default theme: #0b0b0b ground,
   Space Grotesk display/body, JetBrains Mono labels, terracotta
   #c1440e accent. Uppercase letter-spaced mono for section labels.
9. **Reduced motion respected** on every animation, including the
   constellation map. Focus states visible on all interactive elements.
10. **Cultural material is load-bearing.** Obangsaek colors, mansion
    names, hangul glyphs, and Vietnamese copy are content, not
    decoration. When uncertain about a cultural detail, leave a TODO
    for the studio rather than guessing.

## Technical frame

- Vite + TypeScript, multi-page (enter.html, station.html, journey.html).
- AR layers: A-Frame 1.5.0 + MindAR 1.2.5 via CDN, loaded only on pages
  that need them.
- Deploy: GitHub Actions to Pages from dist/. Never commit dist/.
- One shared interface for triggers (src/triggers) and one for layers
  (src/layers); new modules implement the interface, nothing else
  changes.
- Test with: npm run dev, then phone on the deployed preview for
  anything involving camera, sensors, or NFC. Sensor code paths must
  handle both iOS (webkitCompassHeading, DeviceOrientationEvent
  .requestPermission) and Android (deviceorientationabsolute).

## Working style

- Verify each phase's acceptance criteria from PLAN.md before declaring
  a phase done; show the verification, not just the claim.
- Prefer deleting code to adding options.
- Commit messages: present tense, one line, what changed and why.
- When a decision trades visitor experience against developer
  convenience, the visitor wins.

---
> Source: [binyoun/ritual-engine](https://github.com/binyoun/ritual-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
