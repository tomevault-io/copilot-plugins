## onepic-desktop-pet

> 1. 开始工作前先阅读 `文件夹说明.md`、`技术更新.md` 和 `agent-guide/AGENT_GUIDE.md`。

# OnePic Desktop Pet Agent 规则

1. 开始工作前先阅读 `文件夹说明.md`、`技术更新.md` 和 `agent-guide/AGENT_GUIDE.md`。
2. 环境检查是制作流程的第一步；不得在没有用户确认的情况下安装系统软件、修改环境变量或申请管理员权限。
3. Python 依赖只能安装到项目 `.venv`，优先调用 `scripts/` 中已有脚本。
4. 未明确授权公开的图片只能放入 `user_assets/`；虚拟角色或其他已获授权素材可以进入公开演示目录，但必须同步写明授权范围。
5. 修改 Python 文件时同步维护文件顶部的中文模块文档字符串。
6. 原图登记后先让用户选择保留原画风、轻度 Q 版或完整 Q 版，再只生成一张标准角色候选；必须通过确认窗口获得用户确认，未确认时不得批量生成动作。
7. 标准角色确认后才能生成动作；走路必须包含左右落脚、下压、迈步和抬升八相位，并生成 GIF 交给用户确认。
8. `workflow.json` 中角色和走路未同时标记为 `approved` 时，不得绕过门禁加载或打包私有宠物。
9. 自拍成片必须使用最初登记的原图，不得用动画帧、标准角色图或其他演示图片代替。
10. 新增角色动作后必须运行素材、行为、配置、流程和窗口测试；自动测试不能替代视觉验收。
11. 不删除历史素材，不覆盖用户原图；需要替换动作时保留可恢复副本。
12. 打包公开版本前检查 `user_assets/`、`dist/` 和打包资源，确认不存在私人图片和本机绝对路径。

---
> Source: [Taylor154/OnePic-Desktop-Pet](https://github.com/Taylor154/OnePic-Desktop-Pet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
