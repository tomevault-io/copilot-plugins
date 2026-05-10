## rz-radar-2026

> 1. **读取所有 context 中引用的文件**（system-reminder 里出现的文件路径全部 Read）

# CLAUDE.md — 每次对话必须遵守

## 强制初始化流程（每次对话开始，无一例外）


1. **读取所有 context 中引用的文件**（system-reminder 里出现的文件路径全部 Read）
2. 然后再回答问题

## 开发环境

- OS: Ubuntu 22.04，纯 Python
- 代码在**宿主机**修改
- 
## 项目：RZ_radar-2025（RoboMaster 雷达站）

### 关键模块

| 文件/目录 | 说明 |
|---|---|
| `main.py` | 主程序入口，串口、相机、检测、UI 调度 |
| `detect_function.py` | YOLOv5 + TensorRT 目标检测（car/armor） |
| `hik_camera.py` | 海康相机 SDK 接口（MvImport_Linux） |
| `information_ui.py` | PyQt6 信息面板与地图显示 UI |
| `RM_serial_py/ser_api.py` | 串口通信协议（发送坐标/决策/云台信息） |
| `calibration.py` | 相机→地图仿射变换标定工具 |
| `config.yaml` | 全局配置（队伍颜色、相机模式、串口、滤波器等） |
| `models/` | TensorRT engine 模型（`car.engine`, `armor.engine`） |
| `utils/` | YOLOv5 工具库（loss、metrics、dataloaders 等） |

### 运行模式（`config.yaml` 中 `camera_mode`）

| 模式 | 说明 |
|---|---|
| `hik` | 海康相机，正式比赛用 |
| `test` | 读取本地 `.npy` 测试数据 |
| `video` | USB 相机 / VideoCapture |

### 串口配置

- 端口：`/dev/ttyUSB0`，波特率：`115200`
- `use_serial: False` 时可纯视觉调试，不影响主流程

### 标定文件

- 红方仿射矩阵：`arrays_test_red.npy`
- 蓝方仿射矩阵：`arrays_test_blue.npy`
- 重新标定使用 `calibration.py`

## 工作规范

- 禁止盲目猜测；推断时必须告知用户；不清楚就问
- 贴了日志先解释日志含义
- 添加代码前先读完整份文件
- 遇到问题按：**复现 → 定位 → 排查 → 解决 → 验证 → 复盘**
- 用英文思考，用中文回答  Think in English, respond in Chinese.
- 自动迭代,我喜欢自动化
- 充分利用和插件
- 第一次回答末尾发 1，第二次最后发2,以此类推

当前目标：

---
> Source: [git-alphabet/rz_radar-2026](https://github.com/git-alphabet/rz_radar-2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
