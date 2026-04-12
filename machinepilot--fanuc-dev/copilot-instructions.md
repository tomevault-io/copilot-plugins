## fanuc-dev

> FANUC ROS 2 Driver context — ros2_control, MoveIt 2, CRX models, mock vs physical hardware


# FANUC ROS 2 Driver

When assisting with the FANUC ROS 2 Driver, ros2_control, MoveIt 2, or CRX robot topics:

1. **Use the index** — `physical_ai/ros2/FANUC_ROS2_INDEX.md` maps topics to workspace resources.

2. **Reference doc** — `physical_ai/references/FANUC_ROS2_Driver_Reference.md` for architecture, packages, and supported models.

3. **Packages**: `fanuc_driver`, `fanuc_crx_description`, `fanuc_moveit_config`, `fanuc_hardware_interface`, `fanuc_controllers`, `slider_publisher`.

4. **Launch files**:
   - URDF: `ros2 launch fanuc_crx_description view_crx.launch.py robot_model:=crx10ia`
   - Mock: `ros2 launch fanuc_moveit_config fanuc_moveit.launch.py robot_model:=crx10ia use_mock:=true`
   - Physical: `ros2 launch fanuc_moveit_config fanuc_moveit.launch.py robot_model:=crx10ia robot_ip:="<ip>"`

5. **Supported models**: CRX-5iA, CRX-10iA, CRX-10iA/L, CRX-20iA/L, CRX-30iA, CRX-30-18A.

6. **Colcon workspace**: `~/ws_fanuc/` — source with `source ~/ws_fanuc/install/setup.bash` before launch.

7. **Official docs**: https://fanuc-corporation.github.io/fanuc_driver_doc/main/index.html

8. **Controller requirements** (physical): R-30iB+ with J519 Stream Motion + R912 Remote Motion, or S636 External Control Package.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/machinepilot)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/machinepilot)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
