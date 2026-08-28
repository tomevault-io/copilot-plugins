## cloa

> 面向所有 AI 编码 agent 的唯一规范与架构文档：硬性规则 + 完整项目描述、架构与目录导航。

# AGENTS.md — Cloa 虚拟衣柜 iOS App

面向所有 AI 编码 agent 的唯一规范与架构文档：硬性规则 + 完整项目描述、架构与目录导航。

## Quick Reference
- **Platform**: iOS 26.0+ only（deployment target: 26.2）— 不向下兼容 iOS 25 及以下
- **Language**: Swift 6
- **UI Framework**: SwiftUI + Liquid Glass（Apple 液态玻璃设计语言）
- **Architecture**: MVVM with `@Observable`
- **Storage**: SwiftData + CloudKit
- **Testing**: Swift Testing + XCTest UI Tests
- **Package Manager**: Swift Package Manager
- **AI**: 火山方舟 Ark（Doubao-Seedream，图像生成 / 多图融合）

## 构建与验证（强制）
- **改完代码即停手，不构建、不运行、不截图、不测试**——一律由用户自行在 Xcode / 模拟器验证
- **禁止调用任何 XcodeBuildMCP 工具**（build / run / test / screenshot / snapshot_ui / 日志捕获等全部包含在内），也不要用 `xcodebuild`、`simctl` 等 shell 命令绕过
- 完成后只需说明改了哪些文件、改动的原理与可调参数，把验证交回用户
- **唯一例外**：用户在当次对话中明确要求「构建 / 运行 / 跑测试 / 截图看看」时才执行；该授权仅对当次请求有效，不延续到后续任务
- 若用户授权构建，Xcode 操作走 XcodeBuildMCP：Build `build_sim_name_proj`、Test `test_sim_name_proj`、Run `build_run_sim_name_proj`、Clean `clean`；单测通过 `test_plan` 或 `only_testing` 参数过滤

## 核心规则
- 项目使用 PBXFileSystemSynchronizedRootGroup（Xcode 16+）：`Cloa/` 目录下的新文件**自动纳入编译**，无需修改 .pbxproj
- 可以修改 .pbxproj（调整 Build Settings、添加 capability 等），但新增 Swift 源文件不需要
- **最低系统 iOS 26.0**，`IPHONEOS_DEPLOYMENT_TARGET = 26.2`；允许无条件使用 iOS 26 专属 API（含 Liquid Glass），禁止为 iOS 25 及以下做 `if #available` 兼容分支
- 禁止使用 UIKit，除非包装无 SwiftUI 替代的系统 API（`UIImage`、`UIGraphicsImageRenderer` 等除外）
- 禁止 force unwrap（`!`）
- 优先 struct 而非 class
- 使用 `@Observable` 而非 `ObservableObject`
- 使用 async/await，不使用 Combine
- 所有 public API 必须有文档注释
- 所有颜色使用 Asset Catalog 语义化 Color Set（Light/Dark Appearance），禁止硬编码颜色值
- API Key 一律经 `AppSecrets` 从本地 `Config/Secrets.xcconfig`（已 gitignore）读取，**禁止硬编码任何密钥**，也禁止把真实 Key 写回源码、文档或提交历史

### UI 规范：Liquid Glass（强制）
- 全局采用 Apple **Liquid Glass**（液态玻璃）设计语言，所有内容都要适配。容器、卡片、工具栏、TabBar、悬浮按钮、fullScreenCover 浮层、试穿结果操作条等一律使用 `.glassEffect(...)` / `GlassEffectContainer`，强调半透明玻璃质感与背景折射、明暗自适应。
- 浮层控件与导航层统一玻璃化；避免用不透明纯色大色块遮挡玻璃层次，让内容在玻璃之下透出。
- 语义化 Color Set 仍是唯一颜色来源，但需保证文字/图标在玻璃材质上于 Light/Dark 下均有足够对比度；玻璃之上的文字一律用语义 token（如 `CloaTextTertiary`），禁止硬编码颜色。
- 因最低系统为 iOS 26，Liquid API 直接使用，无需 `if #available` 回退。

## 架构概览

