## angelina-pet

> 本仓库是一个独立的《明日方舟》透明桌宠项目，默认角色为予愿安洁莉娜。它不依赖 Codex / ChatGPT，聊天功能通过用户自行配置的 OpenAI 兼容 API 实现。

# Angelina-pet

本仓库是一个独立的《明日方舟》透明桌宠项目，默认角色为予愿安洁莉娜。它不依赖 Codex / ChatGPT，聊天功能通过用户自行配置的 OpenAI 兼容 API 实现。

## 核心约定

- 程序入口：`main.py`
- 默认角色：予愿安洁莉娜
- 素材目录：`pets/<角色名>/`，包含 `manifest.json`、`frames/<state>/frame_0000.png`、`webm/`
- 设置文件：运行目录下的 `settings.json`
- 固定动画状态：`idle`（Relax）、`interact`、`move`、`sit`、`sleep`
- 飞行动画状态：`fly_begin`、`fly`、`fly_end`
- 固定状态帧率：20fps；飞行状态帧率：60fps（以 `manifest.json` 内各状态的 `fps` 为准）
- 帧尺寸：1000x1000 透明 PNG，飞行帧按 bbox 裁剪
- 单文件交付：`Angelina-pet.exe`（自解压，双击即用）
- 便携交付：`Angelina-pet-portable.zip`（解压后双击 `启动桌宠.exe`）
- 打包方式：便携 Python + PySide6 + C# 自解压 stub，不使用 PyInstaller

## 功能

- 右键菜单：坐下、放松、睡觉、形态、聊天、历史对话、全屏自动隐藏、锁定/解锁、设置、缩放、退出
- 系统托盘：显示/隐藏、开机自启动、关闭
- 形态：固定形态（不飞行）、自由移动形态（自动随机飞行，不跨显示器）
- 聊天：双击进入聊天模式，支持 OpenAI 兼容 API，角色头顶显示字幕
- 主动说话：闲置和飞行时有固定台词
- 本地历史对话：保存在 `chat_history.json`，右键 -> 聊天 -> 历史对话 查看

## 开发注意

- 程序有单实例保护，重复启动会提示“桌宠已经在运行了”。
- 冻结 exe 运行时，设置写到 exe 同目录；素材从自解压出的运行时目录读取。
- `settings.json` 可能包含真实 API 密钥，禁止提交到仓库。
- 聊天记录、便携运行时、打包产物均被 `.gitignore` 排除。

---
> Source: [AstrariaX/Angelina-pet](https://github.com/AstrariaX/Angelina-pet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
