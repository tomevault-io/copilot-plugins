## sakurairo-theme

> A comprehensive skill for using the Sakurairo WordPress theme features. Use this skill when creating articles, building pages, using shortcodes, setting up galleries/exhibitions, configuring homepage components, working with ChatGPT/AI features, managing Bilibili/Steam/QQ integrations, customizing theme options, or writing content with theme-specific features. Triggers on tasks involving Sakurairo content creation, page building, shortcode usage, theme functionality, and visual customization.


# Sakurairo Theme - 功能使用技能

## 概述

本技能提供使用 Sakurairo WordPress 主题各项功能的完整指导。Sakurairo 是一款基于 Sakura V3 Series 重构开发的 WordPress 主题，具有 AI 辅助阅读、多服务 API 集成（B站、Steam、QQ、Bangumi）、丰富的页面构建能力、灵活的首页组件系统、多样的文章展示样式等特性。本技能聚焦于**内容创作者和网站管理者**的日常使用场景。

---

## 一、文章写作与内容展示

### 1.1 文章 AI 摘要（AI Excerpt）

主题集成了 ChatGPT AI 辅助阅读功能，可为文章自动生成摘要。

**启用方式：**
- 在后台主题选项 → ChatGPT/AIGC 设置中配置 API Key
- 发表或更新文章时，通过 REST API 触发摘要生成：
  - 管理员调用 `GET /wp-json/sakura/v1/chatgpt?post_id={文章ID}`
- 生成的摘要存储在文章自定义字段 `ai_summon_excerpt` 中
- 在前台单页文章顶部以蓝色提示框显示，带有 `AI Excerpt` 标签

**使用场景：** 为长篇文章自动生成精炼摘要，提升读者阅读体验。

### 1.2 AI 名词注释（AI Annotations）

自动识别文章中复杂或专业术语，并生成浮动注释。

**启用方式：**
- 管理员调用 `GET /wp-json/sakura/v1/chatgpt/annotate?post_id={文章ID}`
- 注释数据存储在 `iro_chatgpt_annotations` 自定义字段中
- 文章中匹配的关键词会自动显示为带下划线的可点击注释

**使用场景：** 技术文章、科普文章中为专业术语添加即时解释。

### 1.3 文章目录（Table of Contents）

自动根据文章中的标题标签（h1~h6）生成目录。

**启用方式：** 主题选项 → 文章设置 → 开启「自动生成文章目录」

**行为说明：**
- 文章含有一个或多个标题标签时自动显示目录
- 目录在桌面端侧边栏浮动显示
- 移动端通过 `layouts/mo_toc_menu.php` 显示为折叠菜单

### 1.4 文章字数统计与阅读时间

自动统计文章字数并估算阅读时长。

**功能：**
- 支持 CJK（中日韩）字符和英文单词混合统计（详见 `inc/word-stat.php`）
- 阅读时间按每分钟 220 字估算
- 显示在文章 meta 区域，格式如："约 5 分钟"、"不到 1 分钟"

### 1.5 文章许可协议

每篇文章可单独设置或继承全局知识共享许可协议。

**支持协议：**
- `cc-by-nc-sa`（默认，署名-非商业性使用-相同方式共享 4.0）
- `cc0`（公共领域贡献）
- 可在文章编辑页面单独设置 `license` 自定义字段覆盖全局设置

### 1.6 文章内容样式

主题提供两种文章内容渲染风格，可在主题选项中切换：
- **Sakura 风格**（`css/content-style/sakura.css`）- 柔和圆润
- **GitHub 风格**（`css/content-style/github.css`）- 简洁清晰

### 1.7 代码高亮

文章中的代码块自动应用语法高亮（通过 Highlight.js）。

Gutenberg 编辑器中的代码块可设置 `language` 属性，自动添加 `language-xxx` CSS 类。

---

## 二、短代码（Shortcodes）

### 2.1 使用说明

在文章或页面编辑器中直接插入以下短代码即可调用相应功能。

### 2.2 音乐播放器（APlayer / Meting）

```
[ap layer]
```
通过 Meting API 集成网易云音乐/QQ音乐，在文章中嵌入音乐播放器。

**后台配置：** 主题选项 → 社交/集成 → Meting API 设置

### 2.3 Bilibili 视频

```
[bilibili bvid="BV1xx411c7mD"]
```

嵌入 Bilibili 视频到文章中。支持通过 BVID 或 AVID 引用视频。

### 2.4 画廊/相册

