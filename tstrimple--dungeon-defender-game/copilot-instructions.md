## dungeon-defender-game

> > This file contains detailed development information for Claude Code.

# Dungeon Defender Game - Development Notes

> This file contains detailed development information for Claude Code.
> User-facing instructions are in README.md

## Project Setup
- **Engine**: Bevy 0.16 with minimal features for faster compilation
- **Features enabled**: bevy_winit, bevy_render, bevy_core_pipeline, bevy_pbr, bevy_asset, bevy_scene, bevy_color, x11
- **Features disabled**: Audio (alsa issues), UI, text, gltf, animation, png to reduce build time

## Game Architecture
- **Player**: Red cube (1x2x1) with white front face, black back face for orientation
- **Controls**: Standard third-person (WASD movement, mouse look turns player and camera)
- **Physics**: Gravity-based system with Velocity component, jump force of 8.0, gravity of 20.0
- **Camera**: Third-person camera follows behind player's back
- **Level**: 20x20 ground plane with gray boundary walls and brown obstacles
- **Collision**: AABB collision detection between player and all objects with Collider components
- **ECS Systems**: player_movement, player_physics, collision_detection, camera_controller, mouse_look
- **Components**: PlayerController (with yaw), Velocity, Collider, CameraController (pitch only)
- **Hierarchy**: Player entity has child entities for body, front marker, back marker
- **Mouse Look**: Horizontal mouse controls player yaw, vertical mouse controls camera pitch

## Development Notes
- First build takes 5-15 minutes due to Bevy dependencies
- Subsequent builds are much faster with incremental compilation

## Physics Fixes
- Fixed sliding bug: Gravity only applies when player is not grounded
- Proper collision detection: Ground vs wall collisions handled separately
- Velocity clearing: All velocity zeroed when landing on ground or obstacles

## Known Issues
- **GPU Error**: "Unable to find a GPU!" on headless/virtualized systems
  - Bevy requires hardware GPU acceleration
  - Won't run in SSH sessions or VMs without GPU passthrough
  - Need X11/Wayland display and GPU drivers
- **Solution**: Run on system with physical GPU and display

## File Maintenance
- **README.md**: User-facing documentation (build instructions, features, controls)
- **CLAUDE.md**: Development notes for AI assistance (architecture details, troubleshooting)
- **shell.nix**: NixOS development environment specification
- Keep README.md concise and user-friendly
- Keep CLAUDE.md detailed for development context

---
> Source: [tstrimple/dungeon_defender_game](https://github.com/tstrimple/dungeon_defender_game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
