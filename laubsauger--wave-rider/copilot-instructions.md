## wave-rider

> Spec-driven: read `SPEC.md` (caveman format, see ck plugin FORMAT). §V invariants, §T tasks, §B bug ledger. ROADMAP.md = future scope. Never edit spec sections except via /ck:spec; /ck:build flips §T status only.

# wave-rider — project notes

Spec-driven: read `SPEC.md` (caveman format, see ck plugin FORMAT). §V invariants, §T tasks, §B bug ledger. ROADMAP.md = future scope. Never edit spec sections except via /ck:spec; /ck:build flips §T status only.

## Architecture
- Pure core in `src/lib/` (audio analysis, track gen, physics) — deterministic, vitest-covered. No three.js in physics; `sample.ts` bridges (frames: positions/tangents/normals/binormals/widths/walls per ~3m).
- Scene in `src/scene/` (r3f + three/webgpu + TSL). HUD = DOM via rAF reading `src/game/telemetry.ts` (mutable singleton, no React re-renders mid-race).
- Physics is track-space (s, d): `stepShip` fixed 120Hz via accumulator. Falling/airtime ride the `air`/`vy` fields.

## Multiplayer (naive v1)
- `src/lib/network/p2p.ts`: PeerJS 1:1 host/join, reliable DataConnection, `NetworkMessage` union. 10Hz `state_update` {s,d,v,yaw,finished} heartbeats (also during 'waiting' — that's the sync handshake).
- `RaceScene` MP: opponent state in `sim.current.opponent`; `NetworkShip` reads it via a `source()` closure per frame (NEVER gate mount on sim refs read at render time — B19) + dead-reckons between packets.
- Ghosts: `network/ghost.ts` records 10Hz frames, base64 in `?ghost=` URL.
- Known naive: no clock sync, no rollback, trust-based finish times, 2 players max.

## Gotchas
- WebGPU only; preview tab occlusion suspends rAF (B11 — never await rAF alone in pipelines).
- Audio files in `audio/` are gitignored; `bundled.ts` globs them; meta loads on hover intent only (35MB trap, T68).
- Determinism: no Math.random in gen/sim paths (V8); seeded mulberry32 only.

---
> Source: [laubsauger/wave-rider](https://github.com/laubsauger/wave-rider) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-13 -->
