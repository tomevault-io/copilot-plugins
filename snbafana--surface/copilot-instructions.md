## surface

> Before implementing code changes, first ask what existing code or abstraction can be reused or deduplicated. Inspect the current implementation for overlapping responsibility before adding new files, types, helpers, daemons, runtimes, registries, or wrappers. Prefer extending or moving the existing owner over creating a parallel system, and call out the dedup decision briefly before editing.

# Surface Agent Instructions

Before implementing code changes, first ask what existing code or abstraction can be reused or deduplicated. Inspect the current implementation for overlapping responsibility before adding new files, types, helpers, daemons, runtimes, registries, or wrappers. Prefer extending or moving the existing owner over creating a parallel system, and call out the dedup decision briefly before editing.

## Plugin UI Iteration Loop

Every plugin UI should be iterated through the block preview harness before relying on the full overlay.

1. Render one plugin block with a deterministic fixture:
   ```bash
   swift run block-preview quicksave --fixture notes-and-captures --size 420x520
   ```
2. Render the full plugin fixture set:
   ```bash
   swift run block-preview all --output .build/block-previews
   ```
3. Inspect the PNGs in `.build/block-previews/`.
4. Run the preview smoke tests:
   ```bash
   swift test --filter BlockPreviewTests
   ```
5. Patch the plugin view or fixture, then rerun the single-plugin preview command.

The preview harness must use the real `Block` / `BlockRuntime.makeView()` path. Do not create a second preview-only UI for a plugin unless the real runtime cannot be made deterministic through `Block.Context`.

Evidence this loop works in this repo:

- `BlockPreview.renderAll` renders every current plugin fixture through `Blocks.registry`.
- `BlockPreviewTests` verifies that every current plugin has preview fixture coverage and that rendered PNGs are nonblank.
- The expected fast loop is one plugin at a time: render PNG, inspect, patch, rerender, then run the smoke test.

---
> Source: [snbafana/surface](https://github.com/snbafana/surface) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
