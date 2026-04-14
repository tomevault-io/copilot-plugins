## auctionmaster-titan

> > 本文档是 AI 开发助手的首要参考文件。每次开始新任务前，**必须先阅读并按需更新本文档**。

# CLAUDE.md — AuctionMaster 泰坦重铸迁移项目

> 本文档是 AI 开发助手的首要参考文件。每次开始新任务前，**必须先阅读并按需更新本文档**。

## 项目性质

这是一个**迁移项目**：将 AuctionMaster 插件从 WoW 经典怀旧服（Classic Era / TBC Classic, Interface 20501）迁移到**泰坦重铸版本**（Titan Reforged），使其能在新版本客户端上正常运行。

- 原始版本：Classic TBC (Interface: 20501)
- 目标版本：泰坦重铸（Titan Reforged, Interface: 38000）
- 框架：Ace3 全家桶（Release r1390）
- 全局命名空间：`vendor`
- SavedVariables：`VendorDb`, `AuctionMasterMiscDb`

## 文档维护协议

### 强制规则

1. **开发前必读**：每次开始新任务，先完整阅读本文件
2. **开发后必更新**：完成任务或遇到重要发现后，立即更新本文件的相关章节
3. **错误必记录**：任何因 API 变更、版本差异导致的错误和解决方案，必须记录在「已知问题与解决方案」章节
4. **无效尝试必记录**：尝试过但失败的方案，必须记录在「无效尝试记录」章节，避免重复踩坑

### 更新原则

- 只记录**已验证的结论**，不记录猜测
- 保持条目简洁，重点记录「做了什么 → 结果是什么 → 正确做法是什么」
- 过时的信息及时删除或标注

## 架构参考

详见 [ARCHITECTURE.md](doc/ARCHITECTURE.md) — 包含完整的模块结构、依赖关系、设计模式和数据流。

### 关键路径速查

| 用途 | 路径 |
|------|------|
| TOC 入口 | `AuctionMaster_Titan.toc` |
| 第三方库 | `libs/` |
| XML 模板 | `src/xml/` |
| 本地化 | `src/locale/` |
| 核心业务 | `src/main/` |
| 公开 API | `src/api/` |
| 测试 | `src/test/` |
| 插件主入口 | `src/main/Core/Vendor.lua` |
| 拍卖行核心 | `src/main/AuctionHouse/AuctionHouse.lua` |
| 扫描引擎 | `src/main/Scanner/Scanner.lua` |

## 迁移注意事项

详见 [API_CHANGES.md](doc/API_CHANGES.md) — 包含关键差异表和 WoW API 变更明细。遇到新的 API 变更时更新该文档。

## 已知问题与解决方案

详见 [KNOWN_ISSUES.md](doc/KNOWN_ISSUES.md) — 包含所有已解决问题的现象、根因和修复方式。

## 无效尝试记录

> 尝试过但被证明无效的方案，避免后续重复踩坑。

<!-- 格式：
### [日期] 尝试内容
- **目标**：想解决什么问题
- **做法**：具体怎么做的
- **结果**：为什么失败
- **教训**：从中学到了什么
-->

### [2026-02-28] 尝试 1：SetSelectedAuctionItem + PlaceAuctionBid 同一次点击
- **目标**：解决 error 467
- **做法**：在 `_OnOk` 中 `PlaceAuctionBid` 前加 `SetSelectedAuctionItem(ahType, info.index)`
- **结果**：单件购买大概率 467，小概率能通过
- **教训**：仅添加 SetSelectedAuctionItem 不够，可能需要分不同事件调用

### [2026-02-28] 尝试 2：触发原生 StaticPopup_Show("BUYOUT_AUCTION")
- **目标**：借助原生弹窗的 OnAccept 调用 PlaceAuctionBid（Blizzard 代码 = 安全上下文）
- **做法**：`_OnOk` 中不直接调用 PlaceAuctionBid，改为 `StaticPopup_Show("BUYOUT_AUCTION")`
- **结果**：弹窗没出现（被 BuyDialog 遮挡），下单也没成功也没报错
- **教训**：从插件代码调用 StaticPopup_Show 创建的弹窗是 tainted 的，OnAccept 内的受保护函数被静默拦截

