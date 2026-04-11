## legendarypotato

> This is a 2D game developed with the Godot Engine. The project is named "legendarypotato" and features a player character that can explore procedurally generated rooms. The core mechanics include player movement, animation, and dynamic room creation with various props.

# Project Overview

This is a 2D game developed with the Godot Engine. The project is named "legendarypotato" and features a player character that can explore procedurally generated rooms. The core mechanics include player movement, animation, and dynamic room creation with various props.

## Key Technologies

*   **Engine**: Godot Engine (version 4.5)
*   **Language**: GDScript

## Architecture

The project is structured as follows:

*   **`scenes/`**: Contains the game's scenes, including the player, levels, and props.
    *   `player/player.tscn`: The player character scene.
    *   `levels/room_01.tscn`: The main scene that is loaded when the game starts.
    *   `levels/room_procedural.tscn`: A scene that generates a procedural room.
    *   `props/`: Contains scenes for various props like chests, pots, and torches.
*   **`scripts/`**: Contains the GDScript files that control the game's logic.
    *   `player.gd`: Controls player movement and animation.
    *   `room_generator.gd`: Procedurally generates rooms.
*   **`assets/`**: Contains the game's assets, such as sprites and tilesets.

# Building and Running

To run the project, open it in the Godot Engine and press the "Play" button. The main scene is set to `res://scenes/levels/room_01.tscn`.

# Development Conventions

*   **Player Control**: The player is controlled using the standard Godot input actions: `ui_left`, `ui_right`, `ui_up`, and `ui_down`.
*   **Procedural Generation**: The `room_generator.gd` script is responsible for creating rooms dynamically. It includes functions for generating the floor, walls, and placing props.
*   **Animation**: The player character has animations for walking and idling in four directions (up, down, left, right).

# AI Character Generation

The `ai_animation` directory contains a Python-based system for generating character sprites and animations dynamically using AI.

## Architecture

The process works as follows:
1.  An in-game UI (triggered by `Tab`) allows the user to enter a character description.
2.  The `generate_improved.py` script is called, which makes 6 API calls to the OpenRouter API to generate a base character, 4 directional rotations, and 3 walk frames for each direction.
3.  The output is an ASCII art representation of the animation frames, stored in `improved_animation.json`.
4.  The `ascii_to_sprite.py` script converts the ASCII art from the JSON file into a PNG sprite sheet.
5.  Finally, the `ai_character_loader.gd` script in Godot loads the generated PNG, creates the necessary `SpriteFrames`, and applies them to the player's `AnimatedSprite2D`.

## Key Files

### Python Scripts (`ai_animation/`)
-   **`generate_improved.py`**: The main script for generating character animations via the OpenRouter API.
-   **`ascii_to_sprite.py`**: Converts the ASCII art JSON output to a PNG sprite sheet.
-   **`requirements.txt`**: Lists the required Python packages (`httpx`, `Pillow`).

### Godot Scripts (`scripts/`)
-   **`character_generator_ui.gd`**: Controller for the in-game character generation UI.
-   **`ai_character_loader.gd`**: Loads the generated sprite sheet and creates the animations in Godot.

## Manual Testing

You can generate a character from the command line:
```bash
cd ai_animation
source venv/bin/activate
# Set the API key
export OPENROUTER_API_KEY="your_key_here"
# Generate the character
python generate_improved.py "red warrior"
# Convert to a sprite sheet
python ascii_to_sprite.py improved_animation.json generated_character.png 8
```
This will create `generated_character.png` in the `ai_animation` directory.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RonanHevenor)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/RonanHevenor)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