### 目录导航
```
Cloa/
├── App/                  应用入口与根导航（CloaApp.swift = Schema 注册 / MainTabView.swift = 四 Tab）
├── Core/
│   ├── Models/           SwiftData @Model（ClothingItem / Outfit / VirtualModel / TryOnRecord + 枚举）
│   ├── Services/         图像 I/O、Ark 生成、试穿执行器、感知哈希、展示背景常量
│   ├── Extensions/       UIImage 等扩展（flattened / resized / 像素尺寸对齐）
│   └── Localization/     多语言（AppLanguage.swift）
├── DesignSystem/         语义 token、字体/间距/圆角、Liquid Glass 复用助手（DesignTokens.swift）
├── Features/
│   ├── Scanner/          衣物拍摄 / 裁剪 / 预览 / 编辑（ScannerFlowCoordinator 状态机）
│   ├── Wardrobe/         衣柜网格 / 详情 / 编辑（WardrobeViewModel）
│   ├── Outfit/           搭配列表 / 详情 / 编辑 / 试穿历史（OutfitViewModel）
│   ├── VirtualModel/     虚拟形象两条创建路径 + 列表管理（VirtualModelFlowCoordinator 状态机）
│   └── TryOn/            试穿卡片与执行 UI（异步，与详情页共用 TryOnRunner）
├── Assets.xcassets/      语义 Color Set（唯一颜色真源）
└── Info.plist / Cloa.entitlements
```
- 新增 Swift 文件放入对应 `Features/<模块>` 或 `Core/<层>`，因 `PBXFileSystemSynchronizedRootGroup` 自动纳编，无需改 .pbxproj。

### 数据层
- **SwiftData models** (`Core/Models/`): `ClothingItem`, `Outfit`, `VirtualModel`, `TryOnRecord`
- Schema 注册在 `CloaApp.swift` — 新增 `@Model` 必须同步更新那里的 `Schema([...])` 数组
- 图像以文件存 `Documents/`，SwiftData 只存相对路径
- `ImageStorageService.shared` 是所有图像 I/O 的单一网关（Wardrobe PNG、VirtualModels PNG、TryOn JPEG）
- **服饰单品图不做抠图**：拍摄/导入后由用户裁剪，原样落盘；展示时统一垫在 `DisplayBackground.color`（`#F2F2F3`）卡片上（见 `ClothingHeroImage`），与 AI 生成图背景保持一致。该色号唯一真源是 `ImageBackground.swift` 的 `DisplayBackground`，AI 提示词模板同样引用它
- **虚拟形象「Use My Photo」直接使用拍摄/相册选择的整身原图**作为虚拟形象，**不调用 Ark、不替换背景、无 AI 提示词**（用户负责纯色背景取景）；确认预览后原样保存
- **虚拟形象支持创建多个**：Settings →「Manage Virtual Models」进入 `VirtualModelListView`（右上角「+」新建、左滑删除、仅展示预览图）。保存走 `VirtualModelSaving.save`（不再删除已有记录）；TryOn / Settings 取 `virtualModels.first`
- **所有 AI 生成图尺寸与原图完全一致**：调用 Ark 时按参考图宽高比设置输出 `size`（避免裁切）并在接口约束内取最高分辨率（清晰度最高），返回后再用 `UIImage.resized(toPixelSize:)` 对齐到原图**精确像素尺寸**（见 `VolcArkImageService.matchedSize` / `maxResSize` 与 `ImageBackground.swift`）

### AI 服务层 (`Core/Services/`)
云端图像生成走 **火山方舟 Ark**（VolcEngine Ark），不再使用 KlingAI / 阿里 Dashscope。**服饰扫描全程本地、不调用 Ark**；**虚拟形象「Use My Photo」直接用原图、也不调用 Ark**；仅虚拟形象「参数合成」（场景 1B）与试穿（场景 2）调用 Ark。

| Service | API | Purpose |
|---|---|---|
| `VolcArkImageService` | Ark `POST /api/v3/images/generations` | 云端图像生成入口：虚拟形象参数合成（场景 1B）+ 虚拟试穿（多图融合），**同步返回**；输出 `size` 按参考图宽高比动态计算，结果对齐原图精确像素。（「Use My Photo」不经此服务，直接用原图） |

