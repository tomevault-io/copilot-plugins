## jornv02

> Useful to understand how maps are created

Map Creation (External Tool - Tiled):
The maps are created using the Tiled Map Editor, a popular open-source tool for designing 2D game levels.
In Tiled, developers can:
Define tile layers for visuals (e.g., ground, water, buildings) using tilesets.
Create object layers to place and define game entities like the player's starting position, NPCs, and special trigger areas.
Assign custom properties to these objects. For example, an NPC object might have a dialog_id property, and a "scene area" object might have a scene property indicating which map to transition to.
These maps are saved as .tmx files and are stored in the /public/maps/ directory of the project (as seen in asset.manager.ts, though the rule currently just says /maps/ which is relative to public).
Map Asset Loading (asset.manager.ts):
The AssetManager is responsible for loading these .tmx files.
In its setup_maps() method, it creates instances of TiledMapResource (from the @excaliburjs/plugin-tiled library) for each map file (e.g., main_menu.tmx, farm.tmx).
These resources are added to Excalibur's Loader to be loaded when the game starts.
The loaded map resources are stored in assetManager.maps, keyed by the MAPS enum (e.g., MAPS.FARM).
Level Configuration (level.manager.ts):
The LevelManager uses a levels_config array. Each entry in this array defines a game level/scene.
This configuration links a map name (using the MAPS enum) to other scene properties like the scene's name, background music (SONGS enum), and an array of dialogues available in that scene.
For each configuration, a new Level instance (from src/scenes/level.scene.ts) is created.
Map Integration and Display in Scenes (level.scene.ts):
The Level class (which extends Excalibur's Scene) takes a configuration object (from levelManager.levels_config) in its constructor.
In its init(engine) method (called during onInitialize):
It retrieves the corresponding TiledMapResource from assetManager.maps using the map_name from its configuration.
Crucially, it then adds the Tiled map to the Excalibur scene using this.map.addTiledMapToScene(engine). This renders the tile layers of the map.
It calculates the map's dimensions (map_width, map_height) based on the Tiled data.
Spawning Actors from Tiled Object Layers (level.scene.ts):
After displaying the map, the Level class iterates through specific object layers defined in the Tiled map:
TILED_OBJECT.SCENE_AREA ("scene_area"): Creates SceneArea actors. The custom property scene on these Tiled objects determines which map the player will transition to when entering this area.
TILED_OBJECT.CHICKEN ("chickens"): Creates Chicken NPC actors at the specified positions. The custom property dialog_id is extracted using get_dialog_id to link the NPC to specific dialogue trees in dialog.manager.ts.
TILED_OBJECT.COW ("cows"): Similar to chickens, creates Cow NPC actors with associated dialogues.
TILED_OBJECT.PLAYER ("player"): Determines the initial position for the Player actor in that scene.
These actors (Player, Chicken, Cow, SceneArea) are then added to the Excalibur scene.
Camera Setup (level.scene.ts):
The Level class uses the map's dimensions (map_width, map_height) to set up the camera.
this.camera.strategy.limitCameraBounds(map_bounds) ensures the camera does not scroll beyond the edges of the current map.
The camera is also set to lock onto the Player actor (this.camera.strategy.lockToActor(this.player)), so it follows the player's movement.
A default zoom level is applied (this.camera.zoom = 2).
In essence, Tiled is used as an external level editor. The game loads these Tiled files, renders their visual layers, and then uses the object layers within them as a blueprint to dynamically populate the game world with interactive actors and define their properties and behaviors. The level.manager.ts orchestrates which maps are loaded as scenes, and level.scene.ts handles the actual rendering and entity creation based on the Tiled data.

Would you like a more detailed explanation of any of these steps, or perhaps explore how a specific map feature (like scene transitions or NPC placement) is implemented in the code?

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Zacxxx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