### [2026-02-28] 尝试 3：先隐藏 BuyDialog 再弹原生 Popup
- **目标**：解决尝试 2 中弹窗被遮挡的问题
- **做法**：`_OnOk` 中先 `self:Hide()`，再 `StaticPopup_Show("BUYOUT_AUCTION")`
- **结果**：弹窗可见了，但点确定没有任何效果（不成功也不报错）
- **教训**：确认了 taint 是根本原因——插件代码触发的 StaticPopup 整条执行链都是 tainted 的，即使用户物理点击 Accept，OnAccept 中的 PlaceAuctionBid 仍然被拦截

### [2026-02-28] 尝试 4：直接调用 + GetAuctionItemInfo 验证
- **目标**：排除数据过期的可能性，确认 index 和 buyout 数据有效
- **做法**：`_OnOk` 中先 `GetAuctionItemInfo` 验证拍卖品数据，打印详细信息，再 `SetSelectedAuctionItem` + `PlaceAuctionBid`
- **结果**：数据验证通过（`Buying: 青铜锭 index=50 buyout=6800 bid=6800`），仍然 467
- **教训**：拍卖品数据有效，467 不是因为数据过期或 index 错位

### [2026-02-28] 尝试 5：预选分离（SetSelectedAuctionItem 在 ShowForDirectBuy，PlaceAuctionBid 在 _OnOk）
- **目标**：模拟原生 UI 三步流程——选择和购买在不同点击事件中
- **做法**：`ShowForDirectBuy` 打开时调用 `SetSelectedAuctionItem` 预选，`_OnOk` 只调用 `PlaceAuctionBid`
- **结果**：BuyDialog 弹窗不显示了，仍然 467
- **教训**：`SetSelectedAuctionItem` 在 `ShowForDirectBuy` 中调用可能导致 Lua 错误阻止后续 frame:Show()；且分离事件本身不解决 467

### 综合分析
- 原生 UI 购买 100% 成功，插件购买大概率 467
- 467 是服务端错误，不是客户端 taint 错误（ADDON_ACTION_BLOCKED 是另一个问题）
- 所有从插件代码直接/间接调用 PlaceAuctionBid 的方式都会 467
- 原生 UI 的完整流程：点击拍卖行列表行 → SetSelectedAuctionItem → 点击一口价按钮 → StaticPopup_Show("BUYOUT_AUCTION") → 点击确定 → PlaceAuctionBid（三次独立点击，全部在 Blizzard 安全代码中）
- **下一步**：需要阅读 Blizzard_AuctionUI 源码，找出原生 UI 在 PlaceAuctionBid 之前还做了哪些我们没做的事情

## 开发工作流

### 每次任务的标准流程

1. **阅读本文档** — 了解最新状态和已知问题
2. **阅读 doc/ARCHITECTURE.md** — 如果涉及不熟悉的模块
3. **开始开发** — 代码修改
4. **更新本文档** — 记录新发现、新问题、新方案
5. **提交代码** — 确保文档和代码一起更新

### 代码修改原则

- **最小改动**：迁移以「能跑」为第一目标，不做不必要的重构
- **保留原始逻辑**：除非 API 强制变更，否则不改变原有业务逻辑
- **注释变更原因**：对每处因迁移而改动的代码，添加注释说明改动原因，格式：`-- [Titan Migration] 原因说明`
- **libs 目录谨慎修改**：第三方库优先通过升级版本解决兼容问题，避免直接改库代码

## 当前进度

> 已完成事项详见 [PROGRESS.md](doc/PROGRESS.md)。

### 进行中

（暂无）

### 待办

- [ ] 逐模块排查 WoW API 兼容性
- [ ] 游戏内加载测试

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WidgetA)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/WidgetA)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
