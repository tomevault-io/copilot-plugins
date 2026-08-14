## chunui

> Swift Package（swift-tools 6.2 · Swift 5 语言模式 + 默认 MainActor 隔离 + 渐进并发）+ iOS 18.6+ + 唯一依赖 Pow(MIT)

# ChunUI - Monochrome 质感 SwiftUI 设计系统开源包
Swift Package（swift-tools 6.2 · Swift 5 语言模式 + 默认 MainActor 隔离 + 渐进并发）+ iOS 18.6+ + 唯一依赖 Pow(MIT)
定位：从 Zinner（阿奇）提取的可换肤设计系统——宿主只定义 CCColors 调色板即得同质感全家桶；所有 shader/点阵动效随包。

<directory>
Sources/ChunUI/ - 主库 target（99+ Swift 文件；@_exported 系统框架基座，第三方仅 Pow 限 CCButtons）
Sources/ChunUI/Core/ - 包门面（ChunUI.configure 配置入口/CCStrings 文案表/CCExports 导入基座/CCDesign 命名空间/Blur 渐进模糊）
Sources/ChunUI/Theme/ - 设计令牌（CCColors.current 活体调色板 + Color.cc 访问器/CCTypography 三梯度/CCTokens/CCMotion/CCModifiers 玻璃降级边界/CCSheetChrome/ViewFunc）
Sources/ChunUI/Components/ - CC 组件族（按钮/表单/卡片/toast/沉底 alert/原生 sheet/骨架屏/相机/相册/浏览器等 40+ 件）
Sources/ChunUI/Effects/ - 视觉特效（极光/生成中点阵/九分形地板/扫光/粒子消散/全息/元球/丝绸等）
Sources/ChunUI/Shaders/ - 11 个 Metal 着色器 + CCShaders 门面（Bundle.module 取库，禁 ShaderLibrary.default）
Sources/ChunUI/FluidGradient/ - 流体渐变（Oskar Groth FluidGradient 迁入）
Sources/ChunUI/ORB/ - 拟物 AI 球（发光/粒子/波浪/真实阴影）
Sources/ChunUI/Markdown/ - Markdown 渲染
Sources/ChunUI/Presentation/ - 呈现层（AppHelper sheet/alert/toast 门面 + CCNav 导航发现 + TopCardPresenter）
Sources/ChunUI/Extensions/ - 通用系统扩展子集（业务函数已在提取时肃清）
Sources/ChunUI/Icons/ - PikaIcon 组件 + PikaIcons.xcassets（1225 枚模板矢量，9.6M）
Sources/ChunUIDemo/ - 消费者示例 target（GalleryRoot 分区画廊 = 公开 API 面编译测试，被 Example App 消费）
Example/ - 可运行画廊示例 App（xcodegen project.yml → ChunUIGallery.xcodeproj，引用父目录本地包）
docs/ - README 资产（screenshots/ 六张画廊真机截图：主目录/Buttons/Cards/AI Effects/Ambient/Shader Zoo）
skills/ - AI Agent Skill（chunui/：SKILL.md 入口铁律 + usage/ 三份用法 + components/reference.md 全组件参考 + examples/ 五个 Zinner 真实生产页面源码作积木图纸）
</directory>

<config>
Package.swift - SPM 清单：双 target 同构 swiftSettings（.v5 + defaultIsolation MainActor + 渐进并发双 feature）；Pow from 1.0.0
</config>

<rules>
来源同构：本包提取自 Chat0IM/Chat0IM/DesignSystem（上游），上游演进后同步须手动移植；组件名保持 CC 前缀不变。
业务隔离铁律：包内禁止出现宿主类型（MainViewModel/UserManager/AppConfig/ConfigStore）；宿主定制一律走 ChunUI.* 静态接线座与 CCStrings。
资源访问铁律：shader 经 CCShaders（Bundle.module），图标 Image(_, bundle: .module)；禁 ShaderLibrary.default / 裸 Image 取包资产。
公开面纪律：新组件必须 public + 显式 public init，并在 DemoGallery 加一次消费者构造。
验证基线：xcodebuild -scheme ChunUI-Package -destination 'generic/platform=iOS' -derivedDataPath .dd build 双 target 全绿。
</rules>

[PROTOCOL]: 变更时更新此头部，然后检查 CLAUDE.md

---
> Source: [liseami/ChunUI](https://github.com/liseami/ChunUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
