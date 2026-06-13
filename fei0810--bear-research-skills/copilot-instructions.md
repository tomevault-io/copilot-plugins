## bear-research-skills

> 作者在学术科研工作中积累的分析思路和方法，以 Agent Skill 的形式沉淀分享。覆盖选题评估、文献溯源、论点攻防、领域入门等场景，依托文献检索命令行工具 SciMaster CLI，所有分析都基于真实文献进行。

# bear-research · 插件说明

作者在学术科研工作中积累的分析思路和方法，以 Agent Skill 的形式沉淀分享。覆盖选题评估、文献溯源、论点攻防、领域入门等场景，依托文献检索命令行工具 SciMaster CLI，所有分析都基于真实文献进行。

## 必须先安装：scimaster-cli

bear-research 的所有技能依赖 `scimaster-cli`。**安装一次，永久可用。**

```bash
# 需要 Node.js >= 20
npm install -g scimaster-cli

# 完成认证（粘贴 API Key）
sci init

# 验证安装
sci --version
```

获取 API Key：注册 [SciMaster](https://scimaster.bohrium.com/vibe-write/home) 账号后，点击左下角用户头像打开设置界面 → API Key 页面 → 创建新 Key，复制后粘贴到 `sci init` 提示中。CLI 用户每月有免费检索额度，`sci usage` 查看余额。

## 执行原则

技能执行时尽量少打扰用户：
- 除非输入本身含义模糊，否则直接开始检索，不要求用户逐步确认
- 告知进展用一句话（"正在检索 q1…"），不要冗长汇报
- 只在真正需要用户决策时才暂停（输入歧义、CLI 未安装）

## 首次使用检测

每个技能在 Step 0 都会自动运行 `sci --version`：
- 输出版本号 → 继续执行
- 命令未找到 → 给出安装命令并停止，等用户装好后重试
- 认证错误（401）→ 提示运行 `sci init`

用户不需要手动检查，技能自己会在运行前确认依赖可用。

## 输出文件

每次运行在当前目录下创建 `{topic-slug}/` 子文件夹，自动产出四个文件：

| 文件 | 内容 |
|---|---|
| `report.md` | 结构化 Markdown 报告，含 YAML front matter |
| `report.html` | 自包含 HTML 报告（bear-map 为 `map.html`） |
| `references.bib` | 合并去重后的 BibTeX 引用文件 |
| 终端打印 | TUI 格式即时摘要 |

## 基础技能

| 技能 | 用途 |
|---|---|
| `bear-support` | 为观点找支持文献，按支撑强度排序 |
| `bear-counter` | 为观点找反对文献，按威胁等级分类 |
| `bear-map` | 以概念为中心，画出邻近概念知识地图 |
| `bear-scoop` | 选题撞车检测，多角度扫描邻近程度 |
| `bear-trace` | 文献溯源，追溯领域演化脉络 |

## 工作流（多技能串联）

| 工作流 | 技能链 | 用途 |
|---|---|---|
| `bear-review` | support → counter | 同时看正反两侧证据格局，适合准备论点或预判审稿人 |
| `bear-onboard` | map → trace | 同时建立概念版图和演化脉络，适合快速入门陌生领域 |
| `bear-propose` | scoop → support → counter | 立项前评估，支撑和挑战检索聚焦在撞车检测识别出的安静区 |

---
> Source: [fei0810/bear-research-skills](https://github.com/fei0810/bear-research-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-13 -->