```
[gallery]
```
在文章中嵌入图片画廊。支持从媒体库选择图片，也可以通过 REST API 获取图库数据。

### 2.5 其他短代码

更多短代码样式定义在 `css/shortcodes.css` 中，包括各类图文混排布局。

### 2.6 Gutenberg 编辑器块

主题提供了自定义 Gutenberg 块（`inc/blocks/`），在古腾堡编辑器中可用：

- 添加 Font Awesome 图标
- 自定义布局块
- 代码块语言标记支持（自动为代码块添加 `language-xxx` CSS 类）

---

## 三、页面创建与特殊页面类型

### 3.1 首页组件系统

首页不是固定模板，而是由**组件排序系统**驱动的。在主题选项中可调整以下组件的顺序：

| 组件 | 功能 |
|---|---|
| `static_page` | 在首页嵌入一个静态页面内容 |
| `exhibition` | 展示区/精选区（热门文章、特色内容） |
| `primary` | 主文章列表（博客流） |

**配置路径：** 主题选项 → 首页设置 → 首页组件排序

### 3.2 归档页面

主题提供归档信息 REST API 端点（`/wp-json/sakura/v1/archive_info`），用于展示文章归档统计。

通过创建页面并选择归档模板来使用。

### 3.3 友情链接页面

主题启用了 WordPress 内置的链接管理器（通过 `pre_option_link_manager_enabled` 过滤器）。

**友情链接状态检测：**
- 后台新增「链接状态」管理页面（`inc/link-status.php`）
- 可手动检测单个或所有友情链接的可用性
- 记录 HTTP 状态码、失败次数、错误信息

### 3.4 展示区（Exhibition）

首页展示区组件可显示多种站点统计信息方块，在主题选项中配置：

| 功能方块 | 说明 |
|---|---|
| 文章数 | 显示站点文章总数 |
| 评论数 | 显示评论总数 |
| 访客数 | 显示访问量 |
| 链接数 | 显示友情链接数量 |
| 运行时长 | 显示站点已运行时间 |

**配置路径：** 主题选项 → 首页设置 → 展示区组件

### 3.5 分类/标签图片

主题支持为分类和标签添加自定义图片（`inc/categories-images.php`）。

**设置方式：**
- 编辑分类/标签时出现「分类/标签图像」字段
- 支持上传自定义图片
- 前台分类/标签页面显示对应图片

---

## 四、首页与封面设置

### 4.1 封面区域

首页顶部封面支持多种模式：

| 模式 | 设置选项 |
|---|---|
| 视频封面 | 上传/链接视频文件，支持直播流、循环播放 |
| 图片封面 | 设置静态封面图片 |
| 全屏模式 | 封面占满整个视口高度 |
| 非全屏模式 | 封面固定高度 |

**配置路径：** 主题选项 → 首页封面

### 4.2 个人头像

支持使用本地图片作为头像，防止外源 CDN 抽风问题。
- 设置项：`personal_avatar`（在主题选项中上传）
- 兜底：使用 WordPress 默认 Gravatar

### 4.3 文本 Logo

支持自定义文本 Logo：
```php
// 设置示例
text_logo = [
    'color' => '#ff6496',
    'size'  => '24',
    'font'  => 'Noto Serif SC'
]
```

---

## 五、社交媒体与外部服务集成

### 5.1 Bilibili 集成

| 功能 | 说明 |
|---|---|
| 视频嵌入 | 通过 BVID 嵌入 B 站视频 |
| 追番列表 | 展示用户的 Bilibili 追番数据 |
| 收藏夹 | 展示用户的 Bilibili 视频收藏 |
| 电影 | 展示用户的 Bilibili 电影数据 |

**后台配置：** 主题选项 → 社交/集成 → Bilibili 设置
**缓存：** 数据通过 WordPress Transients 缓存，可在「缓存设置」页面管理。

### 5.2 Steam 游戏库

展示用户的 Steam 游戏库信息。
- 需要 Steam API Key
- 数据通过 Transients 缓存

### 5.3 QQ 集成

- 通过 QQ 号获取头像和用户信息
- 常用于评论区的用户头像显示（通过 `sakura/v1/qqinfo` API 端点）

### 5.4 Bangumi 追番

支持 Bangumi.tv 和 Bilibili 两种追番数据源。

### 5.5 MyAnimeList

支持获取用户在 MyAnimeList 上的动漫列表。

### 5.6 音乐播放器（APlayer）

通过 Meting API 集成网易云音乐、QQ 音乐等平台的歌曲播放。

### 5.7 图片上传服务