- **Base URL**: `https://ark.cn-beijing.volces.com/api/v3`
- **Endpoint**: `POST /images/generations`（同步接口，无需轮询、无需 OSS 上传）
- **Auth**: `Authorization: Bearer <ARK_API_KEY>`。**Key 不写进源码与文档**：本地 `Config/Secrets.xcconfig`（已 gitignore）→ xcconfig 构建设置 → `Info.plist` 的 `$(ARK_API_KEY)` → 运行时 `AppSecrets.arkAPIKey` 读取（支持同名环境变量覆盖）。模板见 `Config/Secrets.xcconfig.example`
- **Model ID（可切换）**: 由 `VolcArkImageService.activeModel` 决定，当前 `.seedream50pro`（`doubao-seedream-5-0-pro-260628`）；回退改回 `.seedream45`（`doubao-seedream-4-5-251128`）即可，两者同一 API Key。两代差异（输出像素范围、是否支持 `sequential_image_generation`、最多参考图张数）已收敛进 `Model` 枚举，请求构造与尺寸计算按当前模型取值
- **多图输入**: `image` 字段传字符串或字符串数组，支持 base64 data URL（`data:image/jpeg;base64,...`）或公网 https URL；文生图省略 `image`，图生图 / 多图融合传入参考图（最多 10 张）
- **输出**: `response_format: "url"` 返回 `data[].url`，或 `"b64_json"` 返回 base64；`watermark: false` 去水印
- **尺寸/清晰度**: `size` 传 `"宽x高"` 像素串（非固定 `"2K"`），按参考图宽高比动态计算并在接口约束内取最高分辨率——**总像素范围随模型而定**（4.5：[3,686,400, 16,777,216]；5.0 pro：[921,600, 4,624,220]，见 `Model.outputPixelRange`），单边 ≤ 6000、宽高比 ∈ [1/16, 16] 两代通用。生成后本地 `resized(toPixelSize:)` 对齐原图精确像素，保证「尺寸与原图完全一致 + 不裁切 + 脸部清晰」。输入参考图 JPEG **仅在超过接口硬上限（最长边 > 6000px）时才等比缩放，典型手机照片逐像素原样上传**，质量从 0.95 起、体积上限 24MB（接口允许 6000px / 30MB / 36MP），最大化保留人脸细节；相册导入用 `loadTransferable(Data.self)` 读原始数据、不裁剪
- **请求体骨架**:
  ```json
  {
    "model": "doubao-seedream-4-5-251128",
    "prompt": "<场景指令模板 + 动态内容>",
    "image": ["data:image/jpeg;base64,...", "data:image/jpeg;base64,..."],
    "size": "<按参考图宽高比动态计算，如 3072x4096>",
    "sequential_image_generation": "disabled",   // 仅 4.5 发送；5.0 pro 为单图接口、不带此字段
    "response_format": "url",
    "watermark": false
  }
  ```
- `VolcArkImageService` 为 `actor`（Swift 6 安全）。API Key 经 `AppSecrets` 从本地配置读取。

**移除的服务**：`KlingFaceSwapService`、`DashscopeUploadService`、`AiTryOnService`（含 JWT/HMAC、OSS 上传、异步轮询逻辑）。因 Ark 同步返回且支持 base64 内联，图片无需先上传获取公网 URL，试穿流水线简化为：**编码模特图 + 单品图为 base64 → 组织组合指令 → 单次请求 → 直接得到结果图**。单品图为「穿在他人身上」的照片时，靠提示词约束模型只取对应 tag 的那一件、忽略人脸/手臂/手表/其它品类衣物（不做单品图预提取）。

### AI 提示词规范（约定，非具体文案）
> 具体提示词文案属实现细节，**只维护在代码里，不在本文档抄录**。修改提示词请直接改源码，勿在文档同步整段文案。

