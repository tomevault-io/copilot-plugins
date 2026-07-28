## obsidian-i18n

> 本文件约束任何 AI 代理或自动化脚本在 `docs/` 下修改文档时的行为。

# Obsidian i18n 文档维护说明

本文件约束任何 AI 代理或自动化脚本在 `docs/` 下修改文档时的行为。

## 先验规则

- 先查源码，再写文档
- 写中文主文，再同步英文镜像
- 没有证据的行为不要写进文档
- 不要为了“好懂”而虚构英文 UI 名称

## 优先核对的源码文件

- 设置页结构：
  - `src/settings/index.ts`
  - `src/locales/zh-cn/settings.ts`
- 管理中心结构：
  - `src/views/manager/manager-layout.tsx`
  - `src/locales/zh-cn/manager.ts`
- Cloud 视图结构：
  - `src/views/cloud/cloud-view.tsx`
  - `src/views/cloud/types.ts`
  - `src/locales/zh-cn/cloud.ts`
- 本地导入导出：
  - `src/views/manager/components/translation-manager-panel.tsx`
- GitHub 发布：
  - `src/views/cloud/components/publish-tab.tsx`
  - `src/views/cloud/components/manage-tab.tsx`
- 默认值：
  - `src/settings/data.ts`

## 文档写作边界

- 可以写：
  - 已在源码中出现的标签名、字段名、默认值、文件路径、仓库路径、导入导出扩展名
  - 已通过源码确认的入口关系，例如“Cloud 是独立视图，不是管理中心顶部 tab”
- 不可以写：
  - 未在源码中出现的英文标签名
  - 未验证的后端支持范围
  - 主观推荐语替代事实说明
  - 未来式承诺或猜测

## 双语同步规则

- 每个公开中文页面都必须有 `docs/en/` 对应页面
- 英文页必须保持与中文页同一 slug 和信息架构
- 英文页引用真实 UI 标签时，直接使用当前源码中的中文标签，例如 `语言模型`、`综合设置`、`插件`

## 术语映射

- `Obsidian i18n`：产品名
- `I18N`：Obsidian 插件列表显示名
- `zh-cn` / `ja` / `fr`：插件译文语言码
- `zh-Hans` / `en`：Mintlify 站点语言
- `管理中心`：Manager
- `社区目录` / `探索资源` / `管理中心`：Cloud 视图标签

## 交付前检查

从 `G:\goProjects\obsidian-i18n\docs` 运行：

```bash
npx mint validate
npx mint broken-links
```

如果新增了公开页面，还要检查 `docs/docs.json` 是否同步更新。

---
> Source: [eondrcode/obsidian-i18n](https://github.com/eondrcode/obsidian-i18n) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
