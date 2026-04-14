## ogscope

> 本文件为 Claude AI 助手提供项目上下文指导

# CLAUDE.md

本文件为 Claude AI 助手提供项目上下文指导

## 项目概述

OGScope 是一个基于 Raspberry Pi Zero 2W 的电子极轴镜系统，用于天文摄影中的精确极轴校准。

## 技术栈

- **硬件**: Raspberry Pi Zero 2W, IMX327 相机, 2.4寸 SPI LCD
- **语言**: Python 3.9+
- **包管理**: Poetry
- **Web 框架**: FastAPI + Uvicorn
- **日志**: Loguru
- **测试**: Pytest
- **代码质量**: Black, Ruff, MyPy
- **星空解算控制台前端**: `web/analysis-ui`（Vite + React + Tailwind），构建输出 `web/static/analysis-lab/`；本地执行 `cd web/analysis-ui && npm install && npm run build`；同步到开发板见 `scripts/sync_dev_board.sh`（环境变量 `OGSCOPE_DEV_HOST`、`OGSCOPE_DEV_PATH` 等）。i18n 见 `web/static/i18n/analysis.zh.json` / `analysis.en.json`。

## 开发环境

- **IDE**: PyCharm Professional 2025 (远程开发)
- **版本控制**: GitHub
- **CI/CD**: GitHub Actions

## 项目结构

```
ogscope/
├── core/          # 核心功能：相机、图像处理、板块求解
├── hardware/      # 硬件接口：相机驱动、显示驱动
├── web/           # FastAPI Web 服务
├── ui/            # SPI 屏幕界面
├── algorithms/    # 天文算法
├── data/          # 数据管理
├── indi/          # INDI 集成（Phase 3）
└── utils/         # 工具函数
```

## 开发阶段

### Phase 1 - MVP (当前)
- ✅ 项目结构搭建
- 🔄 基础相机功能
- 🔄 Web 界面和 API
- 🔄 简单极轴校准算法

### Phase 2 - 完整功能
- ⏳ SPI 屏幕支持
- ⏳ 高级板块求解
- ⏳ 移动 App

### Phase 3 - 生态集成
- ⏳ INDI 驱动
- ⏳ 赤道仪控制

## 代码规范

- 行长度: 88 字符 (Black 默认)
- 类型提示: 使用 Python 类型注解
- 文档字符串: Google 风格
- 导入顺序: 标准库 → 第三方库 → 本地模块
- 注释规范: 以后新增或修改注释必须使用中英文双语（中文 / English）

## 测试标记

- `@pytest.mark.unit`: 单元测试
- `@pytest.mark.integration`: 集成测试
- `@pytest.mark.hardware`: 需要实际硬件的测试
- `@pytest.mark.slow`: 运行较慢的测试


## 注意事项

- 避免在代码中硬编码路径，使用配置系统
- 硬件相关代码应提供模拟实现，便于在开发机上测试
- 所有 API 端点应编写单元测试
- 日志使用 Loguru，不要使用 print()

## 项目配置信息

### 服务器连接信息
- **服务器地址**: [配置为环境变量]
- **服务器项目目录**: [配置为环境变量]
- **连接方式**: SSH
- **用户名**: [配置为环境变量]
- **端口**: [配置为环境变量]

### 开发环境配置
- **本地项目路径**: [用户自定义]
- **Python 版本**: 3.9+
- **包管理器**: Poetry
- **虚拟环境**: Poetry 管理

### 部署配置
- **生产环境**: Raspberry Pi Zero 2W 开发板
- **测试环境**: [与生产环境相同]
- **虚拟环境目录**: [用户自定义]

### 系统服务配置
在开发板运行时项目已配置为系统服务，服务配置文件位于 `/etc/systemd/system/ogscope.service`：

