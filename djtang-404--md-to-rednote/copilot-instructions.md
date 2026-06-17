## md-to-rednote

> Convert Markdown articles into RedNote (小红书/Xiaohongshu) style image cards. Use when: generating social media images from markdown, creating rednote posts, converting blog posts to card-style images for rednote, xiaohongshu, 小红书, 图片卡片.


# Markdown 转 RedNote（小红书）图片卡片

## 功能

将 Markdown 格式的文章自动拆分并渲染为多张 RedNote（小红书）风格的图片卡片（1080×1440），可直接用于发布。

支持特性：
- 基于像素高度的智能分页，内容密度均匀
- 标题保护机制（标题不会孤零零出现在页底）
- 手动分页标记 `<!-- pagebreak -->`
- 8 套配色主题
- 超过图片数量上限时自动缩小字号适配
- 任务列表（`- [x]`/`- [ ]`）渲染为复选框样式
- Emoji 完整支持
- 数学公式（LaTeX）支持

## 使用场景

- 将博客文章、学习笔记转为 RedNote 图文笔记
- 将技术文档转为易读的卡片式图片
- 批量生成统一风格的社交媒体内容图

## 使用方法

```bash
python .github/skills/md-to-rednote/scripts/convert.py <markdown_file> [--output <dir>] [--theme <name>] [--max-images <n>]
```

参数说明：
- `markdown_file`：输入的 Markdown 文件路径
- `--output, -o`：输出目录（默认：markdown 文件同目录下的 `<文件名>_cards/`）
- `--theme, -t`：配色主题（默认：warm）
- `--max-images, -m`：最大图片数量，超过会自动缩小字号（默认：18）

### 可选主题

- `warm` — 暖橙色调（默认）
- `cool` — 冷蓝色调
- `green` — 清新绿色调
- `dark` — 暗色主题
- `xiaohongshu` — RedNote 经典粉红
- `zhihu` — 知乎蓝
- `notion` — Notion 简约风
- `douyin` — 抖音暗色

## 依赖

- Python 3.10+
- markdown-it-py
- jinja2
- playwright（需安装 Chromium：`playwright install chromium`）

## 参考资源

- [转换脚本](./scripts/convert.py)
- [封面模板](./assets/templates/cover.html)
- [正文模板](./assets/templates/content.html)

---
> Source: [DjTaNg-404/md-to-rednote](https://github.com/DjTaNg-404/md-to-rednote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
