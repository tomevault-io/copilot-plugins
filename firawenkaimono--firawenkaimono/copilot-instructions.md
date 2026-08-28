## firawenkaimono

> 本仓库用于将 Fira Code 的西文与编程连字，和 LXGW WenKai Mono（霞鹜文楷等宽）

# Repository guide

本仓库用于将 Fira Code 的西文与编程连字，和 LXGW WenKai Mono（霞鹜文楷等宽）
的中文及补充字符合并为 `Fira WenKai Mono`。

## 工具与环境

- Python 环境必须由 `uv` 管理；项目版本和依赖分别记录在 `.python-version`、
  `pyproject.toml` 和 `uv.lock`。
- `source/*.ttf` 必须由 Git LFS 管理；`.gitattributes` 中的 LFS 规则不可删除。
- FontForge 是外部字体引擎，不是 PyPI 依赖，也不由 uv 安装。构建时需要保证
  `fontforge` 在 `PATH` 中，或通过 `--fontforge` / `FONTFORGE` 指定可执行文件。
- Python 侧使用 `fontTools` 检查字体表，使用 `uharfbuzz` 检查 Fira Code 的实际连字塑形。
- 如果执行环境的默认 uv 缓存目录不可写，可以临时设置
  `UV_CACHE_DIR=/tmp/firawenkaimono-uv-cache`；不要把缓存提交到仓库。

## 目录与入口

- `source/`：六个由 Git LFS 跟踪的上游 TTF，包含 Light、Regular、Medium 三组配对源字体。
- `pages/FiraWenKaiMono-logo.png`：项目主 Logo 的透明 PNG；以代码尖括号、东方笔势和
  暖杏色印记表现中西文字体的结合，同时用作 Pages 导航标志、favicon 和 Hero 背景水印。
- `OFL.txt`：字体随附的 SIL Open Font License 1.1 正文及上游版权归属；必须随字体包发布。
- `scripts/build_fonts.py`：面向使用者的构建及验证入口。
- `scripts/merge_fontforge.py`：由 FontForge 内置 Python 解释器执行的合并工作脚本。
- `dist/`：生成的三款 TTF 和 `FiraWenKaiMono.zip`；不要手工修改这些产物，应通过
  构建脚本重建。
- `pages/`：GitHub Pages 静态站点源码，`index.html` 直接作为站点首页。
- `scripts/prepare_pages.py`：将 Pages 源码与字体产物组装到 `_site/`。
- `.github/workflows/build-and-pages.yml`：CI 构建、ZIP 产物上传、Release 发布和 Pages
  部署流程。

## 构建规则

字重必须按下表配对，禁止跨字重合并：

| 成品样式 | Fira Code | LXGW WenKai Mono | OS/2 字重 |
| --- | --- | --- | ---: |
| Light | `FiraCode-Light.ttf` | `LXGWWenKaiMono-Light.ttf` | 300 |
| Regular | `FiraCode-Regular.ttf` | `LXGWWenKaiMono-Regular.ttf` | 400 |
| Medium | `FiraCode-Medium.ttf` | `LXGWWenKaiMono-Medium.ttf` | 500 |

合并时遵守以下不变量：

1. 以 Fira Code 为目标字体调用 FontForge `mergeFonts`，让重叠 Unicode 码位保留
   Fira Code 字形、度量和编程连字。
2. 先将中文源字体的 em 归一到 Fira Code 的 em。
3. 一个中文字符单元必须恰好等于两个西文单元；当前基线为 `A=1200`、`中=2400`。
4. 中文轮廓在 em 归一后必须等比放大到 `1.15` 倍，但前进宽度仍保持 2400；放大后的
   轮廓需要在字符单元中水平居中。轮廓缩放与字符单元宽度是两项独立约束。缩放后还
   必须根据代表字符的轮廓中心中位数自动垂直平移，使中文视觉中心与西文大写及数字
   `H O 0 8` 的视觉中心对齐；禁止用固定偏移量代替动态计算。
