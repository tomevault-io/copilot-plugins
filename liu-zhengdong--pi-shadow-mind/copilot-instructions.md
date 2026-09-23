## pi-shadow-mind

> - 默认使用简单、结论优先的叙述方式。讨论设计或方案时，先说明核心思路、主要差异和推荐结论。

# Project instructions

## Communication style

- 默认使用简单、结论优先的叙述方式。讨论设计或方案时，先说明核心思路、主要差异和推荐结论。
- 不要主动展开文件级改动清单、接口细节、完整边界情况或冗长实现步骤；只有用户明确要求深入时再补充。
- 能用几段短文说明的内容，不要写成长篇分层方案。避免术语堆叠，并保持用户无需反复阅读就能理解。

## Architecture preferences

- 设计阶段就避免以不断增长的 `if/else`、`switch` 或类型判断承载业务差异。优先使用清晰的职责拆分、策略分派、注册表和组合；必要的输入校验与简单控制流不受此限制。
- 不允许出现上帝组件、上帝对象或持续膨胀的入口文件。状态和行为必须放在实际拥有该职责的模块中；入口层只负责组合、编排和暴露能力。
- 新能力落地前先确定所有者和扩展点。若实现主要是在大型组件或主流程中追加状态、分支和方法，应先拆出独立职责，再接入外层。

## UI font-size rules

- 字号按信息角色统一，不随组件随意放大：页面标题约 `22px`，连续正文与主要输入约 `14px`，常规按钮、菜单、导航和列表约 `13px`，辅助说明与紧凑控件约 `11px`，代码约 `12px`。
- 优先复用项目已有的字号 token 或共用类，不散落新的任意字号；确有特殊层级时才局部覆盖，并保持同类组件一致。
- 密集工具栏、Composer 操作行、状态切换和菜单项不能直接沿用组件库偏大的默认字号，应主动校准到 `11–13px`。
- `10px` 只用于极弱、非关键元数据，不用于正文、操作标签、表单内容或必须快速识别的状态；不要用缩小字体代替合理的布局和截断。
- 缩小字号时同步检查行高、图标尺寸、对齐、焦点环和点击区域；文字可以紧凑，但交互命中区不得随之缩小。

## Regression verification

- 涉及上下文投影、配置热更新或报告展示的跨模块改动，按风险补充受影响链路的定向回归：使用真实数据形态，经过实际消费入口，断言最终 Shadow 轨迹或用户可见输出。辅助函数测试用于验证局部逻辑，链路测试用于验证状态更新与模块组合；mock 保留被验证的关键路径。普通任务仍按需验证，不增加统一全量门禁。

## Pull request handling

- 用户要求合并 PR 时，默认执行审计、修复问题、按风险验证并合并；发现较大风险时暂停合并，先汇报风险与需要用户决定的事项。发布新版本按用户要求单独执行。

## Release process

正式发布由 `.github/workflows/release.yml` 在 `v*` tag 上完成；本地只负责准备并推送发布提交：

1. 选择未发布的新版本，运行 `npm version <version> --no-git-tag-version`，一次同步 `package.json` 与 lockfile。
2. 提交并推送代码，然后创建并推送与包版本一致的 `v<version>` tag。
3. 确认 Release workflow 成功。workflow 负责验证、构建、生成并冒烟检查 npm tarball 与 standalone ZIP、通过 OIDC 发布 npm，以及创建带校验和的 GitHub Release。

交付时优先使用 standalone ZIP；解压后运行 `pi install ./pi-shadow-mind-<version>`。本地无需重复制作正式发布产物。

---
> Source: [liu-zhengdong/pi-shadow-mind](https://github.com/liu-zhengdong/pi-shadow-mind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
