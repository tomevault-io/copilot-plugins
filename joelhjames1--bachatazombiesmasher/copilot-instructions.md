## bachatazombiesmasher

> iOS SpriteKit zombie side-scroller with a bachata theme. Portrait main menu,

# BachataZombieSmasher — Claude working notes

iOS SpriteKit zombie side-scroller with a bachata theme. Portrait main menu,
landscape gameplay.

## Hard rules

- **Do NOT edit `ZombieSmasher/Scenes/MainMenuScene.swift`.** The user has
  approved the menu and pushed back hard on iterating. If a fix is truly
  unavoidable, revert from `docs/MAIN_MENU_GOOD_STATE.md` rather than tuning
  new values.
- **Don't recreate `Resources/_originals/` or backup folders inside the
  Resources group.** Xcode's filesystem-synchronized group will bundle them
  and produce "Multiple commands produce" errors. Backups live at
  `_resource_originals/` and `_resource_centered_originals/` (siblings of
  `ZombieSmasher/`, gitignored).
- **Don't push to `main` without being asked.** The remote is
  `https://github.com/JoelHJames1/BachataZombieSmasher.git`.

## Project layout

```
ZombieSmasher/
  Entities/      Player, Zombie, Pickup, Projectile, WeaponKind, PhysicsCategory
  Managers/      AssetCatalog, SaveManager, SpawnDirector, SpriteSheetSlicer
  Scenes/        MainMenuScene (LOCKED), LevelSelectScene, GameScene, GameOverScene
  UI/            HUD, Joystick
  Resources/     PNG assets — auto-bundled via PBXFileSystemSynchronizedRootGroup
  GameViewController.swift  AppDelegate.swift
docs/            MAIN_MENU_GOOD_STATE.md — canonical menu snapshot
scripts/         strip_white_bg.py, repack_logo_sheet.py, center_animation_frames.py, strip_logo_white.py
```

## Orientation flow

`GameViewController.preferredOrientations` is the single source of truth
(default `.portrait`). `requestOrientationUpdate(_:)` flips it and triggers
iOS rotation. Scene transitions register a `pendingTransition` closure that
fires from `viewWillTransition(to:with:)` so the next scene is created with
post-rotation bounds.

- Menu / level select / game over → `.portrait`
- Game → `.landscape`

## Asset pipeline

The bachata logo sprite sheet (`MainLogoBachataAnimatonSpriteSheet.png`) had
non-uniform cells. Run from project root after replacing source:

```
cp ~/Desktop/Assets/<asset>.png ZombieSmasher/Resources/<asset>.png
cp ~/Desktop/Assets/<asset>.png _resource_originals/<asset>.png
rm -f _resource_centered_originals/<asset>.png
python3 scripts/strip_white_bg.py        # only if source has white bg
python3 scripts/repack_logo_sheet.py     # only the menu logo
python3 scripts/center_animation_frames.py
```

`repack_logo_sheet.py` adds 30px padding around each cell. This is why menu
logo sizing must scale by **height** (`maxH = size.height * 0.32`) rather
than width — keeps the sprite stable when source asset cell aspect changes.

`center_animation_frames.py` aligns each frame to the **average centroid**
across all frames in the sheet, not the geometric center, so per-frame
shifts stay sub-pixel and feet aren't clipped.

## Gameplay invariants

- `groundY = -size.height * 0.38`. Physics ground edge at this y.
- Player and Zombie `anchorPoint = (0.5, 0.62)` so visible feet land near
  `groundY` (the painted top of `GroundLevel1.png`).
- Two parallax layers per level: bg (`Level1.png`, parallax 0.3) and ground
  (`GroundLevel1.png`, parallax 1.0 if defined). Each has 2 copies that
  wrap. Add new ground assets via `AssetCatalog.levelGround(_:)`.
- Player has 5 states: `idle, still, run, jump, attack, dead`. After
  movement stops, `still` for 5s then transitions to `idle` (bachata dance).
- Pickup drops on zombie death: 10% grenade, 45% random weapon, 45% nothing.
- Equipping changes the walking sprite sheet via `playerWalkFrames(weapon:)`.
- Grenade has `contactTestBitMask = .zombie` and detonates on impact.

## Build / test

Open `ZombieSmasher.xcodeproj` in Xcode and run on iPhone simulator
(landscape will rotate after entering a level). The project uses Xcode 16+
filesystem-synchronized groups, so dropping new files into `Resources/`
auto-bundles them.

If pre-build diagnostics show "Cannot find type 'Player' in scope", that's
SourceKit failing to traverse the synchronized groups — `xcodebuild` itself
will compile fine.

## Debug / iterate dial

| Symptom | Adjust |
|---|---|
| Character feet sink into ground | raise `anchorPoint.y` (e.g. 0.62 → 0.68) |
| Character feet float above ground | lower `anchorPoint.y` (e.g. 0.62 → 0.55) |
| Background scrolls weirdly | check `parallaxLayers` widths and `parallax` factor |
| Logo too big / small | only scale by `maxH = size.height * X`. **Don't** edit if not asked. |
| Zombies too aggressive | `Zombie.moveSpeed` (default 45) |
| Jump too high | `Player.jumpImpulse` (default 320) |

---
> Source: [JoelHJames1/BachataZombieSmasher](https://github.com/JoelHJames1/BachataZombieSmasher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
