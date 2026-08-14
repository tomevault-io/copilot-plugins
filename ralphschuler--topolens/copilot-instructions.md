## topolens

> TopoLens is a mobile-ready network-engineering tool that visualizes topology changes observed from RIPE RIS Live. It is one static Vite + React + TypeScript application running entirely in the browser and deployed to GitHub Pages.

# TopoLens Copilot Instructions

## Project overview

TopoLens is a mobile-ready network-engineering tool that visualizes topology changes observed from RIPE RIS Live. It is one static Vite + React + TypeScript application running entirely in the browser and deployed to GitHub Pages.

Do not assume or add server-side services, persistence, container orchestration, or separate workspaces. All commands run at the repository root.

## Observation model

TopoLens has no initial RIB snapshot. Its state is valid only for the current browser session and selected RIS subscription:

1. A fetch worker subscribes to a RIPE route collector, optionally narrowed by prefix and AS path.
2. RIS envelopes are normalized into per-prefix announcements and withdrawals.
3. A stateful `RouteTracker` indexes routes by `(collector, peer, prefix)` and compares each update with the last locally observed state.
4. Bounded snapshots feed the change list, inspector, counters, and AS-path graph.
5. Graph computation stays off the React render path.

Never imply that the graph is a complete internet topology. Keep these meanings precise:

- `warming`: live observations are still establishing local baselines;
- `partial`: scope, start time, disconnect gaps, and retention limits make the view incomplete;
- `first-observed`: first announcement for a local route key, not proof of global route creation;
- `withdrawal-observed`: withdrawal without a route baseline, with no fabricated path or edge diff.

## Topology rules

- Preserve ordered ASN path segments and unordered AS sets.
- Build an AS edge only between adjacent, unambiguous ASN segments.
- Ignore AS-prepending self loops in the graph.
- Preserve the full normalized path for the inspector even when it yields no edge.
- Classify path and origin changes deterministically; do not double-count one transition.
- Identical announcements refresh state but are not topology changes.
- Known withdrawals remove retained routes; unknown withdrawals remain honest observations.
- Route and change retention must stay bounded, with eviction/drop counters exposed.
- Applying a new collector/prefix/path scope starts a clean route-tracking session.

## UX requirements

- Treat phone layouts as a primary target, not a scaled-down desktop afterthought.
- Avoid horizontal page overflow and keep the graph usable beneath overlays or drawers.
- Primary touch targets should be at least 44 × 44 CSS pixels.
- Every hover interaction needs a tap/click and keyboard equivalent.
- Preserve visible focus, semantic labels, status announcements, and explicit dismiss actions.
- Keep socket connection status distinct from observation completeness.
- Favor dense, inspectable technical data, but define potentially misleading routing terms nearby.
- Show errors and reconnection state without discarding already retained session data unless the user resets or changes scope.

## Development and quality

Use Node.js 22. Install and run all tooling from the root:

```bash
npm ci
npm run typecheck
npm run lint
npm run format
npm run test
npm run build
```

Use strict TypeScript and avoid `any` at protocol boundaries; validate `unknown` input instead. Prefer pure normalization and topology functions, deterministic fixtures, defensive snapshots, and explicit worker message types. Do not mutate objects owned by another layer.

Tests should cover protocol normalization, scope payloads, route transition classification, collector/peer isolation, exact edge diffs, AS sets, prepending, malformed data, known and unknown withdrawals, reconnect gaps, buffer limits, and user-visible responsive flows. Automated tests must not wait for an unpredictable public RIS update.

Generated `dist/` assets are build output and must not be edited manually.

## GitHub Pages

Keep production browser-only. Vite reads `BASE_PATH` during the Pages build. Any deployment change must preserve asset loading on both `/` and repository subpaths. GitHub Pages deployment is allowed only after root type checking, linting, formatting checks, tests, and build all succeed.

---
> Source: [ralphschuler/TopoLens](https://github.com/ralphschuler/TopoLens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
