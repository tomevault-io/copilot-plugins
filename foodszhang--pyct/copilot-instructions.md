## pyct

> 个人CBCT（锥形束CT）采集→校准→重建全流程软件，跨平台设计（主要Windows）。

# AGENTS.md - CBCT 重建 Pipeline

## 项目概述
个人CBCT（锥形束CT）采集→校准→重建全流程软件，跨平台设计（主要Windows）。

## 构建/测试命令

### 包管理（uv）
```bash
uv pip install -r requirements.txt
uv pip install <包名>
```

### 运行测试
单个测试文件执行：
```bash
python ct_cli_test.py
python thread_test.py
python detector_test.py
python algorithm/astra/test_ast.py
```

### 运行应用程序
```bash
python main.py              # 主入口
python qt_gui/gui.py        # Qt GUI界面
python ct_cli.py            # CLI命令行界面
```

## 代码风格指南

### 导入规范
- 标准库导入放前面
- 第三方库放第二（numpy为np，cv2，numba为nb）
- 分组顺序：标准库 → 第三方 → 本地模块
- 避免 `from xxx import *`，除非是numpy的便利导入

```python
# 推荐
import os
import sys
from threading import Thread
import numpy as np
import cv2
from typing import Optional

# 避免
from multiprocessing.connection import *
```

### 命名规范
- **类名**：CamelCase（如 `ConeBeam`、`Detector`、`Calibration`）
- **函数/方法**：snake_case（如 `load_img`、`calculate`、`seq_start`）
- **常量**：UPPER_SNAKE_CASE 或模块级赋值（如 `sini`、`cosi`）
- **类型变量**：类型注解中用 CamelCase（如 `img: np.ndarray`、`SOD: float`）

### 类型注解
- 函数参数和返回值使用类型注解
- 常用类型：`int`、`float`、`str`、`Optional[T]`、`np.ndarray`
- numpy数组：`img: np.ndarray`，不用 `typing.Any`

```python
# 推荐
def calculate(self) -> tuple[float, float, float, float, float]:
def load_img(self, path: str) -> list[list[tuple[int, int]]]:

# 避免
def calculate(self):  # 无返回类型
```

### 错误处理
- 尽量使用特定异常
- GUI错误：用户提示用 `QtWidgets.QMessageBox.critical()`
- 致命错误：先向stdout写错误码，再 `sys.exit(1)`

```python
# 推荐
try:
    scanner = DexelaPy.BusScannerPy()
except DexelaPy.DexelaExceptionPy as e:
    sys.stdout.write("ERROR01\n")
    sys.exit(1)

# GUI错误处理
QtWidgets.QMessageBox.critical(self, "警告", "错误信息")
```

### 跨平台注意事项
- Windows命名管道：`r"\\.\pipe\detectResult"`
- 串口端口：通过 `config.yaml` 配置（如COM3、COM4）
- 文件路径：始终使用 `os.path.join()` 构建路径
- 线程：后台线程使用 `daemon=True`

```python
# 推荐
full_path = os.path.join(self.proj_path, f'{i}.tif')
client = Client(r"\\.\pipe\detectResult", authkey=b"ctRestruct")

# 避免
full_path = f"{self.proj_path}/{i}.tif"  # 仅Unix有效
```

### Numba JIT函数
- 性能关键循环使用 `@nb.jit(nopython=True)`
- 可并行循环加 `parallel=True` 和 `nb.prange`
- 避免在紧循环中混用Numba编译和Python代码

```python
@nb.jit(nopython=True, parallel=True)
def _backproject(img: np.ndarray, i: int, result_voxel: np.ndarray, ...):
    for k1 in nb.prange(N):
        ...
```

### 类结构
- `__init__` 用于初始化，需要时用 `__del__` 清理
- 当前代码使用普通类，而非dataclass
- PySide6信号槽：`QtCore.Signal(np.ndarray)`

### 文件组织
- 算法代码：`algorithm/` 目录
- GUI代码：`qt_gui/` 目录
- 校准模块：`algorithm/calibration/cal.py`
- 重建模块：`algorithm/astra/conebeam.py`
- 配置文件：`config.yaml`（YAML格式）

### 文档
- 个人项目可使用中文注释
- 复杂算法简要说明即可
- 当前代码库文档字符串较少，保持简洁

## 架构说明

### 流程图
1. **采集**：`detector.py`（DexelaPy集成）→ `pipe.py`
2. **校准**：`algorithm/calibration/cal.py`（计算SOD、SDD、u0、v0、theta）
3. **重建**：`algorithm/astra/conebeam.py`（FDK_CUDA算法）
4. **界面**：`qt_gui/gui.py`（PySide6界面）

### 关键依赖
- `numpy`、`opencv-python` - 数组/图像处理
- `numba` - 重建核的JIT编译加速
- `astra-toolbox` - GPU加速CT重建
- `PySide6` - Qt GUI框架
- `pyserial` - X射线控制器的串口通信
- `pyyaml` - 配置文件解析
- `DexelaPy` - Windows专用探测器硬件接口（.pyd文件）

### 硬件接口
- X射线控制器：`UltraBrightController`（串口）
- 旋转台：`ZolixMcController`（串口）
- 探测器：DexelaPy（仅Windows）

## 配置说明
- 所有设置在 `config.yaml`
- 配置段：`BrightController`、`CalibrationParam`、`ReconParam`、`ZolixMcController`
- YAML加载：`yaml.load(open("config.yaml"), Loader=yaml.FullLoader)`

## 执行长任务的方式
对于可能超时的命令（安装包、编译、训练），
必须用以下方式执行而不是直接运行：
1. 用 nohup 或 & 后台运行，输出重定向到日志文件
2. 然后轮询日志文件判断是否完成

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/foodszhang)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/foodszhang)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
