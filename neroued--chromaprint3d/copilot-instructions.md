## release-changelog

> 发版时 manifest changelog 的撰写规范


# 版本更新日志撰写规范

执行 `scripts/release.sh` 时需要传入 `changelog` 参数，该内容会写入 `version-manifest.json` 并展示给最终用户（包括 Web 页面弹窗和 Electron 更新提示）。

## 必须遵守

- **面向用户**：用普通用户能理解的语言描述变化，而不是照搬 commit message。
- **突出价值**：说明"用户能得到什么"，而不是"开发者改了什么"。
- **避免专业术语**：不使用模块名、函数名、架构概念（如"子模块"、"重构"、"迁移"等）。
- **简洁有序**：每条一行，以 `- ` 开头，最重要的变化放在最前面。
- **中文撰写**：changelog 使用中文。

## 改写示例

| commit message（内部） | changelog（面向用户） |
|---|---|
| feat: version update notification for web and Electron clients | 新增版本更新提醒，有新版本时自动通知 |
| refactor: migrate vectorization to neroued_vectorizer submodule | 矢量化处理性能优化 |
| fix: restore calibration board meta JSON download | 修复校准板配置文件无法下载的问题 |
| feat: 自动聚类检测 + 三态聚类模式 | 智能颜色分析，自动识别图片最佳配色方案 |
| fix(backend): use WHOLE_ARCHIVE to preserve controller registration | 修复部分接口偶发不可用的问题 |

## 格式

changelog 在 JSON 中以 `\n` 分隔多行，示例：

```
"- 新增版本更新提醒，有新版本时自动通知\n- 矢量化处理性能优化"
```

---
> Source: [Neroued/ChromaPrint3D](https://github.com/Neroued/ChromaPrint3D) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