- **单一 `prompt` 字段**：Ark 图像模型无独立 system/user 角色，每个使用场景对应一个固定「指令模板」（等价 system prompt），运行时与动态内容拼接后作为 `prompt` 发送。
- **模板真源（source of truth）**：
  - 场景 1B（人脸 + 身材参数合成全身模特）与场景 2（多单品虚拟试穿）的模板与动态指令拼接，均在 `VolcArkImageService.swift`。
  - 性别描述文案（`genderText`）内联在 `VolcArkImageService.swift`；身材由身高/体重参数自然推断，无独立体型枚举与映射服务。
  - 展示背景色号只走 `ImageBackground.swift` 的 `DisplayBackground`（`#F2F2F3`），提示词内部**不写字面色号**（避免模型渲染成背景文字）。
- **生成路径（业务导航）**：
  - Use My Photo（**非 AI 路径，无编号**）：整身原图直接落盘，**不调用 Ark、无提示词**。
  - 场景 1B — 参数合成：人脸为身份基准 + 身材参数（身高/体重）→ 全新全身模特图，调用 Ark。
  - 场景 2 — 虚拟试穿：图 1 恒为模特图，其后按固定类别顺序附加单品图，执行「精确局部替换」，调用 Ark。**串图防护（单品图里那个人的脸/手臂/手表/其它品类衣物不得进入结果）与彻底替换（原衣物元素不残留、不与新单品混合）全部靠提示词约束**，不做单品图预提取。
- **图序约定（改试穿逻辑必须遵守）**：图 1 固定模特图；单品按固定类别顺序映射为图 2、图 3……（上衣 → 外套 → 连体装 → 下装 → 鞋 → 包 → 帽子 → 眼镜 → 项链 → 耳环 → 其它，见 `categoryOrder`）。同一类别多件按用户选择顺序（视为由内到外的叠穿次序）用稳定排序保序。动态组合指令中的「图 N」编号必须与实际传入 `image` 数组下标（从 1 起）严格对应。
- **场景 2 类别语义**：`ClothingCategory` 由原「上衣」拆分出 `outerwear`（外套，最外层叠加、保留内搭）与 `onePiece`（连体装/连衣裙，同时替换上衣与下装）；试穿置换分三类——脱旧换新（上衣/下装/鞋/帽子/眼镜）、层叠或覆盖（外套/连体装）、新增佩戴（包/项链/耳环），`other`/`unknown` 交模型自行判断部位。核心原则：图1为目标底图，除可见的被换衣物外逐像素冻结；只换画面中心主体、只换实际入镜的部位（部位缺失则该件跳过、全缺则原样返回）；单品含图案/logo 整体等比缩放贴合；单品图若为「穿在他人身上」的照片，只提取该类别那一件。

### Feature 模块

- **Scanner**: `ScannerFlowCoordinator`（状态机：`camera → preview → edit`），拍摄/导入 → 裁剪 → 确认预览 → 填标签保存，全程本地无网络调用
- **Wardrobe**: `WardrobeViewModel` 通过 SwiftData 查询 `ClothingItem`；grid + detail + edit 视图
- **Outfit**: `OutfitViewModel` 管理 `Outfit`（与 `ClothingItem` 多对多）；`OutfitListView` → `OutfitDetailView`（顶部最近试穿大图 + 单品条 + History 条）→ `OutfitTryOnHistoryView`（全部记录、左滑删除带确认）
- **VirtualModel**: `VirtualModelFlowCoordinator` 状态机驱动两条路径 — Mode A（参数合成）`setup → faceCapture → capturePreview → generating → result`，Mode B（Use My Photo）`bodyCapture → capturePreview →（确认即保存）`。两条路径拍摄/选择后都有 `CapturePreviewView` 确认预览；Mode A 生成后再有 `VirtualModelResultView`（使用/丢弃）。人脸/整身拍摄页均无取景引导框、仅底部提示，且支持相册上传。**支持创建多个**虚拟形象：`VirtualModelListView` 列表（右上 `+` 新建、左滑删除、仅展示预览图），从 Settings →「Manage Virtual Models」进入；保存统一走 `VirtualModelSaving.save`（不删旧记录）
- **TryOn**（异步生成，单品与搭配同构）：详情页点 Try On 后**不跳转任何全屏等待页**——先定形象（无 → 提示去 Settings 创建；1 个 → 直接用；多个 → `VirtualModelPickerView` 网格选择），随即插入一条 `TryOnRecord` 并交给 `TryOnRunner.run(record:modelPath:items:onError:)` 后台生成，用户可离开页面。
  - `TryOnRunner`（`Core/Services/TryOnRunner.swift`）是**单品试穿与搭配试穿唯一的执行入口**：置 `pending` → 读图 → `running` → Ark `tryOn` → 存图置 `succeeded`；`CancellationError` 记 "Cancelled"、其它异常记 `errorMessage` 并回调 `onError` 弹一次提示。任务句柄登记到 `TryOnTaskCenter`（进程级，离开页面再回来仍可取消）；App 被杀留下的 `pending`/`running` 僵尸记录由 `TryOnTaskCenter.failInterruptedRecords(in:)` 在启动时判失败。
  - UI 约定：同一目标同一时刻只允许一个未完成任务（`hasRunningRecord` 禁用 Try On 按钮并显示 `ProgressView`）；生成中卡片带 Cancel（中断但**保留记录**，标记为失败 "Cancelled"）；失败卡片带 Retry（优先复用记录里的 `virtualModelPath`，形象已删则重新选）；删除记录 / 删除单品或搭配前先 `TryOnTaskCenter.cancel`，避免回写到已删除对象。

