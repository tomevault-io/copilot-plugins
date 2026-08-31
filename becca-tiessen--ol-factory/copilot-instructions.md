## ol-factory

> Godot 4.x with GDScript

Engine & Language

Godot 4.x with GDScript
Use Godot 4 syntax: @onready, @export, signal_name.emit(), typed arrays (Array[BaseIngredient]), FileAccess, etc.
Do NOT use Godot 3 syntax (onready, export, emit_signal(), File.new())

Project Structure
scripts/ # All GDScript files
data/ # Ingredient .tres resources, npcs.json, requests.json
scenes/ # Scene files (.tscn)
assets/ # Sprites, textures, icons
Autoload Singletons (never instantiate — access globally by name)

InventoryManager — stores gathered ingredient counts as { BaseIngredient: int }. Persists to user://save_data.json automatically on every change.
SceneManager — handles door/scene transitions between Outside, Lab, and Cellar.
RequestManager — tiered NPC request system.
AccordManager — stores discovered accords and their recipes.

Ingredient Data Model
Ingredients are Godot custom resources (.tres files in res://data/) using a BaseIngredient script class. Each has:

display_name: String
icon: Texture2D
liquid_color: Color
scent_profile: Vector3
scent_family: String — one of: "floral", "woody", "citrus", "sweet", "green", "spicy"
intensity: float
note_position: String — one of: "top", "middle", "base"

When adding new ingredients, create a new .tres resource following this pattern. Do not hardcode ingredient data in scripts.
Two Interaction Systems (don't mix these up)

Interactable (interactable.gd) — for gatherable resource nodes (rose bush, etc.)

Extends Area2D, uses collision layer 2
Has @export var ingredient: BaseIngredient
collect() adds to InventoryManager, plays tween, then queue_free()
Player detects these via PlayerInteraction script, triggers on E key

BaseInteractable (base_interactable.gd) — for furniture/UI triggers (mixing bench, cellar rack, etc.)

Extends StaticBody2D, uses collision layer 3
Opens UI panels or triggers scene logic

Key Conventions

Signals are typed where possible: signal ingredient_gathered(ingredient: BaseIngredient)
Inventory keyed by resource reference (the loaded .tres), not by string name
Blend/mixture operations use arrays of { "ingredient": BaseIngredient, "amount": float } dictionaries
Quality scoring is 0–10 scale with tiers: Poor (<3), Decent (<5.5), Good (<7.5), Excellent (≥7.5)
NPC data is JSON-driven (data/npcs.json), referenced by npc_id in requests
Save data uses user:// path with JSON format and a version field for migration

Scent Families & Compatibility
Compatibility between families is looked up via ScentCompatibility.get_compatibility(family_a, family_b) — a static table, not computed. When adding new families, update that table.
Style Notes

The game's tone is cozy artisan fantasy — use language like "Essence of Rain on Stone" not "Iso E Super"
NPCs are French-named with distinct personality tags that affect dialogue voice
Keep the player in the fantasy of being a village perfumer, not a chemist

---
> Source: [becca-tiessen/ol-factory](https://github.com/becca-tiessen/ol-factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
