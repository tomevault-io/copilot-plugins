## super-llmario

> CURRENT ARCHITECTURE:

CURRENT ARCHITECTURE:
Use the recipe-cartridge split.

The engine owns durable functionality:

* input actions and keyboard/gamepad polling
* gamepad Learn UI and saved mappings
* pause/status-bar menu
* audio context / RetroCharm SFX path
* physics, collision, camera, update/render loop
* entity behavior: enemies, powerups, blocks, portal, player states
* recipe dispatch
* reusable drawing primitives and prefabs
* HUD behavior, overlays, panels, attributions UI
* external cartridge loading/normalization/validation

The cartridge owns reproducible/changeable world content:

* title, HUD label text, messages
* world dimensions and player spawn
* platform layout
* floating blocks and block contents
* coins, stars, enemies, signs, portal
* theme data
* drawing recipes
* optional audio overrides
* optional resource/attribution records
* optional decorative "resourceScenery"

The boundary is practical, not purist. The engine can include rich drawing functions/prefabs such as mushroom mascot, blob, portal, coin, star, grass platform, rounded block, HUD panel, parallax sky, glow, particles, oval, capsule, roundedRect, swirl, trees, mushrooms, pipes, arches, etc. The cartridge feeds custom data into those functions. Avoid putting custom "drawPortal()"-style functions in normal cartridges unless intentionally making a trusted custom/hack cartridge.

EXTERNAL CARTRIDGES:
External cartridges use JSON and should normally be named:

something.llmcart.txt

Android file pickers may only expose ".txt" reliably, so ".llmcart.txt" is the current working convention.

Expected cart format:
format: "llmcart-toybox-recipe"
cartVersion: 2

PLACEMENT / CLEANUP CONTRACT:
Generated or draft cartridges should prefer the normal, managed placement path. The loader runs a conservative one-time cleanup pass on external cartridges, after JSON parse and before gameplay reset. Its job is to prevent recurring draft mistakes, not to redesign a finished level.

* Surface props: resourceScenery pipes, trees, shrubs, mushrooms, and arrow signs are treated as surface-bound unless they declare anchor:"raw". The normalizer finds the closest valid walking surface near the prop and places the prop base cleanly on that surface.
* Pipes marked solid:true rebuild their collision from the corrected pipe base.
* Surface props that cannot be placed on a nearby surface are dropped rather than left floating over a chasm. Use anchor:"raw" for intentional weird/floating decorative placement.
* Floating blocks are binary: either their bottom is snapped to the walking surface as a grounded/static block, or their underside is high enough for the big hero plus padding to walk underneath. The forbidden middle zone is normalized away to prevent small-hero-only crawl spaces and big-hero teleport/shove bugs.
* Final hand-authored carts can opt out of all placement cleanup with placement:{autoCleanup:false}; individual objects can opt out with anchor:"raw".

The loader should:

* load from the pause menu
* parse JSON
* validate/normalize expected level fields
* reset gameplay state after loading
* replace world arrays from the external cart
* inherit built-in recipes/audio/resources unless overridden
* avoid leaking demo-only decorative "resourceScenery" into loaded carts unless the loaded cart provides its own
* keep the engine durable; cartridges should not need to understand the whole engine

CURRENT INPUT STATE:
Keyboard:

* arrows / WASD movement
* Space / Up / W jump
* Shift / Z / X run
* P / Escape / Enter pause
* R restart

Default gamepad mapping:

* Left: Button 14
* Right: Button 15
* Up: Button 12
* Down: Button 13
* Run: Button 0
* Jump: Button 1
* Pause/Start: Button 9

Gamepad Learn lives in the pause/status-bar menu. The visible always-on gamepad icon was removed. Tapping/clicking the top status bar opens pause/settings.

CURRENT AUDIO STATE:
RetroCharm SFX are integrated and working. The engine has one shared audio path intended to support future theme music/MIDI work later.

