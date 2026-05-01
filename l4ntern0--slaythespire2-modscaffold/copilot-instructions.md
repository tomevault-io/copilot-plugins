## slaythespire2-modscaffold

> 本文件是 AI Agent（如 OpenCode）在此脚手架项目中的工作规范。

# AGENTS.md - AI Agent 规范

本文件是 AI Agent（如 OpenCode）在此脚手架项目中的工作规范。

**在开始开发之前，必须先阅读 [`rules.md`](rules.md)。**

## 项目概览

这是一个 **Slay the Spire 2 Mod 脚手架**，使用 C# + Harmony 进行 Mod 开发。

- **开发规范**: `rules.md` / `rules_en.md` — 必读，约束所有开发行为
- **入口文件**: `src/ModEntry.cs` — 禁止修改
- **Hook 文件**: `src/Hooks/` 目录 — 按功能模块拆分文件
- **清单文件**: `src/com.vibecoding.sts2mod.json` — Mod 元数据（id、名称、版本等）
- **项目文件**: `src/Sts2ModScaffold.csproj` — .NET 10 类库

## 执行流程

### 0. 初始化 Mod 信息（首次必须执行）

**每个新 Mod 的第一步，必须先修改以下文件中的 Mod 身份信息：**

1. `src/com.vibecoding.sts2mod.json` — 修改 `id`（必须与文件夹名/DLL名一致）、`name`、`author`、`description`、`pck_name`
2. `src/ModEntry.cs` — 修改 `ModId`（与 manifest 的 `id` 一致）和 `ModName` 常量
3. 运行 `tools/rename-scaffold.ps1 -NewModName "你的Mod名"` 自动重命名 csproj、namespace、manifest 文件

**规则：**
- `id` 必须与文件夹名、DLL文件名、PCK文件名一致（详见 `rules.md`）
- `ModEntry.cs` 中的 `ModId`（Harmony 补丁 ID）必须与 manifest 的 `id` 一致
- 如果不执行此步骤，Mod 将以 "Sts2ModScaffold" 身份加载，可能与其他 Mod 冲突

### 1. 理解需求

当用户描述一个 Mod 需求时：
1. 使用 **ILSpy MCP** 反编译 `sts2.dll`，找到目标 Hook 的位置和签名
   - `decompile_type`: 查看类结构
   - `decompile_method`: 查看方法签名
   - **不要猜测签名，必须用 MCP 验证**
2. 查阅 `Modding-Tutorial/` 目录中的示例
3. 在 `docs/plans/` 下创建实现计划

### 2. 制定计划

**必须先在 `docs/plans/` 下创建计划文件，再编写代码。**

文件命名：`docs/plans/[功能名].md`

### 3. 编写代码

1. 编辑 `src/Hooks/` 下的 Hook 文件
2. 代码必须与计划一一对应
3. 使用 `Logger.Log()` 记录关键操作

### 4. 代码 Review

编写完代码后，逐条对照计划检查：
- [ ] 每个计划中的 Hook 都已实现
- [ ] 方法签名与 ILSpy MCP 反编译结果一致
- [ ] 代码逻辑与计划一致

### 5. 编译与安装

```powershell
install-mod.bat
```

### 6. 启动游戏并自动测试

**Agent 必须自行完成全部测试流程，使用脚本和 STS2MenuControl + STS2MCP 操作游戏。**

**前置条件：Steam 必须已启动并登录，否则游戏无法启动。启动前先检查 Steam 状态。**

#### 6.1 关闭已运行的游戏

```bash
powershell -ExecutionPolicy Bypass -File "tools\close_sts2.ps1"
```

#### 6.2 清除旧日志

```bash
powershell -Command "Remove-Item \"$env:APPDATA\SlayTheSpire2\logs\mod_log.txt\" -Force -ErrorAction SilentlyContinue; Remove-Item \"$env:APPDATA\SlayTheSpire2\logs\godot.log\" -Force -ErrorAction SilentlyContinue"
```

#### 6.3 启动游戏

**必须使用脚本启动游戏：**