5. 字体家族名固定为 `Fira WenKai Mono`，PostScript 名为
   `FiraWenKaiMono-{Light,Regular,Medium}`。必须替换继承的旧名称记录，避免 Light 和
   Medium 被系统识别为 Regular。
6. 必须保留 Fira Code、LXGW WenKai 和 Klee 项目的版权归属，以及 SIL Open Font
   License 1.1 的名称表字段和链接。

当前三个成品各覆盖 46,706 个 Unicode 字符。这是回归基线，不应在源字体未变化时下降。

## 常用命令

安装/同步 Python 环境：

```bash
uv sync
```

构建并自动验证全部字体：

```bash
uv run python scripts/build_fonts.py
```

构建成功后必须同时生成 `dist/FiraWenKaiMono.zip`，其中包含三个 TTF 文件和 `OFL.txt`。

FontForge 不在 `PATH` 时：

```bash
uv run python scripts/build_fonts.py --fontforge /path/to/fontforge
```

仅检查 Python 脚本和 uv 锁文件：

```bash
uv lock --check
uv run python -m compileall -q scripts
```

预览演示网页：

```bash
uv run python scripts/prepare_pages.py
uv run python -m http.server 8000 --directory _site
```

浏览 `http://localhost:8000/`。`_site/` 必须与 GitHub Pages 的发布内容保持一致。

## 验证要求

`build_fonts.py` 成功退出前必须检查：

- 家族名、子家族名、PostScript 名和 OS/2 字重正确；
- `A`、`0`、`中`、`文`、全角逗号等代表字符存在；
- 西文与中文前进宽度严格为 1:2；
- 代表性中文轮廓相对归一后的源字体严格放大到 1.15 倍，同时前进宽度不变；
- 代表性中英文轮廓中心的中位数差不超过 2 units，中文横向中心与字符单元中心的偏差
  不超过 20 units；
- GSUB 表存在，并且 HarfBuzz 对 `-> => != ===` 启用/禁用 `calt` 时结果不同；
- SIL OFL 1.1 许可证说明和 URL 存在。

演示页使用 TTF，以保证构建稳定。此超大字库的 WOFF2 Brotli 压缩在受限 ARM 环境中
可能长时间无输出；除非任务明确要求，不要把 WOFF2 生成加入默认构建流程。

## GitHub Actions

- Pull Request：构建、验证并上传字体 ZIP，不部署 Pages。
- 推送到 `main` 或手动触发：执行完整构建、创建 GitHub Release，并部署 Pages。
- checkout 步骤必须设置 `lfs: true`，否则构建读取到的只是 LFS 指针文件。
- Pages 站点首页直接位于 `_site/index.html`，字体与 `OFL.txt` 位于 `_site/fonts/`；
  `pages/style.css` 必须使用相对的 `fonts/` 路径。
- 页面主标题旁使用带 GitHub Logo 的独立链接跳转到仓库，下载入口链接到 GitHub
  Releases 页面；ZIP 不复制进 Pages 站点。
- ZIP 作为普通 Actions artifact 上传，并作为 GitHub Release 附件发布。
- Release 使用 `build-<run_number>` 标签和对应标题，附带 `FiraWenKaiMono.zip`，并标记为
  latest；重复执行同一次 workflow run 时必须复用已有 Release，不能因标签重复而失败。
- Release job 只应获得 `actions: read` 和 `contents: write`；其他 job 不应获得仓库内容写权限。
- 仓库 Pages 必须预先选择 GitHub Actions 作为发布源。当前静态站点直接使用
  `upload-pages-artifact` 和 `deploy-pages`，不需要 `configure-pages` 查询站点元数据。
- 更新工作流 action 版本时优先参考各 action 的官方仓库与 GitHub Pages 官方文档。

---
> Source: [FiraWenkaiMono/FiraWenkaiMono](https://github.com/FiraWenkaiMono/FiraWenkaiMono) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
