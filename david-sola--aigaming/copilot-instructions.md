## aigaming

> "name": "AIGaming Racing",

{
    "game": {
        "name": "AIGaming Racing",
        "version": "1.0",
        "description": "A browser-based multiplayer racing game inspired by TrackMania."
    },
    "controls": {
        "keyboard": {
            "accelerate": ["ArrowUp", "W"],
            "brake": ["ArrowDown", "S"],
            "turn_left": ["ArrowLeft", "A"],
            "turn_right": ["ArrowRight", "D"],
            "reset_car": ["R"]
        }
    },
    "physics": {
        "engine": "Cannon.js",
        "car": {
            "max_speed": 200,
            "acceleration": 10,
            "braking_force": 15,
            "friction": 0.98,
            "gravity": -9.81,
            "collision_response": "elastic"
        },
        "track": {
            "surface_friction": 0.9,
            "jump_boost": 1.2,
            "checkpoint_system": true
        }
    },
    "multiplayer": {
        "enabled": true,
        "max_players": 8,
        "networking": {
            "server": "Node.js",
            "protocol": "WebSockets",
            "sync_rate": "60hz",
            "lag_compensation": true,
            "position_interpolation": true
        },
        "race_logic": {
            "lobby_system": true,
            "ready_check": true,
            "start_countdown": 3,
            "respawn_after_fall": true
        }
    },
    "singleplayer": {
        "enabled": true,
        "ai_opponents": {
            "enabled": false,
            "difficulty_levels": ["Easy", "Medium", "Hard"],
            "behavior_model": "path-following"
        },
        "time_trial": {
            "enabled": true,
            "ghost_racer": true
        }
    },
    "leaderboard": {
        "enabled": true,
        "global_ranking": true,
        "personal_best_tracking": true,
        "data_storage": "MongoDB"
    },
    "track_editor": {
        "enabled": false,
        "save_system": "localStorage",
        "track_sharing": "not available"
    },
    "graphics": {
        "engine": "Three.js",
        "render_quality": "medium",
        "low_poly_mode": true,
        "shadows": false,
        "textures": "minimal"
    },
    "sound": {
        "enabled": true,
        "background_music": "enabled",
        "sound_effects": ["engine", "collision", "race_start"]
    },
    "ads_and_monetization": {
        "ads_enabled": false,
        "in_app_purchases": {
            "enabled": false,
            "cosmetics": []
        }
    }
}

---
> Source: [David-Sola/AIGaming](https://github.com/David-Sola/AIGaming) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
