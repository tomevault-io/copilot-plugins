## go2-ros2-sdk

> C++/ROS 2 rules for go2_ros2_sdk edits


# go2_ros2_sdk (C++ / ROS 2) Rules

- Use C++17-compatible code.
- Prefer clear ownership semantics (RAII) and avoid raw `new/delete`.
- Keep node execution non-blocking; avoid long blocking loops in callbacks.
- If adding nodes, prefer ROS 2 best practices (parameters, QoS explicitness).
- Keep build system changes minimal and localized.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KongPedia) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
