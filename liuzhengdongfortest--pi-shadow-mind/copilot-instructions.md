## pi-shadow-mind

> - 默认使用简单、结论优先的叙述方式。讨论设计或方案时，先说明核心思路、主要差异和推荐结论。

# Communication style

- 默认使用简单、结论优先的叙述方式。讨论设计或方案时，先说明核心思路、主要差异和推荐结论。
- 不要主动展开文件级改动清单、接口细节、完整边界情况或冗长实现步骤；只有用户明确要求深入时再补充。
- 能用几段短文说明的内容，不要写成长篇分层方案。避免术语堆叠，并保持用户无需反复阅读就能理解。

# Architecture preferences

- 设计阶段就避免以不断增长的 `if/else`、`switch` 或类型判断承载业务差异。优先使用清晰的职责拆分、策略分派、注册表和组合；必要的输入校验与简单控制流不受此限制。
- 不允许出现上帝组件、上帝对象或持续膨胀的入口文件。状态和行为必须放在实际拥有该职责的模块中；入口层只负责组合、编排和暴露能力。
- 新能力落地前先确定所有者和扩展点。若实现主要是在大型组件或主流程中追加状态、分支和方法，应先拆出独立职责，再接入外层。

# UI font-size rules

- 字号按信息角色统一，不随组件随意放大：页面标题约 `22px`，连续正文与主要输入约 `14px`，常规按钮、菜单、导航和列表约 `13px`，辅助说明与紧凑控件约 `11px`，代码约 `12px`。
- 优先复用项目已有的字号 token 或共用类，不散落新的任意字号；确有特殊层级时才局部覆盖，并保持同类组件一致。
- 密集工具栏、Composer 操作行、状态切换和菜单项不能直接沿用组件库偏大的默认字号，应主动校准到 `11–13px`。
- `10px` 只用于极弱、非关键元数据，不用于正文、操作标签、表单内容或必须快速识别的状态；不要用缩小字体代替合理的布局和截断。
- 缩小字号时同步检查行高、图标尺寸、对齐、焦点环和点击区域；文字可以紧凑，但交互命中区不得随之缩小。

# Release process

发布目标是同时生成两个产物：用于 npm 发布的 `pi-shadow-mind-<version>.tgz`，以及解压后可由 `pi install ./pi-shadow-mind-<version>` 安装的 standalone ZIP。

1. 发布前更新 `package.json` 版本，并运行 `npm install` 同步 lockfile。不要覆盖同版本的既有正式产物；代码有变化时应提升版本。
2. 运行 `npm run verify`，要求类型检查及全部测试通过。
3. 运行 `npm run build`，确认扩展入口为 `dist/index.js`，包内不分发 `src/` 和测试。构建必须把 `yaml` 等第三方运行时依赖打入入口 bundle；Pi 核心包与 `typebox` 保持 external。
4. 运行 `npm pack --ignore-scripts --pack-destination release` 生成 `.tgz`。若未提前执行验证和构建，则改用普通 `npm pack`，由 `prepack` 自动完成。
5. 制作 standalone ZIP：将 `.tgz` 解包至同名目录，再压缩整个同名目录。ZIP 内必须保留顶层包目录，且不得依赖接收方另行安装 `yaml` 等普通运行时依赖。
6. 在隔离目录中通过 Pi `DefaultResourceLoader` 加载 standalone 包的 `dist/index.js`。冒烟检查必须设置 `noExtensions: true` 并仅通过 `additionalExtensionPaths` 加载待测入口，避免本机已配置的同名插件造成假冲突。
7. 确认加载无错误，且可识别 `/shadow` 和全部管理工具。随后为 `.tgz`、ZIP 生成 SHA-256，并写入 `release/SHA256SUMS.txt`。
8. 交付时优先提供 standalone ZIP；`.tgz` 用于 npm 发布。说明 ZIP 的安装方式：解压后运行 `pi install ./pi-shadow-mind-<version>`。

清理或覆盖 `release/` 下的暂存目录和旧产物前，必须解析并核对目标的绝对路径确实位于当前项目的 `release/` 内，不得对工作区根目录使用递归删除。

---
> Source: [liuzhengdongfortest/pi-shadow-mind](https://github.com/liuzhengdongfortest/pi-shadow-mind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-15 -->
