## comfyui-node-template

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

ComfyUI 自定义节点模板，面向 ComfyUI Registry 标准设计。为开发者和 AI 助手提供从项目初始化到发布的标准化流程。

## 核心架构

```
ComfyUI 加载流程:
__init__.py → NODE_CLASS_MAPPINGS → nodes.py 中的节点类

节点类必须实现:
├── INPUT_TYPES(cls)    # 类方法：定义 required/optional 输入参数
├── RETURN_TYPES        # 类属性：输出类型元组 (如 "IMAGE", "INT")
├── RETURN_NAMES        # 类属性：输出参数名元组
├── FUNCTION            # 类属性：执行函数名 (如 "process")
├── CATEGORY            # 类属性：菜单分类路径
└── process()           # 实际处理逻辑
```

## ComfyUI 数据类型

| 类型 | 说明 | 参数配置 |
|------|------|----------|
| `IMAGE` | 图像张量 (BHWC, 0-1 float) | 直接作为输入类型 |
| `MASK` | 遮罩张量 | 直接作为输入类型 |
| `INT` | 整数 | `{"default": 10, "min": 0, "max": 100, "step": 1}` |
| `FLOAT` | 浮点数 | `{"default": 1.0, "min": 0.0, "max": 10.0, "step": 0.01, "display": "slider"}` |
| `STRING` | 字符串 | `{"default": "...", "multiline": False}` |
| 枚举选择 | 下拉列表 | `(["opt1", "opt2"], {"default": "opt1"})` |

## 发布流程

1. 配置 `pyproject.toml` 中的 `[tool.comfy]` 段：
   - `PublisherId`: 从 registry.comfy.org 获取
   - `DisplayName`: 菜单显示名称

2. 在 GitHub 仓库 Settings → Secrets 添加 `REGISTRY_ACCESS_TOKEN`

3. 修改 `pyproject.toml` 中的 `version` 字段并推送到 `main` 分支

4. GitHub Action (`publish.yml`) 自动触发发布到 ComfyUI Registry

## 本地测试

将项目目录放入 ComfyUI 的 `custom_nodes/` 目录，重启 ComfyUI 即可加载节点。

## 节点开发要点

- 设备选择：支持 `auto/cpu/mps/cuda`，`auto` 时自动检测可用设备
- 进度条：使用 `from comfy.utils import ProgressBar` 提供视觉反馈
- IMAGE 张量：ComfyUI 默认在 CPU 上，如需 GPU 计算需手动 `.to(device)`
- 返回值：必须与 `RETURN_TYPES` 顺序一致的元组

## 添加新节点

1. 在 `nodes.py` 中创建新的节点类，实现上述必需方法
2. 在 `nodes.py` 底部的 `NODE_CLASS_MAPPINGS` 中注册：`"NewNode": NewNode`
3. 在 `NODE_DISPLAY_NAME_MAPPINGS` 中添加显示名称

## AI 助手引导模板

创建新节点时可使用此 prompt 结构：
```
Create a new ComfyUI node based on the ComfyUI-Node-Template.
1. Update pyproject.toml: set name to comfyui-<description>, PublisherId to <id>
2. Implement a node in nodes.py that does <功能描述>
3. Inputs should include <参数列表>, output type is <类型>
4. Update __init__.py to register the node
```

## 示例工作流

`examples/basic_workflow.json` - ComfyUI 工作流示例，包含：
- ExampleNode 节点配置（参数：int=20, float=1.5, device=cpu）
- 可直接在 ComfyUI 中加载（拖拽 JSON 到界面或 Load 按钮）

## 文件结构

```
├── __init__.py              # 导出 NODE_CLASS_MAPPINGS 和 NODE_DISPLAY_NAME_MAPPINGS
├── nodes.py                 # 节点实现
├── pyproject.toml           # 项目配置 + ComfyUI Registry 元数据
├── examples/                # 工作流示例
│   └── basic_workflow.json  # ExampleNode 使用示例
└── .github/workflows/publish.yml  # 自动发布 Action
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kana112233) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