Current built-in SFX hooks include:

* jump
* coin
* star
* bonk
* mushroom emerge
* powerup collect
* enemy stomp
* hurt/shrink/life loss
* level clear

The audio system is intentionally ready for future music, but full MIDI parser/drop-folder/song scheduling is not part of the current baseline.

CURRENT GAMEPLAY STATE:
Current built-in cartridge is the Physics Test Plains / Jump Test style level. It is a known-good gameplay baseline with:

* working run/jump/fall tuning
* standing and running jumps that feel fair
* mushroom powerup block
* big/small player loop
* enemies, stomps, shrink/hurt/lives
* coins and stars
* bonk blocks and messages
* moving/bobbing floating blocks
* portal win condition
* level clear overlay
* signs and speech bubbles
* counters/HUD updating correctly without overlap

Current physics tuning has been validated by play feel. Avoid casual changes to jump/fall/run constants unless deliberately doing a physics pass.

CURRENT VISUAL STATE:
The current hero is a derivative mushroom-cap mascot. It is much stronger than the previous round/helmet/jellybean-headed mascot. Preserve the basic silhouette unless intentionally redesigning.

Grass/dirt rendering lesson:
Grass should not read like a green jellybean sitting on top of dirt. It should overlap/drape onto the dirt like hair over soil. Avoid continuous bright horizontal seam lines at the top of the dirt. If adding new grass details, preserve continuity between the grass cap/lip and the dirt body.

EPS / VECTOR SOURCE WORKFLOW:
EPS files are useful as vector-ish source material, not as runtime assets. Use them to derive:

* silhouettes
* proportions
* color cues
* decorative motifs
* reusable canvas prefab ideas

Do not import giant Illustrator path soup directly into runtime. Do not rely on JPG tracing when a vector source exists. Translate useful source ideas into canvas math / engine prefabs / cartridge recipes.

Attributions live in the pause menu as a pager: one card visible at a time, with previous/next and close controls. Keep it small enough to fit inside the artificial game-screen frame.

IMPORTANT WORKFLOW:
Start from the latest actual file on disk, patch it with code tools, then verify before linking. Extract the script contents and run "node --check" on it. When possible, render a screenshot/headless preview to ensure it is not blank. If screenshot/headless rendering is blocked by the sandbox, say so honestly. Never claim a file exists until it has actually been written and verified.

Use the latest ".htmlz" as the source of truth. Do not reconstruct from memory if the file is available.

**Some historical traces**

v68 added first-pass local-row slope platforms (up/down); v69 fixes the slope-to-flat handoff seam on steeper ramps while keeping v67 bonk-box behavior; v70 fixes the flat-to-slope seam jump teleport; v71 fixes steep slope-to-flat walking/running exit pause; v72 guards bonk boxes against seam-jump side-overlap false bonks; v74 extends the true-underside rule to solid image sprites and rectangular platforms; v75 adds cartridge-toggled bustable/breakable floating blocks with shard animation; v76 makes mushroom powerups row-aware, adds a simple emergence phase, ignores the source block briefly, and lets mushrooms ride existing slope platforms; v77 supports the split source model: load .llmtheme.txt and .llmmap.txt as two documents, combining them in memory without turning maps back into base64 asset carts; v78 makes theme-truth image assets draw as their original sprites while collision truth handles physics, including line/slope truth.

v79 stops demo resourceScenery trees/shrubs/mushrooms from leaking into loaded maps.

v83 adds first-pass global ladder/climb-zone physics: maps can place ladder/vine climbables with engine fallback art or theme skins; up/down climbs, jump detaches.
v84 smooths ladder seams by merging/treating touching climb-zone pieces as one climb column, so stacked ladder/vine pieces do not clamp-stop at joins.

v85 adds global pipe/tube support as an engine noun: maps can place solid green fallback pipes, themes can still skin later, and optional JSON targets can make down-press pipe travel.

