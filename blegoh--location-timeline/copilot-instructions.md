## location-timeline

> This repository contains a privacy-first location-history visualizer. Users import a Google Maps Timeline JSON file and explore their journeys on an interactive map.

# AGENTS.md

## Project purpose

This repository contains a privacy-first location-history visualizer. Users import a Google Maps Timeline JSON file and explore their journeys on an interactive map.

Location history is extremely sensitive. Protecting it is a core system requirement.

## Product principles

1. Process imported files entirely in the browser.
2. Never transmit imported or derived location data.
3. Make privacy behavior explicit and verifiable.
4. Keep the initial workflow understandable without documentation.
5. Maintain an original visual identity.
6. Prefer correctness and clarity over adding more features.
7. Provide a fictional sample journey so users can evaluate the product safely.

## Working method

Before editing:

1. Inspect the repository structure and existing instructions.
2. Read the relevant implementation and tests.
3. Check the current working tree and preserve unrelated user changes.
4. Identify the smallest coherent change that satisfies the task.
5. State any assumption that materially affects behavior.

While editing:

- Keep changes scoped to the request.
- Follow existing conventions unless they conflict with this document.
- Do not perform broad rewrites without a concrete reason.
- Keep parsing, domain logic, map rendering, and UI state separated.
- Add or update tests alongside behavior changes.
- Use established dependencies before introducing new ones.
- Do not silently weaken privacy, accessibility, validation, or error handling.

After editing:

- Run formatting, linting, type checking, and relevant tests.
- Test the affected user flow.
- Review the final diff for unrelated changes.
- Report verification results and any known limitations.

## Technical baseline

Unless the repository already defines another stack, use:

- Next.js App Router
- React
- TypeScript in strict mode
- Tailwind CSS
- MapLibre GL JS
- Vitest
- Playwright

Do not migrate an established stack merely to match this baseline.

## Architecture

Keep these concerns independent:

- Source-format detection
- Timeline parsing
- Data normalization
- Geographic calculations
- Filtering
- Statistics
- Playback
- Map presentation
- User-interface state
- Export
- Privacy controls

Map components must consume normalized domain data. They must not understand every variation of Google’s source schemas.

New source formats should normally be implemented as adapters that produce the shared normalized model.

## Privacy and security rules

These rules are mandatory.

- Never upload imported files.
- Never send coordinates, timestamps, routes, filenames, or derived statistics to a server.
- Never include imported information in analytics, telemetry, crash reports, URLs, query parameters, or page titles.
- Never log raw imported records.
- Never store imported data in cookies, local storage, IndexedDB, caches, or service workers unless a user-facing requirement explicitly authorizes persistence.
- Do not use server actions, API routes, or remote parsing for imported data.
- Do not introduce third-party scripts without reviewing their privacy impact.
- Do not expose sensitive data through verbose errors.
- Do not use `eval` or execute content from imported files.
- Treat imported JSON as untrusted input.
- Validate coordinates, timestamps, array sizes, and nested structures.
- Prevent prototype-pollution patterns when transforming arbitrary objects.
- Remove in-memory dataset references when the user selects “Remove imported data.”
- Synthetic sample data must not resemble a real person’s precise location history.
- Map tiles will require ordinary network requests. Document this clearly and ensure imported coordinates are not deliberately added to tile-provider requests beyond normal tile selection behavior.

Any requested change that conflicts with these rules must be surfaced rather than implemented silently.

## Data handling

Use explicit TypeScript types for normalized data.

Parsers should:

- Detect their supported schema
- Validate required fields
- Tolerate optional-field variations
- Skip recoverable invalid records
- Collect structured warnings
- Fail clearly when no usable timeline remains
- Return normalized values
- Avoid mutating the source object

Clear or release raw-file references once normalization completes.

For large files, prefer streaming techniques or Web Workers where practical. Do not freeze the main interface during expensive parsing.

## Geographic correctness

- Validate latitude as `-90...90`.
- Validate longitude as `-180...180`.
- Use a documented geodesic distance method.
- Keep units explicit.
- Handle timestamps and time zones deliberately.
- Do not silently interpret missing time-zone information as local time unless this is documented.
- Treat implausible jumps as possible measurement errors rather than silently deleting them.
- Preserve enough information to explain ignored or repaired records.

## Map implementation

