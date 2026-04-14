## donglua-github-io

> 这是 Donglu (DL) 的个人技术博客。基于 [Jekyll](https://jekyllrb.com/) (Ruby 静态网站生成器) 构建，使用默认的 `minima` 主题，通过 Markdown 文件生成静态页面。

# Jekyll Blog Project Context

## 1. 项目概览 (Project Overview)
这是 Donglu (DL) 的个人技术博客。基于 [Jekyll](https://jekyllrb.com/) (Ruby 静态网站生成器) 构建，使用默认的 `minima` 主题，通过 Markdown 文件生成静态页面。

- **主要语言:** Ruby, Markdown, HTML, YAML
- **框架版本:** Jekyll ~> 4.4.1
- **使用主题:** minima
- **使用插件:** jekyll-feed, nokogiri, mermaid_processor.rb (自定义插件)

## 2. 核心行为准则 (Core Directives)
- **写作风格 (Writing Style):** 
  - **硬核技术风**：行文简洁客观。避免夸张营销用语（如"炸裂"、"令人绝望"等）和过多 emoji，保持清晰干练的技术叙述。
  - **逻辑严密**：文章结构必须具备强逻辑性。通常遵循"问题现象 -> 根本原因 -> 解决方案 -> 原理扩展/总结"的金字塔结构，层层递进。
  - **精炼准确**：结论要一针见血，避免啰嗦和无关紧要的发散。如果问题出在特定场景/特定前置条件，必须在开篇明确指出，不泛泛而谈。
- **内容规范:** 除非有特殊要求，文章内容及代码注释请优先使用简体中文。
- **文章创建规范:** 新文章必须存放在 `_posts/` 目录下，文件命名格式必须为 `YYYY-MM-DD-title.md`。必须在文件顶部使用 YAML Front Matter 指定 layout, title 等元数据。

## 3. 构建与运行 (Building and Running)
本项目使用 Bundler 管理 Ruby gem 依赖。每次执行 Jekyll 相关命令前，必须使用 `bundle exec` 以确保使用正确的 gem 版本。

- **安装依赖:**
  ```bash
  bundle install
  ```
- **本地开发服务器运行:**
  ```bash
  bundle exec jekyll serve --livereload
  ```
  *(注：开启 livereload 会自动刷新，但修改 `_config.yml` 后需要重启服务器)*
- **构建静态站点:**
  ```bash
  bundle exec jekyll build
  ```
  *(生成的静态文件会存放在 `_site/` 目录下)*

## 4. 项目结构 (Project Structure)
- `_config.yml`: 站点全局配置 (title, url, theme, plugins 等)。
- `Gemfile` / `Gemfile.lock`: Ruby 依赖管理文件。
- `_posts/`: Markdown 格式的博客文章存放目录。
- `_layouts/`: 自定义 HTML 布局模板 (如 `post.html`)。
- `_includes/`: 可复用的 HTML 代码片段 (如 `google-analytics.html`, `mermaid.html`)。
- `_plugins/`: 自定义 Ruby 插件 (如用于渲染 Mermaid 图表的 `mermaid_processor.rb`)。
- `_site/`: 生成的静态站点目录 (已被 Git 忽略)。
- `.github/workflows/jekyll.yml`: 用于自动化构建和部署的 GitHub Actions 工作流。

## 5. 特定功能 (Specific Features)
- **Mermaid 支持:** 站点内置了对 Mermaid 的支持。通过 `_plugins/mermaid_processor.rb` 和 `_includes/mermaid.html` 渲染图表，在编写 Markdown 时可直接使用 mermaid 语法块。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/donglua)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/donglua)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
