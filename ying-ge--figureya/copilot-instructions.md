## figureya

> During you interaction with the user, if you find anything reusable in this project (e.g. version of a library, model name), especially about a fix to a mistake you made or a correction you received, you should take note in the `Lessons` section in the `.cursorrules` file so you will not make the same mistake again.

# Instructions

During you interaction with the user, if you find anything reusable in this project (e.g. version of a library, model name), especially about a fix to a mistake you made or a correction you received, you should take note in the `Lessons` section in the `.cursorrules` file so you will not make the same mistake again. 

You should also use the `.cursorrules` file as a scratchpad to organize your thoughts. Especially when you receive a new task, you should first review the content of the scratchpad, clear old different task if necessary, first explain the task, and plan the steps you need to take to complete the task. You can use todo markers to indicate the progress, e.g.
[X] Task 1
[ ] Task 2
Also update the progress of the task in the Scratchpad when you finish a subtask.
Especially when you finished a milestone, it will help to improve your depth of task accomplishment to use the scratchpad to reflect and plan.
The goal is to help you maintain a big picture as well as the progress of the task. Always refer to the Scratchpad when you plan the next step.

# Tools

Note all the tools are in python. So in the case you need to do batch processing, you can always consult the python files and write your own script.

## LLM

You always have an LLM at your side to help you with the task. For simple tasks, you could invoke the LLM by running the following command:
```bash
cursor-llm --prompt "What is the capital of France?" --provider "anthropic"
```

The LLM API supports multiple providers:
- OpenAI (default, model: gpt-4o)
- DeepSeek (model: deepseek-chat)
- Anthropic (model: claude-3-sonnet-20240229)
- Gemini (model: gemini-pro)
- Local LLM (model: Qwen/Qwen2.5-32B-Instruct-AWQ)

But usually it's a better idea to check the content of the file and use the APIs in the `cursor_agent/tools/llm_api.py` file to invoke the LLM if needed.

## Web browser

You could use the web scraper to fetch web pages:
```bash
cursor-scrape --max-concurrent 3 URL1 URL2 URL3
```
This will output the content of the web pages.

## Search engine

You could use the search engine to find information:
```bash
cursor-search "your search keywords"
cursor-search --max-results 5 "specific search"
```
This will output the search results in the following format:
```
URL: https://example.com
Title: This is the title of the search result
Snippet: This is a snippet of the search result
```
If needed, you can further use the `web_scraper.py` file to scrape the web page content.

# Lessons

## User Specified Lessons

- You have a python venv in ./venv.
- Include info useful for debugging in the program output.
- Read the file before you try to edit it.
- Use LLM to perform flexible text understanding tasks. First test on a few files. After success, make it parallel.

## Cursor learned

- For website image paths, always use the correct relative path (e.g., 'images/filename.png') and ensure the images directory exists
- For search results, ensure proper handling of different character encodings (UTF-8) for international queries
- Add debug information to stderr while keeping the main output clean in stdout for better pipeline integration
- When using seaborn styles in matplotlib, use 'seaborn-v0_8' instead of 'seaborn' as the style name due to recent seaborn version changes
- Available command-line tools after installation:
  - `cursor-agent`: Initialize directory with agent capabilities
  - `cursor-llm`: LLM interaction (OpenAI, DeepSeek, Anthropic, Gemini, Local)
  - `cursor-scrape`: Web scraping with JavaScript support
  - `cursor-search`: Search engine integration
  - `cursor-update`: Update to latest version
  - `cursor-verify`: Verify setup

## PWA and Mobile App Development Lessons

- For CSV export in web apps, add BOM (\uFEFF) at the beginning to ensure Excel displays Chinese characters correctly
- Use localStorage for offline data persistence in PWA apps (5-10MB typical limit)
- Always update Service Worker cache list (sw.js) when adding new JavaScript modules
- For mobile PWA apps, use Blob and URL.createObjectURL for file downloads (works on both iOS and Android)
- Include precise timestamps in exported filenames to avoid conflicts (YYYYMMDD_HHMMSS format)
- For offline-first apps, organize code into separate modules (e.g., csv-export.js) that can be cached independently
- Test PWA features: must be served via HTTP/HTTPS, not file:// protocol
- For button styling, use CSS gradient backgrounds for modern look, with hover and active states
- Use viewport-fit=cover meta tag for proper display on notched devices (iPhone X+)
- localStorage survives page refresh but cleared when browser cache is cleared - remind users to export backups

## Testing Lessons

