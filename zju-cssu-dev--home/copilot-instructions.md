## home

> 浙江大学计算机科学与技术学院学生会网站源码仓库。线上站点地址：

# Home

浙江大学计算机科学与技术学院学生会网站源码仓库。线上站点地址：
[https://zju-cssu-dev.github.io/home/](https://zju-cssu-dev.github.io/home/)

本仓库使用 `MkDocs + Material for MkDocs` 构建静态站点，内容主要面向学院同学，覆盖通知发布、意见反馈、知识共享与资源分享四个板块。

## 仓库定位

- `docs/`：站点内容源文件，绝大多数页面维护都发生在这里。
- `mkdocs.yml`：站点配置中心，包含导航栏、主题、插件、额外 CSS/JS 等配置。
- `main.py`：`mkdocs-macros-plugin` 的入口，目前提供通知中心卡片聚合宏。
- `.github/workflows/publish.yml`：GitHub Pages 自动部署工作流，`main` 分支有新提交后会自动发布。
- `requirements.txt`：本地开发与 CI 构建所需 Python 依赖。

## 仓库结构

当前仓库可以按下面的方式理解：

```text
home/
├── docs/
│   ├── index.md                  # 首页
│   ├── intro.md                  # 网站介绍与使用指南
│   ├── Notification/             # 通知中心
│   │   ├── Academic/             # 教学事务
│   │   ├── Awards/               # 评优评先和资助
│   │   ├── Growth/               # 形策二课
│   │   ├── Research/             # 学业科研
│   │   ├── Career/               # 就业发展
│   │   ├── Campus/               # 校园信息
│   │   ├── Exchange/             # 对外交流
│   │   └── Activities/           # 文体活动
│   ├── 意见反馈/                  # Issue 教程、匿名反馈入口、回应内容
│   ├── 知识共享/                  # 培养方案、课程笔记、课外学习资料
│   ├── 资源分享/                  # 个人笔记、个人项目、投稿指南
│   ├── macros/                   # Jinja 宏模板
│   ├── stylesheets/              # 自定义样式
│   └── javascripts/              # 自定义脚本
├── mkdocs.yml
├── main.py
└── requirements.txt
```

## 站点栏目与内容映射

站点当前导航以 [`mkdocs.yml`](mkdocs.yml) 为准，顶层栏目如下：

- `主页`：[`docs/index.md`](docs/index.md)
- `通知中心`：[`docs/Notification/`](docs/Notification)
- `意见反馈`：[`docs/意见反馈/`](docs/意见反馈)
- `知识共享`：[`docs/知识共享/`](docs/知识共享)
- `资源分享`：[`docs/资源分享/`](docs/资源分享)

说明：

- 仓库里存在部分历史目录或未接入导航的内容，是否会出现在站点中，以 [`mkdocs.yml`](mkdocs.yml) 的 `nav` 配置为准。
- 新增页面后，如果希望它出现在导航栏或左侧目录中，需要同步修改 `mkdocs.yml`。

## 本地开发

### 环境准备

需要以下工具：

- `git`
- `uv`：用于创建虚拟环境与安装依赖。安装说明见 [uv 官方文档](https://docs.astral.sh/uv/getting-started/installation/)

### 克隆仓库

```bash
# HTTPS
git clone https://github.com/ZJU-CSSU-Dev/home.git

# 或 SSH
git clone git@github.com:ZJU-CSSU-Dev/home.git
cd home
```

### 安装依赖

```bash
uv venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
uv pip install -r requirements.txt
```

### 本地预览

```bash
mkdocs serve
```

如果没有激活虚拟环境，也可以直接运行：

```bash
uv run mkdocs serve
```

默认访问地址为 [http://127.0.0.1:8000](http://127.0.0.1:8000)。

### 本地构建检查

在提交前，建议至少执行一次静态构建，检查导航、链接和页面渲染是否正常：

```bash
uv run mkdocs build
```

## 内容维护指南

### 1. 新增或修改普通页面

常规内容页面直接在 `docs/` 下对应目录维护 Markdown 文件即可，例如：

- 首页：[`docs/index.md`](docs/index.md)
- 知识共享概览：[`docs/知识共享/index.md`](docs/知识共享/index.md)
- 资源分享提交说明：[`docs/资源分享/提交指南.md`](docs/资源分享/提交指南.md)

如果页面需要进入导航栏，还要同步更新 [`mkdocs.yml`](mkdocs.yml) 的 `nav`。

### 2. 维护通知中心

通知中心不是手工把所有通知写在一个页面里，而是采用“分类页定义卡片，首页自动聚合”的方式：

- 各分类页面在 YAML front matter 中维护 `cards`
- [`main.py`](main.py) 中的 `collect_notifications()` 会扫描 `docs/Notification/` 下所有 Markdown 文件
- [`docs/Notification/index.md`](docs/Notification/index.md) 会调用这个宏，把所有通知统一展示出来
- [`docs/macros/card_macro.html`](docs/macros/card_macro.html) 负责卡片渲染与按 `ddl` 倒序展示

一个通知卡片的典型写法如下：

```yaml
---
cards:
  - ddl: 2026-03-08
    title: 社会实践第三课堂记点补充认定
    detail: 这里写摘要信息
    href: https://example.com
    tags:
      - text: 社会实践
        class: tag-category
      - text: 重要
        class: tag-priority
---
```

字段说明：

- `title`：通知标题
- `detail`：通知摘要
- `href`：跳转链接
- `ddl`：可选，截止日期；存在时会参与排序与状态展示
- `tags`：可选，卡片标签列表

如果只是更新某一类通知，优先修改对应分类页面，而不是直接改通知中心概览页。

### 3. 维护样式和交互

- 自定义样式位于 [`docs/stylesheets/`](docs/stylesheets)
- 自定义脚本位于 [`docs/javascripts/`](docs/javascripts)
- 宏模板位于 [`docs/macros/`](docs/macros)

相关资源是否生效，同样以 [`mkdocs.yml`](mkdocs.yml) 中的 `extra_css`、`extra_javascript` 和 `plugins` 配置为准。

## 部署说明

仓库已配置 GitHub Actions 自动部署：

- 工作流文件：[`.github/workflows/publish.yml`](.github/workflows/publish.yml)
- 触发条件：向 `main` 分支推送代码
- 部署方式：工作流执行 `mkdocs gh-deploy --force`，将生成内容发布到 GitHub Pages

这意味着：

- 本地开发主要关注内容正确性与构建通过
- 合并到 `main` 后会自动上线，无需手工部署

## 贡献方式

如果是维护者，建议流程如下：

1. 新建分支修改内容
2. 本地执行 `uv run mkdocs build`
3. 确认页面和导航正常后发起 Pull Request
4. 合并到 `main`，等待 GitHub Actions 自动发布

如果是面向普通投稿者，可参考资源分享区的投稿说明：

- [`docs/资源分享/提交指南.md`](docs/资源分享/提交指南.md)

## 常见维护建议

- 新增页面后，先检查它是否真的被 `nav` 引用，否则页面可能存在但入口不可见。
- 修改通知内容时，优先保持卡片字段完整，尤其是 `title`、`detail` 和 `href`。
- 改动样式或宏后，最好本地跑一次 `mkdocs serve` 手动检查首页、通知中心和相关栏目页。
- 如果页面资源较多，提交时尽量保持目录命名清晰，避免中英文目录并存但用途不明。

---
> Source: [ZJU-CSSU-Dev/home](https://github.com/ZJU-CSSU-Dev/home) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
