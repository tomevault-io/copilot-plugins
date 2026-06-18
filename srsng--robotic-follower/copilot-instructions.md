## robotic-follower

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个基于 ROS2 Humble 的机械臂视觉跟随系统，采用开环控制架构，充分复用社区成熟方案：

- **realsense2_camera**: 相机采集（ROS2官方包，apt安装）
- **ros2_dummy_arm_810**: 机械臂驱动 + MoveIt2运动规划（含PyMoveIt2接口）
- **robotic_follower**: 自定义包，核心工作区，代码基本都在这里改

## 开发环境配置

### Python 环境

本项目使用全局 `python3` + `pip`, **不使用 conda**。

### ROS2 环境

已经添加 "source /opt/ros/humble/setup.bash" 到 ~/.bashrc, 不用再处理

### 工作空间环境

已经添加 "source ~/ros2_ws/install/setup.bash" 到 ~/.bashrc, 不用再处理

## 开发工作流

### 1. 创建新模块

```bash
# 进入工作空间 src 目录
cd ~/ros2_ws/src

# 创建新的 ROS2 Python 包
ros2 pkg create package_name --build-type ament_python [...options]

# 或使用 C++ 包
ros2 pkg create package_name --build-type ament_cmake [...options]
```

### 2. 编译工作空间

```bash
# 确保在 workspace 根目录
cd ~/ros2_ws

# 编译所有包（使用符号链接，便于开发）
colcon build --symlink-install

# 只编译特定包
colcon build --symlink-install --packages-select package_name
```

项目使用 `colcon build --symlink-install` 编译，所有更改都通过链接自动反应在build目录中。当且仅当有文件增减时，需要重新编译，否则不要编译.

### 3. 运行节点

```bash

# 运行节点
ros2 run package_name node_name

# 运行 launch 文件
ros2 launch package_name launch_file.py

# 使用调试输出
ros2 run package_name node_name --ros-args --log-level debug
```

### 4. 调试

```bash
# 查看节点列表
ros2 node list

# 查看话题列表
ros2 topic list

# 查看话题信息
ros2 topic info /topic_name

# 查看话题内容
ros2 topic echo /topic_name

# 查看服务列表
ros2 service list

# 调用服务
ros2 service call /service_name package_name/srv/ServiceType

# 查看参数列表
ros2 param list

# 查看节点日志
ros2 node info /node_name
```

## 技术选型规范

+ 优先使用python3，其次才是C++
+ 优先使用社区成熟方案/库
+ 保证可用、稳定的前提下，优先使用复杂度较低的方案/库

## 编码规范

### Python 代码规范

- 使用 PEP 8 编码规范
- 使用类型注解（Type Hints）
- 使用 docstring 文档字符串, 避免使用中文标点符号
- 使用 `rclpy` 编写 ROS2 节点

```python
# 示例节点结构
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class MyNode(Node):
    """示例 ROS2 节点。"""

    def __init__(self):
        super().__init__('my_node')
        self.publisher = self.create_publisher(String, 'topic', 10)
        self.timer = self.create_timer(1.0, self.timer_callback)

    def timer_callback(self):
        msg = String()
        msg.data = 'Hello'
        self.publisher.publish(msg)

def main(args=None):
    rclpy.init(args=args)
    node = MyNode()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        node.get_logger().info("收到中断信号")
    finally:
        node.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### ROS2 节点命名规范

- 节点名称：使用小写字母和下划线，如 `camera_sim_node`
- 话题名称：使用斜杠分隔，如 `/camera/color/image_raw`
- 服务名称：使用斜杠分隔，如 `/hand_eye_calibration/add_sample`
- 参数名称：使用下划线，如 `publish_rate`

### 文件命名规范

- Python 文件：小写字母和下划线，如 `camera_sim_node.py`
- Launch 文件：小写字母和下划线，如 `camera.launch.py`
- 配置文件：小写字母和下划线，如 `camera_config.yaml`

## 常用命令

### 开发命令

```bash
# 检查包依赖
ros2 pkg list
ros2 pkg dependencies package_name

# 查找包
ros2 pkg xml package_name

# 生成接口
ros2 interface list
ros2 interface show std_msgs/msg/String
ros2 interface show package_name/srv/ServiceType

# 查看 TF 树
ros2 run tf2_tools view_frames
ros2 run tf2_ros tf2_echo source_frame target_frame

# 记录和回放
ros2 bag record /topic1 /topic2
ros2 bag info my_bag
ros2 bag play my_bag
```

### 测试命令

```bash
# 运行所有测试
colcon test

# 运行特定包的测试
colcon test --packages-select package_name

# 运行特定测试
colcon test --packages-select package_name --test-filter test_name

# 查看测试结果
colcon test-result --verbose
```

## 训练感知模块

### 训练 3D 检测网络（MMDetection3D）

```bash
cd ~/ws/py/mmdetection3d

# 基线 VoteNet 训练
python3 tools/train.py ~/ros2_ws/src/perception/configs/votenet_sunrgbd_baseline.py

# 继续训练
python3 tools/train.py ~/ros2_ws/src/perception/configs/votenet_sunrgbd_baseline.py --resume
```

## 调试工具

### GDB 调试

```bash
# 使用 gdb 运行节点
ros2 run --prefix 'gdb -ex run --args' package_name node_name

# 或使用调试符号编译
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Debug
```

### 日志调试

```bash
# 设置日志级别为 debug
ros2 run robotic_follower node_name --ros-args --log-level debug

# 设置特定节点的日志级别
ros2 run robotic_follower node_name --ros-args --log-level node_name:=debug

