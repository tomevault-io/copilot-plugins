## fppt-std

> >


## 模式路由

### 快速模式
- 触发："快速生成"、"把这个资料做成PPT"、给完整资料的情况
- 流程：理解+规划一步完成 → 逐页生成 → 拼装输出
- 目标：≤15页，≤10分钟
- Prompt: `core/prompts/quick_mode.md`

### 标准模式
- 触发："标准模式"、"用心做"、"正式做"、场景重要但信息不足
- 流程：FPPT五层创作引擎（洞察→调研→架构→呈现→打磨），3个Gate确认
- 目标：≤30分钟（含用户交互）
- Prompts: `core/prompts/L1_insight.md` ~ `L5_polish.md`

### 模糊指令
- AI判断：素材充分常规场景→快速；正式场合/信息不足→标准或反问

## 版本说明

FPPT 标准版为开源免费版本，所有核心功能直接可用。标准版使用用户自带的 DeepSeek API Key，无需注册、无运营依赖。

### 标准版功能
- 7 套内置主题（财经蓝 / 商务灰 / 科技黑 / 杂志 / 稳健绿 / 政务红 / 经典白）
- 15 种标准布局骨架
- 快速模式 + 标准模式 5 层创作引擎
- HTML 输出 + PPTX 导出 + PDF 导出
- 演讲者模式（P 键双窗口 + 计时器 + 备注 + 缩略图导航）
- 键盘翻页、T 键主题切换、ESC 概览、URL hash 恢复
- Chart.js 图表（本地回退）
- 中英文双语界面

### API Key 配置
标准版使用用户自带的 DeepSeek API Key。在 `.env` 文件中配置：
```
DEEPSEEK_API_KEY=sk-your-key-here
```
获取 Key：https://platform.deepseek.com

### 升级到 Pro

以下功能需要 FPPT Pro 许可（详见 [FPPT 官网](https://hzddyy.com/fppt)）：
- 金融行业完整包（5 主题 + 7 布局 + 10+ 金融图表 + 合规模板）
- 金融数据保真体系（数据锁定模式 + 术语规范库 + 合规声明强制检查）
- One-Pager 完整体系（BP / 运营 / 标的 / 策略 / 项目 / 风控，6 种）
- 21 套 Pro 专属主题 + 25 套 Pro 专属布局
- AI 图片生成
- 多画布格式（4:3 / 3:4 / 1:1）
- 长文本预处理 + URL 抓取

### Pro 许可验证

在使用任何 Pro 功能前，SHALL 执行许可检查：

```bash
python scripts/license_check.py
```

- 退出码 0 → Pro 功能可用
- 退出码非 0 → 不可用，按下方引导话术回复

**许可文件位置**：项目根目录下的 `.fppt_license` 文件。
**激活方式**：
1. 在 [FPPT 官网](https://hzddyy.com/fppt) 购买 Pro
2. 从 Dashboard 下载 `.fppt_license` 文件，放入 fppt-std 目录
3. 或运行 `python scripts/license_check.py --activate <许可密钥>` 在线激活

### Pro 资源检测

除许可文件外，还需检查 Pro 资源包是否已解压：

- `assets/themes/pro/` 目录存在且非空
- `assets/layouts/pro/` 目录存在且非空
- `packs/finance/` 目录存在且非空

若许可有效但资源包缺失，SHALL 提示用户从官网下载 Pro 资源包。

### 升级引导话术
当用户请求 Pro 功能但许可无效时，SHALL 回复：
> 此功能为 FPPT Pro 版专属。升级请访问 https://hzddyy.com/fppt，购买后下载许可文件和 Pro 资源包即可使用。

## 生成指南

### 快速模式执行步骤

1. **读取 Prompt 模板**：`core/prompts/quick_mode.md`
2. **读取 seed 模板**：`assets/template.html`（了解可用 CSS 类名和 JS 能力）
3. **理解需求**：从用户输入提取场景/受众/页数/数据 → **立即告知用户**：场景判断 + 预估页数 + 主题 + 预计耗时
4. **规划大纲**：为每页选定布局（从 `assets/layouts/std/`）+ 主题（默认 financial-blue）→ **输出大纲一览**
5. **逐页生成**：基于布局骨架填充内容，遵循设计令牌和美学纪律 → **每 2-3 页输出一次进度**
6. **拼装输出**：将 slides 插入 template.html → 输出 `index.html` → **报告完成 + 文件位置**

### 关键约束
- **模板保护**：生成的 HTML SHALL 完整保留 `assets/template.html` 中的所有 `<style>` 和 `<script>` 块不变，**仅替换 `<!-- SLIDES_HERE -->`** 位置的内容，不得重写导航、WebGL、键盘事件、主题切换等 runtime 代码
- 所有颜色使用 `var(--token)` 引用，严禁硬编码 hex
- 只能使用 template.html 中已定义的 CSS 类名
- 严禁 emoji（用 Lucide 图标替代）
- 图片使用标准比例类或固定高度类
- 金融数据使用 `.c-positive` / `.c-negative` / `.c-neutral`

### 文件结构
```
assets/
├── template.html          ← 种子模板（CSS + WebGL + JS）
├── motion.min.js          ← Motion One 动效库
├── chart.umd.min.js       ← Chart.js 离线回退
├── presenter.js           ← 演讲者模式
├── themes/std/*.css       ← 7套标准版主题
├── themes/pro/*.css       ← Pro版主题（需许可+资源包）
├── layouts/std/*.html     ← 15种布局骨架
├── layouts/pro/*.html     ← Pro版布局（需许可+资源包）
core/
├── prompts/               ← AI Prompt 模板
├── checklist.md           ← P0-P3 质检清单
packs/finance/             ← 金融行业包（Pro专属）
scripts/
├── license_check.py       ← Pro 许可验证脚本
references/                ← AI 参考知识库
```

## 输出
- **文件命名**：基于内容主题生成描述性文件名，格式 `output/<项目名>/<主题关键词>.html`（如 `output/量化策略分类介绍/CTA趋势跟踪策略.html`），**严禁**使用固定名称 `index.html`
- PPTX/PDF 使用与 HTML 相同的前缀（如 `CTA趋势跟踪策略.pptx`）
- 生成后必须将 `assets/presenter.js`、`assets/motion.min.js`、`assets/chart.umd.min.js` 复制到 `output/<项目>/assets/` 目录
- 所有生成需经 `core/checklist.md` P0 检查通过后方可交付

---
> Source: [leonai42/fppt-std](https://github.com/leonai42/fppt-std) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
