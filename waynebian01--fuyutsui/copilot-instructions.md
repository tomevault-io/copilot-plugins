## fuyutsui

> 这个仓库是一个 World of Warcraft Retail Lua AddOn。核心目标是在屏幕顶部绘制 255 个极窄色块，把玩家、目标、队伍、光环、法术冷却、配置开关等游戏状态编码成像素颜色，供外部程序读取。

# AGENTS.md

这个仓库是一个 World of Warcraft Retail Lua AddOn。核心目标是在屏幕顶部绘制 255 个极窄色块，把玩家、目标、队伍、光环、法术冷却、配置开关等游戏状态编码成像素颜色，供外部程序读取。

## 仓库结构

- `Fuyutsui.toc` 是 AddOn 入口和加载顺序。改新增 Lua 文件时必须同步这里。
- `embeds.xml` 与 `libs/` 加载 Ace3、LibStub、LibRangeCheck 等库。通常不要改 vendored libs。
- `core/core.lua` 创建全局 `Fuyutsui` AceAddon，注册 SavedVariables、选项、斜杠命令和事件。
- `core/config.lua` 是静态配置表：法术列表、事件常量、英雄天赋、难度、boss、能量类型、动作条、按键编码、角色映射等。
- `core/block.lua` 管理顶部像素条和计数条。`Fuyutsui:CreatTexture(index, b)` 是主要输出 API。
- `core/macro.lua` 创建 SecureActionButton 宏并绑定预设按键。
- `core/keybinds.lua` 扫描动作条按键，把 spellId 映射到 key/keycode/icon/name。
- `core/auras.lua` 定义职业光环元数据，并根据冷却、施法成功、图标、覆盖法术、屏幕提示等事件维护光环状态。
- `core/quickbutton.lua` 创建游戏内快速切换按钮，操作爆发、AOE、输出模式、药水开关。
- `class/*.lua` 按职业声明 `Fuyutsui.ClassBlocks` 和 `Fuyutsui.MacrosList`。每个文件开头会用 `UnitClassBase("player")` 过滤非当前职业。
- `main.lua` 是运行时主逻辑：加载职业配置、更新状态、处理 WoW 事件、按帧刷新色块。
- `gui.lua` 定义 AceConfig 配置和 `/fu gui` 信息窗口。
- `Keymap.md` 记录键位编码对照。

## 加载顺序很重要

`Fuyutsui.toc` 当前顺序大致是：

1. `embeds.xml` 和库。
2. `core/core.lua` 先创建 `Fuyutsui`、默认配置、基础表。
3. `core/quickbutton.lua`、`core/config.lua`、`core/block.lua`、`core/macro.lua`、`core/keybinds.lua`、`core/auras.lua` 继续给 `Fuyutsui` 挂方法和数据。
4. 所有 `class/*.lua` 依次加载，但只有当前玩家职业文件真正生效。
5. `main.lua` 消费前面的表并注册运行时逻辑。
6. `gui.lua` 最后覆盖/扩展 `Fuyutsui.options`，并提供 GUI。

不要把依赖 `Fuyutsui` 的代码放到 `core/core.lua` 之前。新增模块时尽量放到它依赖的数据之后、被消费之前。

## 核心状态和输出约定

`Fuyutsui` 是全局 AddOn 对象。常用表：

- `Fuyutsui.state`：玩家和运行时状态，如职业、专精、战斗、施法、目标、物品数量等。
- `Fuyutsui.blocks`：当前专精解析后的色块索引映射。
- `Fuyutsui.target`、`focus`、`nameplate`、`group`、`groupList`：单位状态缓存。
- `Fuyutsui.db.char`：角色级开关，包含 `aoeMode`、`cooldowns`、`dpsMode`、`delay`、`potion`、快速按钮位置等。
- `Fuyutsui.db.profile`：配置 profile，目前主要是示例输入。

顶部静态色块由 `core/block.lua` 创建，共 255 个。`Fuyutsui:CreatTexture(i, b)` 会把第 `i` 格写成 `(r=0, g=i/255, b=b, a=1)`。很多逻辑把业务值先归一化到 `0..1`，实际读取端再按 255 反解。

计数条由 `Fuyutsui:CreateAutoLayoutBar(valueType, minValue, maxValue, spellId)` 自动排布，支持 `castCount` 和 `charge`。如果改计数条高度或位置，注意它和主色块条都锚在屏幕顶部。

## 职业表格式

每个 `class/*.lua` 设置：

```lua
Fuyutsui.ClassBlocks = {
    [specIndex] = {
        powerType = "MANA", -- 可选
        [1] = { type = "block", name = "锚点" },
        [25] = { type = "aura", name = "圣光灌注", auraName = "圣光灌注", showKey = "remaining" },
        [38] = { type = "spell", spellId = 20473, name = "神圣震击" },
        [39] = { type = "spell", spellId = 20473, name = "神圣震击", charge = true },
        [70] = { type = "group", num = 6, healthPercent = 1, role = 2, dispel = 3 },
    },
}
```

