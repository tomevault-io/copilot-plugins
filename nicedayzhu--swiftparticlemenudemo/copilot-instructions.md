## swiftparticlemenudemo

> CS2 粒子菜单插件。基于 SwiftlyS2 框架，用 Source 2 粒子系统和图片材质实现游戏内 HUD 菜单（无 Panorama XML）。

# AGENTS.md — SwiftParticleMenuDemo

CS2 粒子菜单插件。基于 SwiftlyS2 框架，用 Source 2 粒子系统和图片材质实现游戏内 HUD 菜单（无 Panorama XML）。

## 技术栈

- **运行时**: .NET 10.0, C# 13, unsafe blocks
- **框架**: SwiftlyS2 (`SwiftlyS2.CS2` v1.4.2-beta.25), MinimumAPIVersion `1.1.0`
- **构建**: `dotnet build` / `dotnet publish`, 输出到 `build/`
- **脚本**: PowerShell 7+ (pwsh) 必需，不支持 Windows PowerShell 5.x

## 配置分层（关键：不要混改）

| 改什么 | 编辑文件 | 是否需要生成 |
|--------|----------|-------------|
| 菜单树、标题、样式 | `config/authoring/menu_definition.json` | 运行 `tools/build_menu.ps1` 后自动更新 `config/runtime/menu_config.json` |
| 叶子 action 行为 | `src/MenuActions.cs` → `RegisterMenuHandlers()` | 重新 `dotnet build` 并部署 |
| 位置/灵敏度/WASD/网络 | SwiftlyS2 插件目录的 `config.jsonc` | 热重载，部分需重开菜单 |
| 动态文字面板 | `config/authoring/particle_text_ui.json` | 重新生成并编译资源 |
| 粒子行为/渲染器 | `tools/templates/` | 重新生成并编译资源 |

**`resources_src/` 是生成产物，永远不要手动编辑。`tools/templates/` 才是 Source 2 结构源。**

## 开发命令速查

```powershell
# 可视化设计器（推荐）
.\menu-studio.cmd
.\menu-studio-stop.cmd   # 关闭 studio

# 从 config/authoring/menu_definition.json 生成所有配置和资源
pwsh -NoProfile -ExecutionPolicy Bypass -File .\tools\build_menu.ps1

# 仅生成 PNG/MKS/VTEX/VPCF 源文件
pwsh -NoProfile -ExecutionPolicy Bypass -File .\tools\generate_menu_assets.ps1

# 编译 Source 2 资源（需 CS2 Workshop Tools）
pwsh -NoProfile -ExecutionPolicy Bypass -File .\tools\build_source2_assets.ps1

# 完整验证（不部署）
pwsh -NoProfile -ExecutionPolicy Bypass -File .\tools\verify_menu.ps1

# 离线预览（不启动 CS2）
pwsh -NoProfile -ExecutionPolicy Bypass -File .\tools\preview_menu.ps1

# 构建插件
dotnet build .\SwiftParticleMenuDemo.csproj

# 一键构建+部署（正式 Workshop 模式，跳过本地 override）
pwsh -NoProfile -ExecutionPolicy Bypass -File .\build_and_deploy.ps1 -SkipLocalVpk -SkipGameInfo -SkipServerVpk -SkipServerGameInfo
```

## 项目结构要点

```
src/
  SwiftParticleMenuDemoPlugin.cs  — 主插件 partial class（~2600 行）：命令、tick、粒子生成、输入处理
  MenuActions.cs                  — 唯一需要开发者编辑的 action 注册点
  ParticleMenuPluginConfig.cs     — config.jsonc 配置模型
  DynamicTextDemo.cs              — !pspeed 独立 demo
  ParticleAtlasPoc.cs             — !patlas 独立 PoC
  ParticleTextDemo.cs             — !ptext 通用文字面板
tools/
  build_menu.ps1                  — 从作者配置生成运行时菜单配置 + 资源
  generate_menu_assets.ps1        — 生成 PNG/MKS/VTEX/VPCF
  build_source2_assets.ps1        — 编译 Source 2 资源（调用 Valve resourcecompiler）
  verify_menu.ps1                 — 完整验证流水线
  preview_menu.ps1                — 离线 HTML 预览
  _common.ps1                     — 共享 PowerShell 函数
  templates/                      — Source 2 资源模板（结构源，非生成产物）
  menu_designer/                  — Menu Studio 设计器
  assets/                         — 工具链辅助资源
config/
  authoring/                      — Menu Studio 项目、菜单定义和文字面板源配置
  runtime/                        — 生成后或手动维护、随插件发布的运行时配置
  schemas/                        — 作者格式的 JSON Schema
assets/box-sounds/                — Box Sound 原图
docs/images/                      — README 和专题文档图片
build_and_deploy.ps1              — 一键构建部署脚本
```

## 关键架构约束

- **每页固定 6 个 item**（`ItemCount = 6`），这是 VPCF 粒子拓扑决定的，不可改
- 最多支持 64 页，`build_menu.ps1` 自动分页
- 运行时使用稳定 item ID 分发；字符串 option 自动派生 ID
- 有 `children` 的节点仅导航，不调用 handler；只有叶子节点触发回调
- 当前资源命名空间只支持一个自定义菜单；多菜单需独立 namespace
- Particle Text 最多 8 行、每行 21 个 Unicode 字形，使用构建期字形子集
- HUD 模式通过 CP17.z（页码）、CP34.y（hover）、CP33.z/CP34.z（光标）通信

## 独立 Demo 子系统

三个独立 demo 各有自己的命令、会话、Tick、传输过滤和清理生命周期：

| Demo | 入口命令 | 主文件 |
|------|----------|--------|
| 速度 HUD | `!pspeed` | `DynamicTextDemo.cs` |
| Sequence Atlas | `!patlas` | `ParticleAtlasPoc.cs` |
| 通用文字面板 | `!ptext` | `ParticleTextDemo.cs` |

修改菜单逻辑时不要碰这些文件；它们是独立的 partial class 片断。

## 常见陷阱

1. **直接改 `config/runtime/menu_config.json`** — 这是 `build_menu.ps1` 的产物，下次生成会覆盖。应改 `config/authoring/menu_definition.json`。
2. **手改 `resources_src/` 下的 MKS/VTEX/VPCF** — 这些是生成的，改动会在下次生成时丢失。
3. **修改粒子行为时改生成产物而非 `tools/templates/`** — 模板才是源。
4. **用 Windows PowerShell 5.x 而非 pwsh** — 脚本需要 PowerShell 7+，低价版会报错。
5. **添加新 item 时忘记在 `MenuActions.cs` 注册 handler** — 未注册的 item 只会发 chat 提示。
6. **部署时混用 override VPK 和 Workshop** — Workshop 正式部署应跳过所有本地 override 步骤。

## 项目元信息

- **PluginMetadata ID**: `swift_particle_menu_demo`
- **Version**: `1.25.0-stable-ptext-panel`
- **Author**: SkinTools
- **SwiftlyS2 配置段**: `ParticleMenu`
- **runtime config 文件名**: `config.jsonc`

---
> Source: [nicedayzhu/SwiftParticleMenuDemo](https://github.com/nicedayzhu/SwiftParticleMenuDemo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