评论和文章中上传图片支持多种后端服务：

| 服务 | API 类型 |
|---|---|
| Imgur | 需要 Client ID |
| SM.MS | 需要 API Key |
| Chevereto | 需要 API Key |
| Lsky Pro | 需要 API Token |

---

## 六、评论系统

### 6.1 功能特性

- AJAX 无刷新提交评论
- 评论排序可配置（正序/倒序）
- 验证码支持（CAPTCHA / Cloudflare Turnstile / Vaptcha）
- QQ 头像自动获取
- Gravatar 代理支持（可配置自定义代理地址）

### 6.2 验证码配置

在主题选项中选择验证码服务：
- **CAPTCHA**（内置，通过 `sakura/v1/captcha/create` API 生成）
- **Cloudflare Turnstile**（隐私友好的替代方案）
- **Vaptcha**（支持语音/图片验证）

---

## 七、导航与交互体验

### 7.1 导航菜单

主题注册了一个导航菜单位置（`primary`），在后台 → 外观 → 菜单中配置。

**特性：**
- 支持多级下拉菜单
- 文本 Logo 显示（可在主题选项配置颜色、大小、字体）
- 移动端汉堡菜单

### 7.2 Pjax 无刷新导航

主题支持 Pjax 无刷新页面跳转（`poi_pjax` 选项），实现：
- 页面切换无闪烁
- 音乐播放不会中断
- 过渡动画平滑

开启后，需要给某些链接添加 `data-no-pjax` 属性来排除（如登录/退出链接）。

### 7.3 实时搜索（Live Search）

开启实时搜索后，搜索框会以模态窗口形式弹出，在输入时即时显示搜索结果，无需刷新页面。

**配置路径：** 主题选项 → 页面设置 → 开启实时搜索

### 7.4 灯箱效果（Lightbox）

文章图片点击放大支持三种灯箱方案，可在主题选项中切换：

| 方案 | 特点 |
|---|---|
| `baguetteBox` | 轻量级，纯 JS |
| `fancybox` | 功能丰富，支持画廊 |
| `lightgallery` | 功能完整，支持视频 |

### 7.5 平滑滚动

主题内置平滑滚动效果（`js/smoothscroll.js`），页面内锚点跳转时平滑过渡。

---

## 八、打赏与文章底部功能

### 8.1 打赏功能

在文章底部显示打赏区域，支持配置：
- 打赏链接（如爱发电、Ko-fi 等）
- 打赏二维码图片（支持最多 2 张，分别带独立链接）
- 在主题选项中配置奖励/打赏内容

**显示位置：** 单篇文章底部，位于作者信息和标签之前。

### 8.2 文章底部功能

每篇文章底部显示（可在主题选项关闭）：
- **许可协议** - Creative Commons 图标链接
- **打赏按钮** - 显示存钱罐图标
- **作者信息** - 头像、昵称、个人简介（quote 模式）
- **文章标签** - 标签图标和标签列表
- **更新日期** - 最后修改时间

---

## 九、主题外观自定义

### 9.1 色彩系统

| CSS 变量 | 说明 |
|---|---|
| `--theme-skin` | 主题主色 |
| `--theme-skin-matching` | 主题配色/强调色 |
| `--theme-skin-dark` | 暗黑模式主题色 |
| `--global-font-weight` | 全局字体粗细 |
| `--style_menu_radius` | 菜单圆角 |
| `--inline_code_background_color` | 行内代码背景色 |

### 9.2 暗黑模式

主题支持完整的暗黑模式，样式定义在 `css/dark.css` 中。

通过 `[theme-mode="dark"]` 选择器触发。主题相关颜色变量在暗黑模式下自动适配。

### 9.3 纪念模式

在特殊纪念日，可一键开启全站灰度模式：
```css
html {
    filter: grayscale(100%) !important;
}
```

### 9.4 自定义 CSS

在主题选项中可添加自定义 CSS 样式，会直接注入到页面 `<head>` 中。

### 9.5 字体设置

- 支持 Google Fonts 多语言字体加载
- 可配置 Font Awesome 图标库源（支持国内 CDN 加速）
- 支持自定义 Google Fonts API 域名（防止被墙）

---

## 十、页脚功能

### 10.1 一言（Hitokoto）

在页脚显示随机一句（一言），通过 `footer_yiyan` 选项开启。

### 10.2 站点统计

显示页面加载时间、数据库查询次数、内存使用情况：
```
Load Time 0.123 seconds | 12 Query | RAM Usage 3.45 MB
```

