## change-map-size

> Apply when changing world dimensions; includes deep-sea ring and ambient edge checks

# Map Size Change Guide

Use this when changing world dimensions (for example `800x800` to `1000x1000`).

## Core Files to Update

1) `server/src/lib.rs`
```rust
pub const WORLD_WIDTH_TILES: u32 = NEW_SIZE;
pub const WORLD_HEIGHT_TILES: u32 = NEW_SIZE;
```

2) `client/src/config/gameConfig.ts`
```ts
const SERVER_WORLD_WIDTH_TILES = NEW_SIZE;
const SERVER_WORLD_HEIGHT_TILES = NEW_SIZE;
```

## Deep Sea / Edge-Zone Sync (Important)

Map-size changes affect edge behavior and ocean presentation. Verify these values remain intentional:

1) Server deep-sea perimeter width:
- `server/src/environment.rs`
- `DEEP_SEA_OUTER_RING_TILES`

2) Client fallback deep-sea edge width:
- `client/src/config/gameConfig.ts`
- `DEEP_SEA_EDGE_TILES`

These two should stay aligned unless you intentionally want different behavior.

## Why Deep Sea Matters

- World-edge ambient logic in `client/src/components/GameCanvas.tsx` uses `gameConfig.worldWidthPx` and `gameConfig.worldHeightPx`.
- Ambient selection in `client/src/hooks/useAmbientSounds.ts` uses `distanceToMapEdge` thresholds.
- Procedural fallback rendering and minimap deep-sea coloring can look wrong if edge constants are out of sync.

## Build / Republish Sequence

From project root:

```bash
spacetime build -p ./server
spacetime publish -c -p ./server broth-bullets-local
spacetime generate --lang typescript --out-dir ./client/src/generated -p ./server
```

Restart client after generation (`npm run dev` if needed).

## Validation Checklist

- [ ] Server and client world tile sizes match.
- [ ] Server `DEEP_SEA_OUTER_RING_TILES` and client `DEEP_SEA_EDGE_TILES` intentionally match.
- [ ] Player edge proximity sounds still feel correct (shore vs deep ocean near map edge).
- [ ] Minimap deep-sea ring looks expected.
- [ ] Ocean/deep-sea spawn behavior near perimeter still matches design intent.

## Common Pitfalls

- Updating only server or only client dimensions.
- Forgetting to regenerate bindings after server-side changes.
- Assuming deep-sea ring should stay fixed for every map size without re-evaluation.

---
> Source: [SeloSlav/2d-multiplayer-survival-mmorpg](https://github.com/SeloSlav/2d-multiplayer-survival-mmorpg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