v67 keeps v66 behavior and makes the existing bonk block vocabulary cartridge/theme-friendly: floating blocks may declare boxKind/kind, payload/contents, usedStyle/usedRecipe/usedImage, oneShot info behavior, and imageBlock/usedBlock recipes. Reward boxes can emit coin bursts or the existing mushroom powerup without hardcoding theme art. Floating blocks may also opt into busting with bustable:true/breakable:true and optional requiresBig:true.

v66 adds optional solid image sprites for asset-backed cartridges. Resource scenery sprites marked solid:true now participate in collision using their rendered image rectangle, so platform-looking PNG/SVG pieces do not accidentally behave like background decoration.

v65 keeps the row-camera latched seam as the first successful row transition implementation: falling across a row boundary latches the target row, gives a short readable seam pause, pans the camera, then releases the fall without old/new row bounce.

v63 adds an optional row-aware camera experiment for carts that define world.rows, including a brief fall-stall while the camera pans down to the next row. Carts without rows should keep the v62 side-scroller camera behavior.

v62 tightens the image-cartridge hooks after the first Jungle/BrownNature visual test: image tile platforms can paint a backing color and overdraw seams so transparent tile gutters do not reveal sky, and themed sign recipes get readable light text without mutating the classic physics-test level.

v60 adds a conservative load-time placement normalizer for external cartridges: surface props such as pipes/trees/shrubs/mushrooms/arrow signs are snapped to nearby walking surfaces, impossible scenery over chasms is dropped unless marked anchor:"raw", solid pipe collision follows the corrected pipe base, and floating blocks are forced into either ground-block placement or big-hero-safe overhead clearance.

v59 fixes the final 1-1 scenery nit: the later tree near x≈3080/3208 now has its base raised so the trunk reads above the grass/ledge. Rest of World 1 is intended to live as external .llmcart cartridge files, not embedded in the engine.

v58 raises the two too-low second-half block pairs so the hero can pass underneath, adds a tap-to-read map coordinate probe in the lower corner, shifts the arrow triangles right so their visual weight is centered, and drops the first decorative mushroom onto the grass.

v57 fixes the sign/mushroom polish nibs and extends built-in 1-1 into a longer second half with extra platforms, coins, enemies, scenery, and a later portal.

v56 fixes the follow-up polish misses: arrow signs now point right, the decorative red mushroom recipe gets a visible stem, and the hero ground shadow no longer follows jumps.

v55 refines the built-in 1-1 cart presentation: removes the confusing starting pipe, gives arrow signs a more balanced double-chevron face, and lifts the small red mushroom so it reads as a mushroom instead of a speckled rock.

v54 is the v53 merged baseline plus built-in 1-1 scenery cleanup: pipes/arches anchored to platforms, pipes optionally solid, and shrubs moved into cartridge data.

v53 is the merged baseline containing:

* v46 milestone systems: SFX/audio path, gamepad Learn, pause/status-bar menu, mobile display fixes, working counters/HUD, no onscreen mobile buttons.
* v52 visual systems: mushroom-cap hero, improved grass/dirt blend, attribution pager, EPS-derived scenery/prefab math.
* v53 external cartridge loader: Load Cartridge in pause menu; supports ".llmcart.txt", ".llmcart", ".json", ".txt".

PROJECT DIRECTION:
Retro charming does not mean technically primitive. Do not force era-accurate 1987 pixel-art limitations. Aim for toybox / N64-ish / modern-retro charm: gradients, rounded shapes, soft lighting, particles, squash/stretch, parallax, glow, richer tiles, expressive backgrounds, and modern UI polish, while keeping chunky silhouettes, readable sprites, side-scroller composition, tile-world logic, limited-ish palettes, and playful iconography.

This is a personal/offline-friendly weird game/art/tool project.

---
> Source: [thesepeoplearenotyourfriends/super_llmario](https://github.com/thesepeoplearenotyourfriends/super_llmario) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-03 -->
