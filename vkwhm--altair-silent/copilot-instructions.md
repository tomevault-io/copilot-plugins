## altair-silent

> - **Build**: `colcon build` (requires `nix develop .nix --accept-flake-config` first)

# AGENTS.md - Coding Guidelines for Altair Silent Robot

## Build/Test Commands
- **Build**: `colcon build` (requires `nix develop .nix --accept-flake-config` first)
- **Source environment**: `source install/local_setup.bash`
- **Run single test**: `python -m pytest src/silent_slam/test/test_flake8.py::test_flake8` 
- **Test package**: `colcon test --packages-select silent_controllers`
- **Code style check**: `ament_flake8 src/package_name/` (flake8 for Python)

## Code Style Guidelines
- **Language**: Python 3 with ROS2 (ament_python packages)
- **Imports**: Standard library first, then third-party (rclpy, geometry_msgs), then local imports
- **Naming**: snake_case for functions/variables, PascalCase for classes, private members prefixed with `__`
- **Type hints**: Use type annotations (e.g., `-> None`, `error: float`)
- **Error handling**: Use assertions for preconditions with descriptive messages
- **Comments**: Minimal comments, focus on self-documenting code
- **Formatting**: Follow PEP 8, enforced by flake8 and pep257 linters

## Architecture Notes
- ROS2 lifecycle nodes pattern used extensively
- Abstract base classes for controllers (AbstractController)
- Service-based architecture for commands (CommandTOL services)
- QoS profiles configured for reliable/best-effort communication
- Private methods use double underscore prefix (`__method_name`)

---
> Source: [VKWHM/Altair-Silent](https://github.com/VKWHM/Altair-Silent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