# 设置特定日志器
ros2 run robotic_follower node_name --ros-args --log-level logger_name:=debug
```

## 模块开发指南

### 添加新的 ROS2 节点

1. 在 `src/robotic_follower/robotic_follower/ros_nodes/` 创建节点文件
2. 在 `src/robotic_follower/robotic_follower/ros_nodes/__init__.py` 导出
3. 在 `setup.py` 的 `entry_points` 添加入口点
4. 创建 launch 文件在 `src/robotic_follower/launch/`
5. 更新 `package.xml` 依赖

### 添加新的服务

1. 创建 srv 文件在 `src/robotic_follower/srv/`
2. 在 `package.xml` 添加 `<build_depend>rosidl_default_generators</build_depend>`
3. 在 `CMakeLists.txt` 添加服务生成（C++）或更新依赖（Python3）
4. 在节点中创建服务提供者

### 添加新的消息

1. 创建 msg 文件在 `src/robotic_follower/msg/`
2. 在 `package.xml` 添加依赖
3. 重新编译包
4. 使用消息类型

## 问题排查

### 常见问题

| 问题         | 原因        | 解决方案                    |
| ------------ | ----------- | --------------------------- |
| 模块编译失败 | 依赖缺失    | 检查 package.xml 并安装依赖 |
| TF查询失败   | TF未发布    | 检查 TF树和坐标系名称       |
| 相机未连接   | USB权限不足 | 配置 udev 规则              |

### 调试 TF 问题

```bash
# 查看 TF 树
ros2 run tf2_tools view_frames

# 查看特定 TF
ros2 run tf2_ros tf2_echo source_frame target_frame

# 监控 TF 变化
ros2 run tf2_ros tf2_monitor source_frame target_frame
```

### 调试话题问题

```bash
# 查看话题发布者
ros2 topic info /topic_name

# 查看 Hertz
ros2 topic hz /topic_name

# 查看 Latency
ros2 topic delay /topic_name

# 查看带宽
ros2 topic bw /topic_name
```

### 调试服务问题

```bash
# 查看服务提供者
ros2 service info /service_name

# 查看服务类型
ros2 service type /service_name

# 查看服务接口
ros2 interface show package_name/srv/ServiceType
```

## 项目架构

```
ros2_ws/
├── src/                              # 源代码目录
│   ├── robotic_follower/          # 手眼标定模块
│   └── ros2_dummy_arm_810/            # 机械臂驱动 + MoveIt
├── install/                           # 编译输出
├── build/                             # 构建中间文件
├── log/                               # 构建日志
├── doc/                               # 文档
├── CLAUDE.md                          # 开发指南（本文件）
└── README.md                          # 项目说明
```

## 模块间数据流

## 系统话题汇总

### 发布话题

| 话题名称                                   | 消息类型                       | 来源                 |
| ------------------------------------------ | ------------------------------ | -------------------- |
| `/camera/color/image_raw`                  | `sensor_msgs/Image`            | realsense2_camera    |
| `/camera/depth/image_rect_raw`             | `sensor_msgs/Image`            | realsense2_camera    |
| `/camera/aligned_depth_to_color/image_raw` | `sensor_msgs/Image`            | realsense2_camera    |
| `/camera/color/camera_info`                | `sensor_msgs/CameraInfo`       | realsense2_camera    |
| `/camera/camera/depth/color/points`        | `sensor_msgs/PointCloud2`      | realsense2_camera    |
| `/perception/detections`                   | `vision_msgs/Detection3DArray` | detection_node       |
| `/joint_states`                            | `sensor_msgs/JointState`       | dummy_arm_controller |

### 服务汇总

| 服务名称                           | 来源                 |
| ---------------------------------- | -------------------- |
| `/hand_eye_calibration/add_sample` | hand_eye_calibration |
| `/hand_eye_calibration/execute`    | hand_eye_calibration |
| `/hand_eye_calibration/reset`      | hand_eye_calibration |
| `/dummy_arm/enable`                | dummy_arm_controller |
| `/gripper_open`                    | dummy_arm_controller |
| `/gripper_close`                   | dummy_arm_controller |

## TF 树结构

```
world (世界坐标系)
  ↓ (world_joint, fixed)
base_link (机器人基座)
  ↓ (joint1-joint6, revolute) [动态TF，由 robot_state_publisher 发布]
link1_1_1 ─ link2_1_1 ─ link3_1_1 ─ link4_1_1 ─ link5_1_1 ─ link6_1_1
  ↓ (手眼变换) [动态TF，由 hand_eye_calibration 节点以10Hz周期发布]
camera_link (相机安装位置，与 URDF 中的末端执行器 link6_1_1 关联)
  ↓
camera_color_optical_frame (深度相机光学坐标系，由 realsense2_camera 发布)
camera_depth_optical_frame (深度相机光学坐标系，由 realsense2_camera 发布)
```

### TF 发布方式说明

| TF 变换                       | 发布者                     | 方式                        |
| ----------------------------- | -------------------------- | --------------------------- |
| world → base_link             | static_transform_publisher | 静态 (固定)                 |
| base_link → link6_1_1         | robot_state_publisher      | 动态 (基于 joint_states)    |
| link6_1_1 → camera_link       | hand_eye_calibration       | 静态 (标定后写入到URDF文件) |
| camera_link → *_optical_frame | realsense2_camera          | 静态/动态 (相机内部变换)    |

## 文档

位于 `doc/` 目录：

---
> Source: [srsng/robotic-follower](https://github.com/srsng/robotic-follower) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