### DesignSystem
- **Liquid Glass**: 全局 UI 采用 Apple 液态玻璃（iOS 26+）。玻璃用于「浮于内容之上的功能控件层」（徽章、悬浮操作条、悬浮/主 CTA 按钮），**不铺在纯色背景的内容卡片上**（会发灰），也避免玻璃叠玻璃。TabView / 导航栏 / 工具栏在 iOS 26 自动玻璃化，无需手动处理。
  - 复用助手（`DesignTokens.swift`）：`.cloaGlassCard(cornerRadius:)`（圆角矩形玻璃）、`.cloaGlassCapsule()`（胶囊玻璃，用于图片上的徽章/悬浮胶囊）。
  - 主 CTA 用 `.buttonStyle(.glassProminent).tint(Color.cloaPrimary)`；次级悬浮按钮用 `.buttonStyle(.glass)`。
  - 玻璃之上只用语义色 token，保证 Light/Dark 对比度。详见「核心规则 → UI 规范：Liquid Glass」。
- 语义色 token（`Color.cloaPrimary`、`cloaBackground`、`cloaSurface`、`cloaTextPrimary`、`cloaTextSecondary`、`cloaTextTertiary`、`cloaBorder`）来自 Assets.xcassets Color Set — 不要在代码里重新定义
- Typography: `Font.cloaTitle`、`cloaHeadline`、`cloaBody`、`cloaCaption` 等（定义在 `DesignTokens.swift`）
- Spacing: `Spacing.xs/sm/md/lg/xl/xxl` | CornerRadius: `CornerRadius.sm/md/lg/xl`
- 衣柜色板: `WardrobeColor.allColors`（字符串 ID 如 `"black"`、`"blue"`，存进 SwiftData）

### Navigation Pattern
- Root: `MainTabView`，四个 Tab（Wardrobe、Outfits、Try On、Settings）
- Feature flows（Scanner、VirtualModel）以 `fullScreenCover` 呈现
- 试穿不再有全屏流程页：生成在详情页内异步进行，`VirtualModelPickerView` / `ImageViewerView` 才是 `fullScreenCover`

## 数据策略
- 所有数据存储在本地 SwiftData + iCloud 同步（CloudKit 待 Developer Portal 配置后启用）
- Debug 构建在 schema 不兼容时自动销毁并重建 SQLite store
- 不使用任何自建后端服务器；本地数据仅在设备/iCloud 间同步
- 例外：AI 图像生成会将模特图与单品图（base64）发送至火山方舟 Ark 以生成结果，属必要的第三方处理

## 提交约定
- 除非用户明确要求，不主动 commit / push；在默认分支上先建分支

---
> Source: [Aerdax/Cloa](https://github.com/Aerdax/Cloa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