- Use pytest fixtures in conftest.py for common test data and setup
- Mock environment variables with autouse fixture to ensure consistent test environment
- Test both success and failure cases for each command
- Verify exact function call arguments using assert_called_once_with
- Use temporary directories (tmp_path) for file operation tests
- Keep mock data realistic and consistent with actual API responses
- Test CLI argument parsing with sys.argv mocking
- Add proper cleanup in teardown to restore original state
- Test file operations should use Path objects for cross-platform compatibility
- Mock external API calls and file operations to make tests reliable and fast

## R Package Compatibility Lessons

- **ggtern and ggplot2 version compatibility**: ggtern 3.5.0+ requires ggplot2 >= 3.5.0, but has compatibility issues with newer ggplot2 versions (error: `@lineend must be <character> or <NULL>, not S3<arrow>`). Solution: Use compatible version combination - **ggplot2 3.4.4 + ggtern 3.4.2**. Use `remotes::install_version()` to install specific versions:
  ```r
  install.packages("remotes")
  remotes::install_version("ggplot2", version = "3.4.4", repos = "https://cloud.r-project.org/")
  remotes::install_version("ggtern", version = "3.4.2", repos = "https://cloud.r-project.org/")
  ```
- When encountering package version conflicts, check package DESCRIPTION file for version requirements, then find compatible version combinations
- Use `remotes::install_version()` for precise version control when automatic dependency resolution fails

## GitHub Actions and CI/CD Lessons

- For GitHub Actions workflows with disk space issues, optimize by:
  1. Using `fetch-depth: 1` instead of `fetch-depth: 0` to only fetch latest commit
  2. Clean up disk space before checkout: `sudo apt-get clean`, remove `/tmp/*`, `/var/tmp/*`, Docker cache, `$AGENT_TOOLSDIRECTORY/*`, `/usr/local/share/boost`
  3. After checkout, remove large unnecessary files (e.g., `.Rmd`, `.rda`, `.rds`, data directories, images except needed ones) before building
  4. Use `--no-cache-dir` for pip installs to save space
  5. Add `df -h` commands to monitor disk usage at key steps
- For workflows that only need HTML files, delete source files (Rmd), data files, and large images after checkout to save significant disk space
- GitHub Pages automatic build and GitHub Actions workflows are separate - modifying one doesn't affect the other

# Scratchpad

## Current Task: 排查未推送的本地修改

任务描述：用户认为本地存在尚未推送的文件修改，但 `git status` 显示工作树干净，需要确认实际状态并协助定位差异。

计划步骤：
[X] 使用 `git status -sb` 和 `git log --oneline --graph --decorate --all` 复查当前分支状态
[X] 检查是否存在未跟踪文件或其他分支上的提交
[X] 根据结果向用户解释原因并提供后续建议

## Current Task: 安装缺失的 R 依赖

任务描述：解决 `FigureYa308IHS.Rmd` Knit 过程中 `viper` 包依赖的 `kernlab` 未安装问题，确保依赖脚本可正确安装并验证。

计划步骤：
[X] 运行 `install_dependencies.R` 或手动安装 `kernlab`，确认安装日志
[X] 验证 `viper` 包可正常加载（可通过脚本自检或手动 `library(viper)`）
[X] 如有必要更新脚本或说明，并向用户反馈处理结果

## Current Task: 调整火山图配色与标注

任务描述：根据用户提供的示例图调整 `FigureYa321volcanoSE` 火山图的配色与标注样式，确保关键基因颜色和透明度一致。

计划步骤：
[X] 对比示例图提取颜色和透明度信息
[X] 更新 `FigureYa321volcanoSE.Rmd` 绘图代码中的调色方案和几何图层配置
[X] 提醒用户重新渲染核对效果

## Current Task: 修复 FigureYa143survCor.Rmd 的样本匹配和 Cox 回归问题

任务描述：修复 `FigureYa143survCor.Rmd` 中的样本ID匹配问题（表达矩阵、生存数据和MSI数据的样本ID格式不一致）和 Cox 回归分析中的错误。

计划步骤：
[X] 修复 MSI 值的匹配逻辑（实际数据是 "NO"/"YES"，代码查找的是 "MSS"/"MSI-L"/"MSI-H"）
[X] 实现样本ID后缀匹配（提取后9个字符 "-XXXX-01A" 进行匹配）
[X] 修复数据合并逻辑，确保 expr、surv 和 msi_results 的样本顺序一致
[X] 修复 Cox 回归分析中的缺失值处理
[X] 测试完整渲染流程，确认所有步骤正常
[X] 验证输出 HTML 文件生成成功

