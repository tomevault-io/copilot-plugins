## file-structure

> ├── .cursor/                    # Cursor IDE 配置

# 项目文件结构指南

## 📁 目录结构概览

```
lucky_ball/
├── .cursor/                    # Cursor IDE 配置
│   └── rules/                  # Cursor 规则文件
├── .git/                       # Git 版本控制
├── .github/                    # GitHub 配置
│   └── workflows/              # GitHub Actions 工作流
├── data/                       # 数据文件存储
├── pics/                       # 图片和图表文件
├── reports/                    # 分析报告文档
├── scripts/                    # 核心分析脚本
├── test/                       # 测试文件和数据
├── README.md                   # 项目说明文档
├── requirements.txt            # Python 依赖包
├── setup_repo.sh              # 项目初始化脚本
├── .gitignore                  # Git 忽略文件配置
└── LICENSE                     # 开源许可证
```

## 📂 目录详细说明

### 🔧 配置文件目录

#### `.cursor/rules/` - Cursor IDE 规则
- [project-overview.mdc](mdc:.cursor/rules/project-overview.mdc) - 项目概览规则
- [github-actions.mdc](mdc:.cursor/rules/github-actions.mdc) - GitHub Actions 规则
- [data-analysis.mdc](mdc:.cursor/rules/data-analysis.mdc) - 数据分析规则
- [development-guide.mdc](mdc:.cursor/rules/development-guide.mdc) - 开发指南规则
- [api-reference.mdc](mdc:.cursor/rules/api-reference.mdc) - API 参考规则
- [file-structure.mdc](mdc:.cursor/rules/file-structure.mdc) - 文件结构规则（本文件）

#### `.github/workflows/` - 自动化工作流
- [update-lottery-data.yml](mdc:.github/workflows/update-lottery-data.yml) - 双色球数据自动更新工作流
- [update-super-lotto-data.yml](mdc:.github/workflows/update-super-lotto-data.yml) - 大乐透数据自动更新工作流

### 📊 数据文件目录

#### `data/` - 原始数据存储
- [lottery_data.json](mdc:data/lottery_data.json) - 双色球历史开奖数据（642KB，约1800期）
- [super_lotto_data.json](mdc:data/super_lotto_data.json) - 大乐透历史开奖数据（957KB，约2700期）

#### `reports/` - 分析报告文档
- [analysis_report.md](mdc:reports/analysis_report.md) - 双色球数据分析报告
- [super_lotto_analysis_report.md](mdc:reports/super_lotto_analysis_report.md) - 大乐透数据分析报告

#### `pics/` - 可视化图表
- [lottery_frequency_analysis.png](mdc:pics/lottery_frequency_analysis.png) - 双色球频率分析图表
- [super_lotto_frequency_analysis.png](mdc:pics/super_lotto_frequency_analysis.png) - 大乐透频率分析图表

### 🔬 核心代码目录

#### `scripts/` - 主要分析脚本
- [lottery_analyzer.py](mdc:scripts/lottery_analyzer.py) - 双色球数据分析器（38KB，1019行）
- [super_lotto_analyzer.py](mdc:scripts/super_lotto_analyzer.py) - 大乐透数据分析器（39KB，1016行）

#### `test/` - 测试文件和数据
**测试脚本：**
- [test_analyzer.py](mdc:test/test_analyzer.py) - 双色球分析器测试脚本
- [test_super_lotto_analyzer.py](mdc:test/test_super_lotto_analyzer.py) - 大乐透分析器测试脚本
- [test_consistency.py](mdc:test/test_consistency.py) - 数据一致性测试脚本

**测试数据：**
- [test_lottery_data.json](mdc:test/test_lottery_data.json) - 双色球测试数据（27KB）
- [test_super_lotto_data.json](mdc:test/test_super_lotto_data.json) - 大乐透测试数据（22KB）

**测试报告：**
- [test_analysis_report.md](mdc:test/test_analysis_report.md) - 双色球测试分析报告
- [test_super_lotto_report.md](mdc:test/test_super_lotto_report.md) - 大乐透测试分析报告

### 📋 项目配置文件

#### 根目录配置文件
- [README.md](mdc:README.md) - 项目主要说明文档（4.4KB）
- [requirements.txt](mdc:requirements.txt) - Python 依赖包列表
- [setup_repo.sh](mdc:setup_repo.sh) - 项目初始化脚本
- [.gitignore](mdc:.gitignore) - Git 忽略文件配置
- [LICENSE](mdc:LICENSE) - MIT 开源许可证

## 🔄 文件生成流程

### 自动生成文件
以下文件由 GitHub Actions 自动生成和更新：

1. **数据文件**（每天23:00 UTC+8 自动更新）
   - `data/lottery_data.json`
   - `data/super_lotto_data.json`

2. **分析报告**（随数据更新同步生成）
   - `reports/analysis_report.md`
   - `reports/super_lotto_analysis_report.md`

3. **可视化图表**（随分析完成自动生成）
   - `pics/lottery_frequency_analysis.png`
   - `pics/super_lotto_frequency_analysis.png`

### 手动维护文件
以下文件需要手动维护和更新：

1. **核心脚本**
   - `scripts/lottery_analyzer.py`
   - `scripts/super_lotto_analyzer.py`

2. **测试脚本**
   - `test/test_*.py`

3. **配置文件**
   - `.github/workflows/*.yml`
   - `requirements.txt`
   - `.gitignore`

## 📏 文件大小参考

### 大型数据文件（>100KB）
- `data/super_lotto_data.json` - 957KB（完整历史数据）
- `data/lottery_data.json` - 642KB（完整历史数据）
- `pics/super_lotto_frequency_analysis.png` - 263KB
- `pics/lottery_frequency_analysis.png` - 229KB

### 中型代码文件（10-100KB）
- `scripts/super_lotto_analyzer.py` - 39KB
- `scripts/lottery_analyzer.py` - 38KB
- `test/test_lottery_data.json` - 27KB
- `test/test_super_lotto_data.json` - 22KB

### 小型配置文件（<10KB）
- 各种 `.md` 报告文件 - 6KB 左右
- 测试脚本 - 2-3KB
- 配置文件 - 1-4KB

## 🎯 文件命名规范

### 命名模式
- **分析器脚本**: `{彩票类型}_analyzer.py`
- **数据文件**: `{彩票类型}_data.json`
- **分析报告**: `{彩票类型}_analysis_report.md`
- **频率图表**: `{彩票类型}_frequency_analysis.png`
- **测试脚本**: `test_{彩票类型}_analyzer.py`
- **测试数据**: `test_{彩票类型}_data.json`

### 彩票类型标识
- `lottery` - 双色球
- `super_lotto` - 大乐透

## 📝 文件更新频率

### 每日自动更新
- 数据文件（`data/`）
- 分析报告（`reports/`）
- 可视化图表（`pics/`）

### 按需手动更新
- 核心脚本（`scripts/`）
- 测试文件（`test/`）
- 配置文件（根目录和 `.github/`）
- 文档规则（`.cursor/rules/`）

### 版本控制
- 所有文件都纳入 Git 版本控制
- 大型数据文件通过 `.gitignore` 选择性忽略
- 自动生成的文件通过 GitHub Actions 提交

---
> Source: [snjyor/lucky_ball](https://github.com/snjyor/lucky_ball) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