### 10.3 页脚信息

支持自定义页脚文本内容。

### 10.4 CDN 赞助商显示

显示 CDN / 云存储服务商信息（默认显示又拍云赞助）。

### 10.5 樱花瓣图标

页脚底部显示 Sakura 主题标志性的樱花 SVG 图标。

---

## 十一、性能与资源管理

### 11.1 CDN 资源加载

主题资源可通过 CDN 加速加载：
- **本地模式：** 从主题目录加载
- **CDN 模式：** 默认使用 jsDelivr CDN
- **可配置 CDN 地址：** 支持自定义 CDN 源

### 11.2 缓存管理

后台 → 缓存设置页面可管理：
- Bangumi 数据缓存
- Steam 数据缓存
- 归档信息缓存（30 秒过期）

### 11.3 PHP 错误报告级别

在主题选项中可控制 PHP 错误报告级别：
- `normal` - 仅显示严重错误（默认）
- `all` - 屏蔽大部分错误
- `inner` - 使用系统默认设置

### 11.4 资源预加载

主题在 `<head>` 中配置了资源预加载：
- Google Fonts 预连接（preconnect）
- Font Awesome 预加载（preload）
- Favicon 设置
- DNS 预获取（dns-prefetch）

---

## 十二、更新维护

### 12.1 主题更新源

支持三种更新源：
| 源 | 说明 |
|---|---|
| `github` | GitHub Releases 发布 |
| `upyun` | 又拍云 CDN 镜像 |
| `official_building` | 官方构建频道（可指定渠道） |

### 12.2 无缝更新

