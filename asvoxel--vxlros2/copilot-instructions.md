## vxlros2

> ASVXL SDK 的 ROS2 封装，将 VXL435 / VXL6X5 深度相机集成到 ROS2 生态系统中。

# VxlROS2

ASVXL SDK 的 ROS2 封装，将 VXL435 / VXL6X5 深度相机集成到 ROS2 生态系统中。

## 项目结构

```
VxlROS2/
├── vxlsdk/              # ASVXL SDK 占位目录（兜底来源；正常开发用同级 vxlsdk 项目）
├── vxl_camera/          # 主节点包 (C++)
├── vxl_camera_msgs/     # 自定义消息/服务定义
├── vxl_description/     # URDF 模型描述
└── docs/                # 设计文档
```

## 技术约束

- **C++ 标准**: C++17
- **ROS2 版本**: Humble+ (最低支持)
- **构建系统**: ament_cmake
- **节点架构**: 必须使用 rclcpp_components 组件化节点，支持 intra-process 零拷贝
- **SDK 依赖**: 优先使用同级 `../vxlsdk/` 项目编译产物（`vxlsdk/sdk/current/`）；也可解压发行包到 `vxlros2/vxlsdk/`
- **SDK 路径优先级**: CMake 参数 > 环境变量 `VXL_SDK_DIR` > 同级 `../../vxlsdk/sdk/current/` > 项目内 `../vxlsdk/`
- **API 统一**: 全部使用 ASVXL C++ API (vxl.hpp)，禁止直接调用 C API

## 编码规范

- 文件名: snake_case (如 `vxl_camera_node.cpp`)
- 类名: PascalCase (如 `VxlCameraNode`)
- ROS2 话题/服务名: snake_case，以 `~/` 前缀表示私有
- 参数名: snake_case，按功能分组 (如 `color.width`, `depth.enabled`)
- 所有公共 API 函数必须有错误处理，不允许裸异常传播

## 关键设计决策

1. **单节点多传感器**: 一个节点管理一个设备的所有传感器 (color/depth/ir)
2. **帧同步**: 使用 ASVXL Pipeline API 的 SyncMode 进行硬件级同步
3. **输出模式**: 默认 RGBD 或 RGB+Depth 两种模式，IR/单流等仅供调测
4. **参数系统**: 通过 ROS2 动态参数暴露 ASVXL 选项，支持运行时调整
5. **资源释放顺序**: Stream → Sensor → Device → Context，严格遵守
6. **双节点变体**: 同时提供两个组件入口
   - `vxl_camera::VxlCameraNode` (executable: `vxl_camera_node`) — 非 lifecycle，启动即工作
   - `vxl_camera::VxlCameraLifecycleNode` (executable: `vxl_camera_lifecycle_node`) — lifecycle 管理 + USB 热插拔自动恢复（启动默认 auto-activate；`--no-auto-activate` 让外部 lifecycle_manager 接管）
   - 共享 `sensor_options.{hpp,cpp}` 中的动态参数表与冷参数集

## 版本管理

- **仓库**: https://github.com/asvoxel/vxlros2
- **版本格式**: vX.Y.Z (语义化版本)
- **打标签规则**: 除非用户明确指定完整版本号，否则只递增最后一位 patch 版本（如 v0.1.0 → v0.1.1 → v0.1.2）

## 构建与测试

```bash
colcon build --packages-select vxl_camera_msgs vxl_camera
colcon test --packages-select vxl_camera
```

## 参考

- 竞品 ROS2 集成: `3rds/{orbbec_ros2, realsense_ros, zed_ros2_wrapper}`
- ASVXL SDK API: `vxlsdk/include/vxl*.hpp`（或同级项目源码 `../vxlsdk/include/vxl*.hpp`）
- SDK 发行包: https://github.com/asvoxel/vxlsdk-rel/releases

---
> Source: [asvoxel/vxlros2](https://github.com/asvoxel/vxlros2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