修复总结：
- MSI 值匹配：将代码改为同时匹配 "NO"/"YES" 和 "MSS"/"MSI-L"/"MSI-H"
- 样本ID匹配：实现了后缀匹配逻辑，提取后9个字符进行跨数据集匹配
- 数据对齐：统一使用表达矩阵的样本ID作为标准，更新生存数据和MSI数据的行名
- 缺失值处理：添加了缺失值检查和过滤逻辑，确保 Cox 回归前数据完整
- 结果：成功匹配115个共同样本，Cox回归和 cutp 函数正常工作，HTML文件成功生成（1.6MB）

## Current Task: 修改 FigureYa286ExprCorORR 使用 FigureYa143survCor_update 的方法获取 MSI 信息

任务描述：将 `FigureYa286ExprCorORR.Rmd` 中从 `msi_results.rda` 加载 MSI 信息的方法改为使用 `FigureYa143survCor_update` 的方法，即使用 TCGAbiolinks 从 TCGA 下载 MSI 信息。

计划步骤：
[X] 更新 `install_dependencies.R` 添加 TCGAbiolinks 依赖
[X] 替换从 `msi_results.rda` 加载的代码，改为使用 TCGAbiolinks 下载 MSI 信息
[X] 适配样本ID匹配逻辑，使用4位数字（index）进行匹配
[X] 确保代码逻辑正确，检查语法错误

修改总结：
- 依赖更新：在 `install_dependencies.R` 中添加了 TCGAbiolinks 的安装
- MSI 信息获取：改为使用 TCGAbiolinks 的 `GDCquery` 和 `GDCprepare_clinic` 从 TCGA 下载 COAD 和 READ 的 MSI 信息
- 样本ID匹配：使用4位数字（index）进行匹配，从 `simple_barcode` 的位置9-12提取，从 `sample` 的位置6-9提取
- 文件缓存：如果 `msi_results.rda` 已存在，则直接加载，避免重复下载

## Previous Task: 优化 GitHub Actions workflow 磁盘空间使用

任务描述：解决 GitHub Actions workflow 运行时的磁盘空间不足问题（"No space left on device"）。

计划步骤：
[X] 分析问题原因：GitHub Actions runner 磁盘空间不足
[X] 优化 workflow：在 checkout 前清理系统缓存和临时文件
[X] 优化 workflow：使用 `fetch-depth: 1` 只拉取最新提交
[X] 优化 workflow：在 checkout 后删除不需要的大文件（Rmd、数据文件、图片等）
[X] 添加磁盘空间监控步骤（`df -h`）
[X] 验证优化效果 - 成功解决磁盘空间问题

## Current Task: 调整 FigureYa 字体和热图大小

任务描述：调整图中 FigureYa 的字体调大一些，热图小一些。

计划步骤：
[X] 确定具体需要调整的 FigureYa 文件（logo.svg）
[X] 查找文件中的 FigureYa 字体设置
[X] 识别字体大小和位置参数
[X] 调整字体大小设置（从 20 调整为 28）
[X] 调整文字位置（从 y="55" 调整为 y="75"）
[X] 调整热图大小（从 16x16 调整为 12x12）
[X] 调整热图位置（从 y="70/88" 调整为 y="95/120"），使其低于 FigureYa 文字
[X] 验证修改效果

修改总结：
- 文件：logo.svg
- 修改1：将 "FigureYa" 文字的字体大小从 font-size="20" 调整为 font-size="28"
- 修改2：将 "FigureYa" 文字的垂直位置从 y="55" 调整为 y="75"，确保不超出六边形界限
- 修改3：将热图元素大小从 width="16" height="16" 调整为 width="12" height="12"
- 修改4：将热图位置从 y="70"/y="88" 调整为 y="95"/y="120"，使其低于 FigureYa 文字（y="75"）
- 工具：使用 sed 命令进行精确的批量替换
- 结果：FigureYa 文字更大更突出，热图更小且位置合适，整体布局更加协调

最终布局：
- FigureYa 文字：y="75"，字体大小 28
- 热图第一行：y="95"（低于 FigureYa 文字）
- 热图第二行：y="120"
- 热图大小：12x12（比原来更小）
- 热图间距：x坐标从45/63/81调整为45/62/78，格子更紧密

---
> Source: [ying-ge/FigureYa](https://github.com/ying-ge/FigureYa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