```ini
[Unit]
Description=OGScope Service
After=network.target

[Service]
Type=simple
User=[用户名]
WorkingDirectory=[项目目录]
Environment=PYTHONPATH=[系统Python路径]
Environment=LD_LIBRARY_PATH=[系统库路径]
Environment=OGSCOPE_RELOAD=false
Environment=OGSCOPE_LOG_LEVEL=INFO
ExecStart=[虚拟环境路径]/bin/python -m ogscope.main
Restart=on-failure
RestartSec=3

[Install]
WantedBy=multi-user.target
```

在本地运行时，使用虚拟环境，因为有些硬件只能在开发板上调用，所以本地只是代码编写，远程测试

### WiFi 与网络（NetworkManager）

- **唯一详解**见 [`docs/development/wifi-nm.md`](docs/development/wifi-nm.md)：热点默认密码、`network.env`、`/debug/system`、Web API、sudoers（`ogscope-wifi` / `ogscope-nmcli`）等。
- **开机引导**：`ogscope-network-boot.service`（root oneshot，`Before=ogscope.service`）在冷启动时若 STA 长时间无可用 IPv4 则切 **AP**，不依赖 Python 进程。
- **运行时 STA 回滚**：用户通过 Web/API 切 STA 成功后，应用内按 `wifi_sta_rollback_*` 超时无 IPv4 再切回 AP；与开机引导**分工不同、不冲突**（先 boot 完成再启动 `ogscope`）。
- 二者均可能最终执行「切 AP」，行为**幂等**；不建议删除运行时回滚，否则仅冷启动有保障，**运行中**切 STA 失败后需手动恢复。

**重要说明**：
- 系统库（如 `libcamera`、`picamera2`）安装在系统环境中，通过 `PYTHONPATH` 环境变量注入到虚拟环境
- `LD_LIBRARY_PATH` 确保系统库的链接库路径正确
- 服务使用虚拟环境运行，但可以访问系统安装的相机驱动库

### 常用命令
```bash
# 连接服务器
ssh [ogstartech]@[192.168.31.16] -p [22]

# 部署到服务器
# 使用 git clone 或手动上传

# 在服务器上运行
sudo systemctl restart ogscope

# 查看服务状态
sudo systemctl status ogscope

# 查看服务日志
sudo journalctl -u ogscope -f

# 手动运行（带环境变量）
cd [项目目录]
PYTHONPATH=[系统Python路径] LD_LIBRARY_PATH=[系统库路径] source [虚拟环境路径]/bin/activate
python -m ogscope.main
```

## Git 工作流

- 主分支: `main` (稳定版本)
- 开发分支: `dev` (开发版本)
- 功能分支: `feature/xxx`
- 修复分支: `fix/xxx`

### 提交信息格式
**必须使用中英文双语提交信息**，格式如下：
```
中文描述 / English description

- 中文详细说明 / English detailed explanation
- 中文变更内容 / English change content
```

示例：
```
修复相机接口500错误 / Fix camera API 500 error

- 添加缺失的方法实现 / Add missing method implementations
- 重新组织服务类结构 / Reorganize service class structure
```

### 提交类型前缀
```
feat: 添加新功能 / Add new feature
fix: 修复bug / Fix bug
docs: 更新文档 / Update documentation
style: 代码格式调整 / Code style changes
refactor: 重构代码 / Refactor code
test: 添加测试 / Add tests
chore: 构建/工具变更 / Build/tool changes
```

## 开发板调试配置

### 开发板连接信息
- **IP地址**: 192.168.31.16
- **用户名**: ogstartech
- **端口**: 22
- **连接方式**: SSH

### 调试工作流程
1. **代码修改后必须上传**: 修改或新建的代码必须先上传到开发板
2. **不要遗漏文件**: 确保所有相关文件都已上传
3. **服务重启**: 项目以系统服务方式运行，修改后需要重启服务
4. **调试前确认**: 如果有不明白的地方，先询问用户再开始工作

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OG-star-tech)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/OG-star-tech)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