`main.lua:Fuyutsui:loadPlayerBlocks(specIndex)` 会解析：

- `type = "block"`：写入 `blocks.state[name] = index`。
- `type = "aura"`：写入 `blocks.auras[index] = entry`，由 `core/auras.lua` 刷新。
- `type = "spell"`：写入 `blocks.spells[spellId].index` 或 `.charge`，由冷却逻辑刷新。
- `type = "group"`：写入 `blocks.groups`，用于队伍成员状态块。
- 非 `type` 字段会被跳过，例如 `powerType`。

新增 block 名称时，要确认 `main.lua` 中有对应 `blocks.state["名称"]` 的更新逻辑；否则它只会存在映射，不会自动写像素。

## 光环约定

`core/auras.lua` 顶部的 `auras[classId]` 是光环元数据来源。职业表里的 `auraName` 必须和这里的键名一致。

常见输出：

- `showKey = "remaining"`：剩余时间。
- `showKey = "count"`：层数。
- `showKey = "isIcon"`：图标/触发状态。

如果新增一个职业光环，通常要同时：

1. 在 `core/auras.lua` 对应职业下定义光环。
2. 在对应 `class/*.lua` 专精中加入 `type = "aura"` 色块。
3. 确认光环由玩家真实 aura、冷却事件、施法成功、图标事件或 overlay 事件中的哪一种驱动。

## 宏和按键

职业文件里的 `Fuyutsui.MacrosList` 包含：

- `dynamicSpells`：按队伍/团队目标展开，一组占 30 个键。
- `staticSpells`：普通 `/cast spell`。
- `specialSpells`：完整宏文本，优先于同序号 static spell。

`core/macro.lua` 使用 SecureActionButton 和 `SetOverrideBindingClick` 创建绑定。战斗中不能创建或修改安全按钮，所以相关逻辑必须避开 `InCombatLockdown()`。

`core/keybinds.lua` 扫描动作条 1..180 槽位，读取 `core/config.lua` 的 `actionBars` 和 `keymap`。如果新增键位编码，优先改 `Fuyutsui.keymap`，并同步 `Keymap.md`。

## 命令和配置

斜杠命令由 AceConsole 注册，不要再手写 `SlashCmdList`。

常用命令：

- `/fu` 或 `/fuyutsui`：默认打开 GUI。
- `/fu help`：命令帮助。
- `/fu options` 或 `/fu config`：打开设置。
- `/fu cd [on|off]`：爆发开关。
- `/fu aoemode [auto|aoe]`：AOE/单体模式。
- `/fu dpsmode [manual|assistant]`：手写逻辑/官方一键辅助模式。
- `/fu potion [on|off]`：爆发药水开关。
- `/fu delay [秒]`：临时 delay 标志。

改 `db.char` 开关后，要调用对应 `Switch*` 或 `updatePlayerConfig()`，让 SavedVariables、聊天提示、快速按钮和顶部色块保持同步。

## 事件和刷新节奏

`Fuyutsui:OnEnable()` 注册大量 WoW 事件，并启动 `StartFrameUpdates()`。

`main.lua:Fuyutsui:OnUpdate(elapsed)` 分两层刷新：

- 每帧：玩家施法/引导/蓄力、目标/焦点施法、队伍血量范围、光环、目标光环数量。
- 每 0.2 秒：防御光环、法术冷却、单位光环块、玩家辅助、符文、目标距离、敌人数、物品冷却。
- 每 1 秒：战斗时间。

事件处理函数通常只更新受影响状态，然后调用对应写色块函数。新增高频逻辑前先确认是否真的需要每帧，避免在战斗中制造额外开销。

## 开发注意事项

- 这个项目运行环境是 WoW Lua，不是标准 Lua；很多 API 只能在游戏内测试。
- 保持 Lua 5.1/WoW 兼容写法，不要使用新版 Lua 语法。
- 文件里已有大量中文注释和中文 block 名称；新增业务名称时沿用中文，保证和现有表键一致。
- 函数名里已有拼写如 `CreatTexture`、`creatColorCurve`，不要为了拼写重构而破坏调用点。
- 读取或修改 `core/block.lua` 前注意当前工作区可能已有未提交改动，避免覆盖用户修改。
- 不要格式化整个仓库或 vendored `libs/`。
- 修改加载文件后检查 `Fuyutsui.toc`。
- 修改职业色块后用 `/reload` 在游戏内验证，并打开 `/fu gui` 查看索引映射是否符合预期。
- 无法在普通 shell 中完整跑 WoW API 测试；最多做静态检查、搜索引用和语法层面的人工复核。

---
> Source: [waynebian01/Fuyutsui](https://github.com/waynebian01/Fuyutsui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