- Use MapLibre GeoJSON sources and layers for routes.
- Do not create thousands of DOM markers.
- Keep source updates controlled and measurable.
- Fit bounds only when appropriate; do not constantly override the user’s viewport.
- Include required map attribution.
- Provide non-color indicators or labels for transport modes.
- Ensure selected, hovered, edited, and playback states are distinguishable.
- Do not couple playback timing to React rerender frequency.

## React and state

- Keep imported dataset state centralized.
- Keep transient map state separate from normalized domain data.
- Derive statistics rather than duplicating them in mutable state.
- Memoize expensive transformations when measurement supports it.
- Cancel workers, timers, and animation frames during cleanup.
- Prevent stale parsing jobs from replacing newer imports.
- Avoid putting very large arrays into URL state or serialized server state.

## User experience

The main sequence is:

1. Understand the product.
2. Try sample data or import a file.
3. Resolve validation problems.
4. Explore and filter the map.
5. Play back or correct routes.
6. Export an image if desired.
7. Remove the imported data.

Requirements:

- Always provide loading, empty, success, and error states.
- Error messages must explain how to recover.
- Features unavailable before import should be hidden or clearly explained.
- Do not rely on disabled controls as the only explanation.
- Keep advanced controls collapsible.
- Confirm destructive in-app actions such as discarding unsaved edits when appropriate.
- Never imply that local edits modify the original JSON file.

## Accessibility

- Use semantic HTML before adding ARIA.
- Every input and button needs an accessible name.
- All workflows must be keyboard-accessible.
- Keep focus visible.
- Announce import progress and errors through an appropriate live region.
- Respect `prefers-reduced-motion`.
- Provide a non-animated alternative to playback.
- Prevent keyboard traps in the map.
- Use sufficient contrast.
- Do not encode information using color alone.
- Maintain practical touch-target sizes.

## Design rules

Create an original interface. Do not copy another product’s branding, exact layout, copywriting, icons, or distinctive visual assets.

Use shared design tokens for:

- Color
- Typography
- Spacing
- Borders
- Radius
- Shadows
- Motion

Favor a quiet cartographic appearance. The map should remain the visual focus after a dataset is loaded.

Desktop should use a sidebar-and-map layout. Mobile should prioritize the map and expose controls through an accessible sheet or drawer.

## Testing expectations

Every parser needs fixture-based tests.

Unit tests should cover:

- Schema detection
- Valid and invalid records
- Partial recovery
- Timestamp conversion
- Distance calculations
- Date filtering
- Mode aggregation
- Gap detection
- Outlier handling
- Playback interpolation

End-to-end coverage should include:

- Opening the fictional sample journey
- Importing a valid fixture
- Rejecting malformed JSON
- Handling an unsupported schema
- Filtering by date
- Playing, pausing, and scrubbing
- Editing a transport mode
- Resetting imported data
- Mobile navigation
- Keyboard navigation

Tests must use synthetic location data only. Never commit a user’s Timeline export.

## Performance expectations

- Avoid blocking the main thread during large imports.
- Avoid repeatedly cloning complete datasets.
- Update map sources incrementally or efficiently.
- Simplify route geometry when needed without corrupting statistics.
- Clean up animation frames, workers, subscriptions, and map instances.
- Measure before adding complex optimization.

## Dependency policy

Before adding a package:

1. Verify that existing code or platform APIs cannot reasonably solve the problem.
2. Check maintenance status and browser compatibility.
3. Review whether it introduces network calls or data collection.
4. Prefer small, focused dependencies.
5. Document important architectural dependencies.

Do not add analytics, advertising, session replay, or remote-error-reporting SDKs without explicit authorization and a privacy review.

## Commands and verification

Use the commands defined by the repository. Typical checks may include:

```sh
npm run format
npm run lint
npm run typecheck
npm test
npm run test:e2e
npm run build
```

Do not report a command as passing unless it was actually run successfully.

If a full test suite cannot run, execute the most relevant checks and clearly identify what remains unverified.

## Documentation

Update the README when behavior, setup, supported formats, architecture, or privacy characteristics change.

The README should explain:

- Local setup
- Development commands
- Supported input formats
- High-level architecture
- Privacy model
- Map and tile-provider network behavior
- Sample-data provenance
- Browser limitations
- Testing
- Known limitations

## Completion report

When finishing a task, report:

- The user-visible outcome
- Important implementation decisions
- Files or areas changed
- Tests and checks run
- Known limitations
- Privacy implications, when relevant

Do not declare completion while required tests are failing or while a known privacy regression remains.

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

---
> Source: [blegoh/Location-Timeline](https://github.com/blegoh/Location-Timeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
