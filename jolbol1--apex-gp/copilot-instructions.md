## apex-gp

> A AAA-quality Formula 1 racing game in three.js. Everything — geometry,

# APEX GP — working notes

A AAA-quality Formula 1 racing game in three.js. Everything — geometry,
textures, sky, audio — is generated procedurally in code. **There are no asset
files and no network fetches; the game must run fully offline from source.**

Read **`CONTRACT.md`** before touching anything: it defines the module map, the
public API of every subsystem, coordinate conventions, the procedural-texture
API and the capture contract. It is the interface other agents rely on.

---

## Commands

### Build (must always pass)
```bash
cd /Users/work/Projects/YouTube/2026-07-29/f1
npx vite build
```

### Dev server
```bash
npx vite --port 5200 --strictPort
# wait for it:
for i in $(seq 1 40); do curl -sf -o /dev/null http://localhost:5200 && break; sleep 0.5; done
```
Use **your own port** — never reuse another agent's.

### Capture a deterministic frame
```bash
node tools/shot.mjs --out shots/chase.png --shot chase \
     --w 1600 --h 900 --warm 90 --url http://localhost:5200
```
`shot.mjs` prints page/console errors — read them. A black or blank image means
the page threw. **Always open the PNG and judge it with your own eyes.**

Shot names: `chase cockpit tv beauty wide grid wheel front hud`

Capture all nine:
```bash
for s in chase cockpit tv beauty wide grid wheel front hud; do
  node tools/shot.mjs --out shots/$s.png --shot $s --w 1600 --h 900 --warm 90 \
       --url http://localhost:5200 | grep -E 'WROTE|error'
done
```

### Debug flags (query string)
```
?fx=0                     bypass the whole post stack (raw render)
?ao=0&bloom=0&look=0      toggle individual passes (also dof, smaa, preGrade)
?env=0.6                  scale IBL intensity
?exposure=1.2             renderer.toneMappingExposure
?fog=0                    disable fog
```
Isolating a pass this way is the fastest route to "why is my frame white".

### In-game
| key | action |
|---|---|
| `W`/`↑` `S`/`↓` | throttle / brake |
| `A`/`←` `D`/`→` | steer |
| `E` / `Q` | shift up / down |
| `F` | DRS |
| `R` | ERS deploy |
| `C` | cycle camera |
| `X` | reset to the racing line |
| `Tab` (hold) | fps / draw calls / triangles / sim & render ms |

With no input the player car is driven by the same AI as the field (attract
mode) — which is also what makes the captures reproducible.

Gamepads are supported (left stick steer, triggers throttle/brake, bumpers shift).

---

## House rules

- **No `Math.random` outside `src/core/rng.js`.** Seed everything from
  `makeRng()` / `rngFor('name')` so captures are byte-reproducible.
- **No network, no asset files.** If you need a texture, add a generator to
  `src/textures/procedural.js` and document it in `CONTRACT.md`.
- **Go through `assets`** (`src/core/assets.js`) for anything shared between
  modules, so it bakes once and disposes as a unit.
- **Only edit the files you own.** If a file you don't own is broken and blocks
  you, wait 20 s and rebuild — another agent is probably mid-edit. Report it
  rather than fixing it.
- **Winding matters.** `loft()` and the ribbon builders assume
  `(ring tangent) × (row direction)` points out of the surface, and
  `right × tangent = +up` for ground ribbons. Inverted winding produces
  invisible geometry with `FrontSide` materials — it has bitten this codebase
  once already.
- **Bloom threshold is in linear scene-referred units.** Sky radiance ≈ 1–1.5,
  sun disc = 30. Never drop the threshold below ~2 or the whole frame hazes.
- **Particle sizes are metres**, not pixels (`fx.setViewport` converts).
- Keep the performance budget in `CONTRACT.md` §6 in mind: instance repeated
  objects, merge static geometry per material, no per-frame allocations.

## Known rough edges (good places to deepen)

- Chassis silhouette is correct in proportion but coarse in detail: the
  sidepod undercut, floor edge, front-wing cascades and rear-wing endplate
  louvres all deserve real geometry.
- Shadows use one tight cascade around the player; distant objects get AO+IBL
  only. `Lighting.setCascade()` is the seam for a true CSM.
- No SSR/planar reflections yet — the wet-track look leans on roughness + IBL.
- The crowd is a single instanced quad per spectator with no animation.
- Audio is synthesised but not spatialised for opponents (no PannerNode graph
  per car yet).

---
> Source: [jolbol1/apex-gp](https://github.com/jolbol1/apex-gp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-05 -->
