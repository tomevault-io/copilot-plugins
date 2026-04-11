## zhihuishu-pro-ppt-downloader

> **Zhihuishu-Pro-PPT-Downloader** 是一套专门用于从 **智慧树 (Zhihuishu AI Smart Course Pro)** 平台全量提取与审计 PPT 资源的工具集。项目采用“分离式架构”来高效处理身份验证与资源提取：

# GEMINI.md - 智慧树 PPT 提取器教学指令上下文

## 项目概览
**Zhihuishu-Pro-PPT-Downloader** 是一套专门用于从 **智慧树 (Zhihuishu AI Smart Course Pro)** 平台全量提取与审计 PPT 资源的工具集。项目采用“分离式架构”来高效处理身份验证与资源提取：

1.  **提取端 (Tampermonkey)**：油猴脚本运行在已登录的浏览器页面中。它支持旧版 `learnPage` 与新版 `singleCourse` 布局，通过调用平台内部 API (`getListNodeResourcesWithStatus`) 扫描所有课程节点，并导出包含完整资源信息的 JSON 清单 (Manifest)。
2.  **审计端 (Python)**：由 `uv` 管理的 CLI 工具负责处理导出的清单。它提供资源摘要、重复项识别，并能将清单与本地下载目录进行“对账 (Reconciliation)”，确保下载的完整性。

### 核心技术栈
- **Python 3.11+**：负责清单处理、统计与对账的核心逻辑。
- **uv**：极致快速的依赖管理与工具运行环境。
- **Tampermonkey / JavaScript**：浏览器侧的资源嗅探（Vue 3, Vuex, Pinia 劫持）。
- **Argparse**：规范化的 CLI 接口实现。

---

## 构建与运行

### 环境准备
- 系统已安装 [uv](https://github.com/astral-sh/uv)。
- 浏览器已安装 [Tampermonkey](https://www.tampermonkey.net/) 插件。

### 初始化
```powershell
uv sync
```

### 执行工作流

#### 1. 导出资源清单 (浏览器侧)
1. 在油猴插件中安装 `tampermonkey/zhihuishu-pro-ppt-downloader.user.js`。
2. 登录智慧树，进入课程学习页（支持 `learnPage` 和 `singleCourse` 布局）。
3. 使用脚本面板点击“导出整门课资源 JSON”，保存 manifest 文件。

#### 2. 生成清单摘要 (CLI)
分析 JSON 清单以查看资源总数、PPT 分布等：
```powershell
uv run zhihuishu-pro-ppt-downloader manifest-summary "path/to/manifest.json"
```

#### 3. 全自动批量下载 (CLI)
直接根据清单中的 URL 批量下载 PPT 文件到本地：
```powershell
uv run zhihuishu-pro-ppt-downloader manifest-download "path/to/manifest.json" --downloads-dir "./downloads"
```

#### 4. 转换 PDF (CLI)
按照课程顺序将 PPT 转换为 PDF（需 Windows + PowerPoint）。

**合并为一个带书签的文件：**
```powershell
uv run zhihuishu-pro-ppt-downloader manifest-export-pdf "path/to/manifest.json" --downloads-dir "./downloads" --output "course_merged.pdf"
```

**分散转换为独立文件：**
```powershell
uv run zhihuishu-pro-ppt-downloader manifest-export-pdf "path/to/manifest.json" --downloads-dir "./downloads" --individual --output-dir "./pdfs"
```

#### 5. 自动化对账 (CLI)
验证清单中的所有 PPT 是否已完整下载到本地：
```powershell
uv run zhihuishu-pro-ppt-downloader manifest-reconcile "path/to/manifest.json" --downloads-dir "./downloads"
```

---

## 核心开发约定

### 架构洞察 (Architecture Insights)
- **API 优先**：提取脚本不依赖 DOM 爬取，而是通过 Vue 实例直接调用站点内部 API，保证了极高的稳定性。
- **全量扫描**：脚本会扫描所有知识点，即使某些节点声明资源数为 0（实测发现某些“零计数”节点内部依然藏有 PPT）。
- **精准识别**：基于 `resourcesType: 1` 和 `resourcesDataType: 11` 进行 PPT 判定。

---

## 重点文件说明
- `src/zhihuishu_pro_ppt_downloader/cli.py`：Python CLI 工具入口。
- `src/zhihuishu_pro_ppt_downloader/manifest.py`：清单解析与对账核心逻辑。
- `tampermonkey/zhihuishu-pro-ppt-downloader.user.js`：浏览器侧提取脚本。
- `doc/zhihuishu-ppt-reverse-analysis-report.md`：逆向分析报告与架构依据。
- `pyproject.toml`：项目元数据与脚本定义。

---

## 技术注意事项
- **签名校验**：站点 API 带有复杂的请求签名，因此不建议开发纯外部 Python 爬虫，保持油猴脚本的“寄生”调用是成本最低的方案。
- **路径清洗**：本地文件名通过 `manifest.py:sanitize_filename` 进行清洗，确保跨平台文件系统的兼容性。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hicancan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hicancan)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
