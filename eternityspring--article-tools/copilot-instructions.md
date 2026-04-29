## article-tools

> 纯静态前端工具集，无构建系统，无依赖安装，直接浏览器打开即用。

# article-tools

纯静态前端工具集，无构建系统，无依赖安装，直接浏览器打开即用。

## 文件结构

```
index.html          工具箱首页，入口导航
cover.html          封面生成器
qrcode.html         二维码生成器
fonts/              本地字体文件（Noto Serif SC、JetBrains Mono、MiSans）
images/source/      封面素材图片（source1.png、source2.png …）
```

## cover.html 架构

三栏布局：左面板（内容 + 素材）| 中间预览 | 右面板（样式）。

**关键数据结构**
- `COLOR_SCHEMES[]` — 配色方案（0–5 纯色，6–12 渐变）
- `PRESETS[]` — 快捷预设，字段：`scheme / deco / font / content / vis / typography / image`
- `coverImgState` — 当前素材图状态：`{ path, visible, size, right }`
- `visState` — 各文字元素显隐状态

**预设字段说明**
```js
{
  name: '预设名',
  scheme: 6,                    // COLOR_SCHEMES 索引
  deco: 'classic',              // classic | cyberpunk | sphere | minimal
  font: 'default',              // FONT_OPTIONS key
  content: { title, label, subtitle, author },   // 覆盖默认文案
  vis: { label: false },        // 覆盖默认显隐
  typography: { contentWidth: 55, titleSize: 5 }, // 覆盖滑块默认值
  image: { path: 'images/source/source2.png', size: 38, right: -3 }
}
```

**素材图片**：放到 `images/source/` 后，同步在 JS 顶部的 `SOURCE_IMAGES` 数组里加一行路径。

**Copy 配置**：`buildConfigPayload()` 输出当前与默认值的差异，含 image 字段（若有选图）。

## 行为规则

- 不添加 "Co-Authored-By: Claude" 到 git commit。

---
> Source: [eternityspring/article-tools](https://github.com/eternityspring/article-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
