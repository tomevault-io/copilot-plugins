## semantic-nav

> Add under a new ## ROS2 Debugging section at the top level\n\nWhen debugging ROS2 issues, systematically trace the pipeline from data source to output before proposing fixes. Never guess at root causes (e.g., QoS, CUDA) — confirm each stage is working first.

Add under a new ## ROS2 Debugging section at the top level\n\nWhen debugging ROS2 issues, systematically trace the pipeline from data source to output before proposing fixes. Never guess at root causes (e.g., QoS, CUDA) — confirm each stage is working first.
Add under ## Code Style or ## General Rules section\n\nNever hardcode paths (model paths, environment-specific paths). Always use portable, relative paths or configurable parameters.
Add under ## ROS2 Debugging or ## Environment section\n\nWhen working on this ROS2 project: the workspace uses conda Python environments — colcon does not handle conda shebangs correctly. Use manual shebang patching when needed, not colcon rebuild.
Add under ## ROS2 Debugging section\n\nFor YOLO lifecycle nodes: check lifecycle state (unconfigured/inactive/active) and model download status before investigating other causes. The YOLO node frequently gets stuck in unconfigured state.

---
> Source: [TingdeLiu/Semantic-Nav](https://github.com/TingdeLiu/Semantic-Nav) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