```bash
powershell -ExecutionPolicy Bypass -File "tools\launch_sts2.ps1"
```

#### 6.4 等待游戏启动

```bash
powershell -ExecutionPolicy Bypass -File "tools\wait_sts2.ps1"
```

#### 6.5 使用 STS2MenuControl 控制主菜单

HTTP API on `localhost:8081`：

```bash
# 打开角色选择（单人）
curl -s -X POST http://localhost:8081/api/v1/menu -H "Content-Type: application/json" -d "{\"action\":\"open_character_select\"}"

# 选择角色
curl -s -X POST http://localhost:8081/api/v1/menu -H "Content-Type: application/json" -d "{\"action\":\"select_character\",\"option_index\":0}"

# 开始游戏
curl -s -X POST http://localhost:8081/api/v1/menu -H "Content-Type: application/json" -d "{\"action\":\"embark\"}"

# 创建多人房间（LAN）
curl -s -X POST http://localhost:8081/api/v1/menu -H "Content-Type: application/json" -d "{\"action\":\"open_multiplayer_host\",\"mode\":\"lan\",\"max_players\":4}"

# 标记准备就绪
curl -s -X POST http://localhost:8081/api/v1/menu -H "Content-Type: application/json" -d "{\"action\":\"set_ready\"}"
```

#### 6.6 使用 STS2MCP 操作游戏内

HTTP API on `localhost:15526`：

```bash
# 获取游戏状态
curl -s http://localhost:15526/api/v1/singleplayer

# 出牌
curl -s -X POST http://localhost:15526/api/v1/singleplayer -H "Content-Type: application/json" -d "{\"action\":\"combat_play_card\",\"card_index\":0,\"target\":\"ENEMY_ID\"}"

# 结束回合
curl -s -X POST http://localhost:15526/api/v1/singleplayer -H "Content-Type: application/json" -d "{\"action\":\"combat_end_turn\"}"

# 选择地图节点
curl -s -X POST http://localhost:15526/api/v1/singleplayer -H "Content-Type: application/json" -d "{\"action\":\"choose_map_node\",\"index\":0}"
```

#### 6.7 关闭游戏

```bash
powershell -ExecutionPolicy Bypass -File "tools\close_sts2.ps1"
```

等待进程完全退出（最多 10 秒）。

### 7. 检查日志确认 Mod 加载成功

```bash
powershell -ExecutionPolicy Bypass -File "tools\read_sts2_logs.ps1"
```

需要确认两点：
1. **godot.log** 中有 `Finished mod initialization for` — Mod 被加载
2. **mod_log.txt** 中有 `[Hook]` 开头的日志 — Hook 被触发

## MCP 工具使用

### ILSpy MCP — 代码反编译

用于理解游戏内部实现和验证 Hook 签名。**不要猜测签名，必须用 MCP 验证。**

所有工具都需要传 `assemblyPath` 参数，指向 `references/sts2.dll`。

- `decompile_type(assembly_path, type_name)` — 查看类结构
- `decompile_method(assembly_path, type_name, method_name)` — 查看具体方法签名
- `get_type_members(assembly_path, type_name)` — 快速查看成员列表
- `list_assembly_types(assembly_path)` — 列出程序集中的所有类型

### STS2MenuControl — 主菜单控制（端口 8081）

> 独立仓库：https://github.com/L4ntern0/STS2-MenuControl（install 时从 GitHub 克隆）

用于在主菜单中操作（角色选择、开始游戏、多人模式等）。**Agent 必须使用此 API 进行主菜单操作。**

| Action | 说明 |
|--------|------|
| `open_character_select` | 打开角色选择（单人模式） |
| `open_multiplayer_host` | 创建多人房间 |
| `select_character` | 选择角色 |
| `embark` | 开始新游戏 |
| `continue_run` | 继续存档 |
| `abandon_run` | 放弃存档 |
| `open_timeline` | 打开时间线 |
| `set_ready` / `set_unready` | 多人准备状态 |
| `get_lobby_status` | 查询房间信息 |
| `confirm_modal` / `dismiss_modal` | 弹窗交互 |
| `close_main_menu_submenu` | 关闭子菜单 |
| `return_to_main_menu` | 返回主菜单 |