配合 [wp-seamless-update 插件](https://github.com/mirai-mamori/wp-seamless-update/releases/latest) 可实现主题自动更新。

---

## 快速参考卡片

**常用功能路径一览：**

| 想要做什么 | 路径 |
|---|---|
| 写文章 | WordPress 后台 → 文章 → 写文章 |
| 开启 AI 摘要 | 主题选项 → ChatGPT/AIGC → 配置 API；然后调用 API 生成 |
| 嵌入 B站视频 | 编辑器中使用 `[bilibili bvid="..."]` |
| 嵌入音乐播放器 | 编辑器中使用 `[ap layer]` |
| 插入画廊 | 编辑器中使用 `[gallery]` |
| 设置首页封面 | 主题选项 → 首页封面 |
| 配置展示区统计方块 | 主题选项 → 首页设置 → 展示区组件 |
| 调整首页组件顺序 | 主题选项 → 首页设置 → 组件排序 |
| 管理友情链接 | 后台 → 链接 |
| 检测链接状态 | 后台 → 链接 → 链接状态 |
| 添加分类图片 | 编辑分类/标签时上传图片 |
| 配置打赏 | 主题选项 → 页面设置 → 打赏区域 |
| 配置追番/Steam | 主题选项 → 社交/集成 |
| 选择灯箱效果 | 主题选项 → 页面设置 |
| 缓存管理 | 后台 → 缓存设置 |
| 自定义 CSS | 主题选项 → 全局设置 → 自定义样式 |
| 开启暗黑模式 | 主题选项 → 全局设置 |

---

## 附录 A：项目文件结构速查

```
sakurairo/
├── style.css                # 主题头部信息 & 基础样式
├── functions.php            # 主题初始化、选项加载、更新检查器
├── header.php               # HTML head、资源预加载、字体
├── footer.php               # 页脚（一言、统计、赞助）
├── index.php                # 组件驱动首页
├── page.php / single.php    # 页面/文章模板
├── archive.php / author.php # 归档/作者页
├── search.php / 404.php     # 搜索/404 页
├── exhibition.php           # 首页展示区
│
├── inc/                     # 核心 PHP
│   ├── api.php              # REST API 路由注册 (sakura/v1)
│   ├── template-tags.php    # 文章元信息函数
│   ├── theme-plus.php       # 视频/头像/时间/打赏/随机图
│   ├── customizer.php       # Kirki 可视化编辑器面板
│   ├── decorate.php         # 动态 CSS 生成（CSS 变量）
│   ├── swicher.php          # 前端 JS 配置注入
│   ├── post_metas.php       # 作者/分类/阅读时间
│   ├── cache_settings.php   # 缓存管理页面
│   ├── categories-images.php # 分类/标签图片
│   ├── link-status.php      # 友链状态检测
│   ├── article-highlight.php # 颜色分析工具
│   ├── word-stat.php        # 中英文混排字数统计
│   ├── blocks/iro_blocks.php # Gutenberg 编辑器块
│   ├── chatgpt/             # AI 辅助阅读
│   │   ├── chatgpt.php      # API 集成
│   │   ├── hooks.php        # WordPress 钩子
│   │   └── aigc-manage.php  # 后台管理页
│   └── classes/             # 外部服务集成
│       ├── Aplayer.php      # 音乐播放（Meting API）
│       ├── Bilibili.php     # B站视频
│       ├── BilibiliFavList* # B站收藏夹
│       ├── Cache.php        # Transient 缓存
│       ├── Captcha.php      # 验证码生成
│       ├── Images.php       # 图床上传（Imgur/SM.MS/Chevereto/Lsky）
│       ├── IpLocation.php   # IP 定位
│       ├── Meting.php       # Meting 音乐代理
│       ├── MyAnimeList.php  # MAL 追番
│       ├── QQ.php           # QQ 头像/信息
│       ├── Steam.php        # Steam 游戏库
│       ├── Turnstile.php    # Cloudflare Turnstile
│       ├── Vaptcha.php      # Vaptcha 验证
│       ├── bangumi.php      # Bangumi 追番
│       └── gallery.php      # 图片画廊
│
├── tpl/                     # 模板片段
│   ├── content-single.php   # 单文章（含 AI 摘要）
│   ├── content-thumb.php    # 缩略图列表
│   ├── content-thumbcard.php # 卡片列表
│   ├── content-page.php     # 页面内容
│   ├── content-search.php   # 搜索结果
│   ├── content-none.php     # 无内容提示
│   ├── section-article-function.php # 文章底部（许可/作者/标签）
│   ├── single-entry-header.php # 文章头部
│   └── single-image.php     # 图片文章
│
├── layouts/                 # 布局组件
│   ├── imgbox.php           # 图片框
│   ├── mo_toc_menu.php      # 移动端目录
│   ├── post-nextprev.php    # 上下篇文章
│   ├── sakura_header.php    # 导航栏布局
│   └── sidebox.php          # 侧边栏
│
├── opt/                     # Codestar Framework 选项系统
│   ├── option-framework.php # 框架入口
│   └── options/theme-options.php # 所有主题选项定义
│
├── css/                     # 样式表
│   ├── animation.css        # 动画
│   ├── dark.css             # 暗黑模式 `[theme-mode="dark"]`
│   ├── responsive.css       # 响应式
│   ├── sakura_header.css    # 导航栏
│   ├── shortcodes.css       # 短代码
│   ├── templates.css        # 通用模板
│   ├── wave.css             # 水波动画
│   └── content-style/       # 文章内容风格
│       ├── sakura.css       # 柔和圆润
│       └── github.css       # 简洁清晰
│
├── js/                      # JavaScript 构建输出
│   ├── app.js               # 主应用
│   ├── nav.js               # 导航
│   ├── page.js              # 页面功能
│   ├── polyfill.js          # 浏览器兼容
│   ├── smoothscroll.js      # 平滑滚动
│   └── lg-*.js              # 懒加载模块
│
├── update-checker/          # 主题更新检查器
├── languages/               # 多语言 (zh_CN/zh_TW/ja/fr)
└── _helper/                 # 开发辅助
    ├── do.py                # CSS/JS 压缩
    └── generate_changelog.sh # 更新日志生成
```

## 附录 B：开发参考

### 选项系统（Codestar Framework）

主题使用 Codestar Framework 管理选项，存储在 `iro_options` 数据库中：

```php
// 获取选项值
iro_opt(string $option_name, mixed $default = null): mixed

// 更新选项值
iro_opt_update(string $option_name, mixed $value): void
```

`iro_opt()` 定义在 `functions.php` 中，处理两种场景：
- **常规模式：** 从 `$GLOBALS['iro_options']` 读取
- **自定义器预览：** 从 `get_theme_mod('iro_options')` 读取预览值

### REST API 技术说明

所有自定义端点注册在 `sakura/v1` 命名空间（`inc/api.php`）：

```php
// 注册新端点示例
register_rest_route('sakura/v1', '/your-endpoint', array(
    'methods'             => 'GET',
    'callback'            => 'your_callback_function',
    'permission_callback' => '__return_true'
));
```

**现有端点速查：**

| 端点 | 方法 | 用途 |
|---|---|---|
| `/sakura/v1/image/upload` | POST | 图片上传 |
| `/sakura/v1/cache_search/json` | GET | 缓存搜索 |
| `/sakura/v1/gallery` | GET | 画廊图片 |
| `/sakura/v1/qqinfo/json` | GET | QQ 信息 |
| `/sakura/v1/qqinfo/avatar` | GET | QQ 头像 |
| `/sakura/v1/bangumi/bilibili` | POST | B站追番 |
| `/sakura/v1/bangumi` | POST | Bangumi 追番 |
| `/sakura/v1/steam` | POST | Steam 库 |
| `/sakura/v1/movies/bilibili` | POST | B站电影 |
| `/sakura/v1/favlist/bilibili` | GET | B站收藏 |
| `/sakura/v1/meting/aplayer` | GET | 音乐数据 |
| `/sakura/v1/captcha/create` | GET | 创建验证码 |
| `/sakura/v1/chatgpt` | GET | AI 摘要（管理员） |
| `/sakura/v1/chatgpt/annotate` | GET | AI 注释（管理员） |
| `/sakura/v1/archive_info` | GET | 归档信息 |

### 服务集成类命名空间

所有服务类位于 `inc/classes/`，使用命名空间 `Sakura\API`：

```php
use Sakura\API\QQ;
use Sakura\API\Cache;
use Sakura\API\Captcha;
use Sakura\API\BilibiliFavListCron;
```

### 动态 CSS 变量系统

主题选项通过 `inc/decorate.php` 转换为 CSS 自定义属性：

```css
:root {
    --theme-skin: [color];
    --theme-skin-matching: [color];
    --theme-skin-dark: [color];
    --global-font-weight: [value];
    --style_menu_radius: [value]px;
    --inline_code_background_color: [color];
    --front_background-transparency: [value];
}
```

### 资源加载机制

主题支持灵活的 CDN/本地资源加载（`functions.php` 中配置）：

```php
// 共享库路径
$shared_lib_basepath = iro_opt('shared_library_basepath')
    ? get_template_directory_uri()                          // 本地
    : (iro_opt('lib_cdn_path', 'https://fastly.jsdelivr.net/gh/mirai-mamori/Sakurairo@') . IRO_VERSION);  // CDN

// 核心库路径（同上模式）
$core_lib_basepath = iro_opt('core_library_basepath')
    ? get_template_directory_uri()
    : (iro_opt('lib_cdn_path', '...') . IRO_VERSION);

// 视觉资源路径
$vision_resource_basepath = iro_opt('vision_resource_basepath',
    'https://s.nmxc.ltd/sakurairo_vision/@3.0/');
```

### 前端 JS 配置注入

`inc/swicher.php` 将主题选项注入为全局 JS 对象 `iro_opt`，包含：
- Pjax、视频封面、AJAX URL
- CAPTCHA 端点、QQ API URL
- 评论排序、REST API nonce
- Google Analytics ID、Gravatar 代理
- 灯箱设置、注释数据

### 主题更新检查器

使用 YahnisElsts/Plugin-Update-Checker 库（`functions.php`）：

```php
switch (iro_opt('iro_update_source')) {
    case 'github':             // GitHub Releases
    case 'upyun':              // 又拍云 CDN
    case 'official_building':  // 官方构建频道
}
```

### 常用开发任务

**添加新主题选项：**
1. 在 `opt/options/theme-options.php` 中按 Codestar 语法定义字段
2. 使用 `iro_opt('option_key', 'default')` 读取
3. 可选：在 `inc/customizer.php` 添加 Kirki 字段
4. 可选：在 `inc/decorate.php` 添加 CSS 输出

**添加新 API 集成：**
1. 在 `inc/classes/` 中创建新类，命名空间 `Sakura\API`
2. 在 `inc/api.php` 中注册 REST 路由
3. 通过 `set_transient()` / `get_transient()` 缓存数据
4. 在主题选项页面添加配置项

**创建新页面模板：**
1. 在 `tpl/` 中创建模板文件
2. 使用 `get_template_part('tpl/your-file')` 引用
3. 如需加入首页，在 `index.php` 的组件排序系统中注册

---

## 资源

### references/
按主题分类的详细参考文档，在需要时加载：
- `theme-options.md` - 完整主题选项参考
- `api-endpoints.md` - REST API 端点文档
- `service-classes.md` - 服务集成类参考
- `template-hierarchy.md` - 模板文件层级
- `hooks-filters.md` - WordPress 钩子和过滤器参考

### scripts/
开发维护工具脚本：
- `update-pot.py` - 重新生成翻译模板 (.pot) 文件

---
> Source: [AKCX2002/sakurairo-theme](https://github.com/AKCX2002/sakurairo-theme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
