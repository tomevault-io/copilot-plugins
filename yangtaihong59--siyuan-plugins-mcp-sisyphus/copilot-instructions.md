## siyuan-plugins-mcp-sisyphus

> 这个页面记录贡献者需要遵守的代码和结构约定。

# 规范约定

这个页面记录贡献者需要遵守的代码和结构约定。

适用场景：你正在实现改动，希望和当前仓库形态保持一致。

## 约定

- 插件入口和 CLI 入口复用同一工具时，行为要保持一致
- 沿用现有的聚合工具模型，不要把每个 action 拆成单独工具
- 文档事实应和 `src/core/config.ts`、`src/core/tool-registry.ts`、当前 build target 保持一致
- 远程安全相关操作优先走 SiYuan API，而不是直接操作本地文件系统

---
> Source: [yangtaihong59/siyuan-plugins-mcp-sisyphus](https://github.com/yangtaihong59/siyuan-plugins-mcp-sisyphus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