### STS2MCP — 游戏内控制（端口 15526）

用于游戏中操作（战斗、地图、事件等）。**Agent 必须使用此 API 进行游戏内操作。**

| Action | 说明 |
|--------|------|
| `combat_play_card` | 出牌（需要 card_index，可能需要 target） |
| `combat_end_turn` | 结束回合 |
| `choose_map_node` | 选择地图节点 |
| `choose_event_option` | 选择事件选项 |
| `proceed` | 继续前进（奖励/事件后） |
| `select_card` / `confirm_selection` | 卡牌选择 |
| `skip_card_reward` | 跳过卡牌奖励 |
| `cancel_selection` | 取消选择 |
| `rewards_claim` | 领取奖励 |
| `rest_choose_option` | 休息站选项 |
| `shop_purchase` | 商店购买 |
| `deck_select_card` / `deck_confirm_selection` | 卡组编辑 |

## 文件规则

- **只修改** `src/Hooks/` 目录（和新建的计划文件）
- **首次使用时修改** `src/com.vibecoding.sts2mod.json`、`src/ModEntry.cs` 中的 Mod 身份信息
- **禁止修改** `src/ModEntry.cs` 的代码逻辑（Initialize 方法和 Logger）
- **禁止修改** `src/Sts2ModScaffold.csproj`
- **Mod 清单** 在 `src/*.json`
- **项目代码** 全部在 `src/` 目录下

## 编译与安装

### 首次安装（环境配置）

```powershell
.\install.bat
```

自动安装 .NET 10 SDK、.NET 8 运行时、Godot、ILSpy、STS2MCP、STS2MenuControl，配置 MCP 服务器。只需运行一次。

### 日常开发

```powershell
install-mod.bat
```

重新构建所有 Mod（脚手架 + STS2MenuControl）并安装到游戏。可加参数：`install-mod.bat "游戏路径"`

### 卸载

```powershell
uninstall-mod.bat
```

从游戏目录移除脚手架 Mod、STS2MCP 和 STS2MenuControl。

### 辅助脚本

| 脚本 | 用途 |
|------|------|
| `tools/rename-scaffold.ps1` | 重命名脚手架（自动修改 csproj、namespace、manifest） |
| `tools/launch_sts2.ps1` | 启动游戏 |
| `tools/close_sts2.ps1` | 关闭游戏 |
| `tools/wait_sts2.ps1` | 等待游戏启动（轮询检测） |
| `tools/read_sts2_logs.ps1` | 查看日志 |

## Mod 文件结构

游戏加载 Mod 时的结构要求见 [`rules.md`](rules.md) 规则4。

## 错误排查

### Mod 未加载
- 检查 `godot.log` 中的 `ERROR` 和 `WARNING`
- 确认清单中的 `id` 与文件夹名、DLL文件名一致
- 确认 `has_dll: true` 和 `has_pck: true` 已设置
- 确认 `mod_manifest.json` 包含 `pck_name` 字段
- 确认 DLL 和 PCK 文件都在游戏 `mods/` 目录的子文件夹下
- 注意：首次启动新 Mod 时游戏可能提示 "mods warning"，需在游戏中确认后才加载

### Hook 不生效
- 用 ILSpy MCP 验证方法签名是否匹配
- 检查 `[HarmonyPatch]` 属性是否正确
- 检查 `Logger.Log()` 是否在 `mod_log.txt` 中出现（确认 Hook 被调用）

### 编译失败
- 确认已运行过 `install.bat`（自动安装 .NET 10 SDK）
- 检查 `references/` 下是否有 `sts2.dll`、`0Harmony.dll`、`GodotSharp.dll`
- 检查 csproj 中的 `GameDir` 路径是否指向正确的游戏目录

---
> Source: [L4ntern0/SlaytheSpire2-ModScaffold](https://github.com/L4ntern0/SlaytheSpire2-ModScaffold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
