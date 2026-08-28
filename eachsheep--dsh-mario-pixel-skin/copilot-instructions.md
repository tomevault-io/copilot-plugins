## dsh-mario-pixel-skin

> 本仓库是 DSH Skin Center v2 的纯资源皮肤，不是 Cordis 或 npm 插件。它只拥有视觉资源，不拥有 Harness 的会话、工作区、模型、权限、工具调用、设置或持久化行为。维护成功意味着皮肤能被 v2 catalog 正确发现，在试穿和应用时完整加载，并在退出后由 Skin Center 清理；仅看到局部颜色变化不算完成。

# DSH Mario Pixel Skin 维护约定

本仓库是 DSH Skin Center v2 的纯资源皮肤，不是 Cordis 或 npm 插件。它只拥有视觉资源，不拥有 Harness 的会话、工作区、模型、权限、工具调用、设置或持久化行为。维护成功意味着皮肤能被 v2 catalog 正确发现，在试穿和应用时完整加载，并在退出后由 Skin Center 清理；仅看到局部颜色变化不算完成。

## 资源合同

`skin.json` 是唯一 manifest，必须保持 `skinManifestVersion: 2`，其中声明的 stylesheet、patches、背景和预览路径都必须位于本目录并真实存在。`skin.css` 只定义稳定主题令牌和基础视觉；`patches.css` 保存对当前 DSH 界面的表面、控件与 decoration layer 适配。加载器会自动把 CSS 限定在 `html[data-dsh-skin="pixel-kingdom"]` 下，因此文件内不要重复写根作用域，也不要通过 `!important` 掩盖层叠问题。

用户皮肤不能运行 hook，这一限制是 v2 的安全边界。标题栏或背景装饰只能使用 Skin Center 提供的 layer 与 CSS 伪元素，不得重新引入 JavaScript、Cordis 生命周期、网络访问、存储读写或业务 DOM 替换。如果仅靠静态资源无法忠实表达新需求，应明确说明能力缺口，而不是绕过加载器。

## 修改与验收

修改前先确认问题属于 manifest、稳定令牌、当前界面适配还是图片资源，只改解决当前问题所需的文件。一次性组合或批量检查脚本放在所配合脚本的临时目录，不进入长期代码。不要为不可能发生的内部状态增加回退或静默默认值；只在 manifest、文件路径和 DSH HTTP/UI 这类系统边界快速失败，任何真实错误都应暴露并修复根因。

静态验收要解析 manifest、核对全部相对资源并确认 CSS 不含旧插件作用域或 `!important`。运行时验收使用当前官方 DSH Web 服务，在“设置 → 皮肤中心”确认 catalog 没有 diagnostics，然后分别检查试穿、亮暗主题、设置页、会话区、文件树和窄侧栏。必须确认两个 CSS 请求和背景请求成功、控制台没有新增错误，并在退出试穿后确认 `data-dsh-skin` 与资源 link 被移除。没有完成运行时检查时，只能报告静态验证通过。

撰写要求、规范或流程时应保持因果和交接关系完整，讲清楚问题、适用条件、执行者、产出及验证，不要用大量孤立条目替代必要叙述。所有改动遵循最小实现原则：不假设、不隐藏困惑、不做推测性功能，只清理本次迁移产生或已被本次迁移明确替代的内容，并在达到上述成功标准前持续验证。

---
> Source: [EachSheep/dsh-mario-pixel-skin](https://github.com/EachSheep/dsh-mario-pixel-skin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
