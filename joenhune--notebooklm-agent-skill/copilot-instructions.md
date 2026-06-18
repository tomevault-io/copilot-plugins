## notebooklm-agent-skill

> 通过 notebooklm-py 查询 Google NotebookLM，获取基于用户上传文档的 Gemini source-grounded 回答。支持认证管理、Notebook 库管理、自动提问与追问。大幅减少幻觉——回答仅来自用户上传的文档。


# NotebookLM 查询助手

通过 notebooklm-py（RPC API）直接操控 Google NotebookLM：创建 Notebook、上传本地文件为 Source、查询文档、生成内容。无需浏览器自动化，速度更快更稳定。

## 触发条件

当用户：
- 提到 NotebookLM
- 分享 NotebookLM URL（`https://notebooklm.google.com/notebook/...`）
- 要求查询文档库/知识库
- 使用类似"问 NotebookLM"、"查我的文档"、"查 notebook"等短语
- 要求将本地文件同步到 NotebookLM

## 核心规则

1. **所有命令必须通过 `run.py` 调用**
2. **Skill 目录**：`~/.claude/skills/notebooklm`
3. **首次运行自动创建 `.venv`** 并安装依赖（含 Chromium），无需手动安装
4. **CLI 模式**用于 notebooklm 命令；**脚本模式**用于自定义功能

## 调用格式

```bash
# CLI 命令（notebooklm-py 原生命令）
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli <命令> [参数...]

# 自定义脚本
cd ~/.claude/skills/notebooklm && python3 scripts/run.py <脚本名> [参数...]
```

> ⚠️ 必须先 cd 到 skill 目录，再调用 run.py。

---

## 工作流

### Step 1：检查认证状态

```bash
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli auth check --test
```

如未认证或 cookie 过期，执行 Step 2。

### Step 2：认证（首次使用 / cookie 过期）

**推荐方式（使用系统 Chrome + 已有登录态）：**

```bash
cd ~/.claude/skills/notebooklm && python3 scripts/run.py login_chrome.py
```

- 直接调用系统 Chrome，**自动继承已有的 Google 登录状态**
- 如果 Chrome 已登录 Google，打开后直接显示 NotebookLM 主页，按回车即可
- 无需重新输入密码

**备选方式（使用 Playwright Chromium）：**

```bash
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli login
```

- 使用独立 Chromium 实例，需要重新登录 Google

### Step 3：管理 Notebook

```bash
# 列出所有 Notebook
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli list

# 创建新 Notebook
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli create "研究主题名称"

# 激活某个 Notebook（后续命令默认使用它）
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli use <notebook_id>

# 重命名 Notebook
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli rename "新名称"

# 删除 Notebook
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli delete <notebook_id>

# 查看 Notebook 元数据和 Source 列表
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli metadata --json
```

### Step 4：管理 Source（文档源）

```bash
# 添加 URL
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli source add "https://example.com/article"

# 添加 YouTube 视频
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli source add "https://www.youtube.com/watch?v=..."

# 添加本地文件（PDF / Markdown / Word / 文本）
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli source add "./paper.pdf"
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli source add "./notes.md"

# 添加 Google Drive 文件
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli source add "https://docs.google.com/..."

# 列出已有 Source
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli source list

# 获取某个 Source 的全文
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli source fulltext <source_id>

# Web 研究并自动导入
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli source add-research "AI robotics"
```

### Step 5：批量同步本地文件（自定义脚本）

> **⚠️ 重要限制：每个 Notebook 最多 50 个 Source。** 超过此限制的上传会被 API 拒绝。
> 大量文件应拆分到多个 Notebook，每个控制在 50 以内。

> **⚠️ 幻影错误：API 有时报 RPC/连接错误但实际已上传成功。**
> 不要盲目重试——这会造成重复 Source。上传完成后用 `dedup_sources.py` 去重。

**推荐流程（批量上传 > 5 个文件）：**

```bash
# 1. 创建文件清单（一行一个绝对路径，# 开头为注释）
cat > /tmp/upload_list.txt << 'EOF'
# 访谈记录
/path/to/interview_1.md
/path/to/interview_2.md
# PDF 资料
/path/to/report.pdf
EOF

# 2. 批量上传（自动检查 50 上限，不重试，3 秒间隔）
#    上传完成后自动检查来源状态并报告错误
cd ~/.claude/skills/notebooklm && .venv/bin/python3 scripts/batch_upload.py \
  --notebook-id <notebook_id> \
  --file-list /tmp/upload_list.txt

# 3. 上传后去重（清理幻影错误导致的重复）
cd ~/.claude/skills/notebooklm && .venv/bin/python3 scripts/dedup_sources.py <notebook_id>

# 4. 修复失败来源（自动查找原始文件 → 删除错误来源 → 重新上传）
cd ~/.claude/skills/notebooklm && .venv/bin/python3 scripts/repair_sources.py \
  --notebook-id <notebook_id> \
  --search-dirs /path/to/dir1 /path/to/dir2

# 5. 检查来源健康状态（不传 notebook_id 则检查所有 notebook）
cd ~/.claude/skills/notebooklm && .venv/bin/python3 scripts/check_sources.py [notebook_id]
```

> 💡 `batch_upload.py`、`repair_sources.py`、`check_sources.py`、`dedup_sources.py`
> 必须用 `.venv/bin/python3` 直接运行，不要通过 `run.py` 调用（stdin 会冲突）。

> 💡 如果遇到 `httpx.ConnectError`，说明需要配置代理。
> 设置环境变量 `HTTPS_PROXY` / `HTTP_PROXY` 指向可用的代理服务即可。

**其他同步方式：**

```bash
# 将整个目录的文件同步到 Notebook（增量同步：仅上传修改过的文件）
cd ~/.claude/skills/notebooklm && python3 scripts/run.py sync_sources.py \
  --notebook-id <notebook_id> \
  --dir "/path/to/workspace" \
  --extensions md,txt,pdf,docx

# 预览将同步的文件（不实际上传）
cd ~/.claude/skills/notebooklm && python3 scripts/run.py sync_sources.py \
  --notebook-id <notebook_id> \
  --dir "./docs" \
  --dry-run

# 查看 Notebook 中已有的 Source
cd ~/.claude/skills/notebooklm && python3 scripts/run.py sync_sources.py \
  --notebook-id <notebook_id> \
  --list-sources
```

### Step 6：提问

```bash
# 向当前激活的 Notebook 提问
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli ask "你的问题"

# 指定 Notebook 提问
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli ask "你的问题" --notebook <notebook_id>
```

### Step 7：生成内容（可选）

```bash
# 生成 Audio Overview（播客）
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli generate audio "make it engaging" --wait

# 生成视频
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli generate video --style whiteboard --wait

# 生成 Quiz
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli generate quiz --difficulty hard

# 生成幻灯片
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli generate slide-deck

# 下载生成的内容
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli download audio ./podcast.mp3
cd ~/.claude/skills/notebooklm && python3 scripts/run.py --cli download quiz --format markdown ./quiz.md
```

---

## 追问机制（关键！）

每次查询后必须：
1. **暂停** — 不要立即回复用户
2. **比对** — 将回答与用户原始请求对比
3. **识别缺口** — 判断是否还需要更多信息
4. **追问** — 如有缺口，立即发起追问
5. **综合** — 合并所有回答后再回复用户

---

## 典型使用场景

### 场景 A：查询已有 Notebook

```
1. notebooklm list → 找到目标 Notebook
2. notebooklm use <id> → 激活
3. notebooklm ask "..." → 提问
```

### 场景 B：从本地项目创建知识库

```
1. notebooklm create "项目名" → 创建 Notebook
2. notebooklm use <id> → 激活
3. sync_sources.py --notebook-id <id> --dir ./project → 批量上传
4. notebooklm ask "..." → 提问
```

### 场景 C：添加单个文件并提问

```
1. notebooklm use <id> → 激活已有 Notebook
2. notebooklm source add ./new-paper.pdf → 添加文件
3. notebooklm ask "summarize the new paper" → 提问
```

---

## 排错

| 问题 | 解决方案 |
|------|----------|
| Cookie 过期 | `--cli login` 重新认证 |
| `notebooklm` 命令找不到 | 删除 `.venv` 后重新运行，自动重建 |
| 首次运行很慢 | 正常——需要创建 venv 并安装 Chromium |
| `auth check --test` 失败 | `--cli login` 重新登录 |
| Source 上传失败 | 检查文件格式是否支持（pdf/md/txt/docx） |
| Rate limit | Google 限制，等待后重试 |
| 批量上传全部失败 | 可能已到 50 source 上限，用 `source list` 检查 |
| 上传报错但 source 出现重复 | API 幻影错误，用 `dedup_sources.py` 去重 |
| `run.py` 调用脚本卡住 | `batch_upload.py` 用 `.venv/bin/python3` 直接运行 |
| Source 状态卡在 ERROR | 用 `repair_sources.py` 自动删除→重传 |
| Source 状态卡在 PREPARING | 等几分钟；若超过 10 分钟仍未就绪，用 `repair_sources.py --include-preparing` |
| httpx.ConnectError | 需要代理访问 Google，设置 `HTTPS_PROXY` / `HTTP_PROXY` 环境变量 |
| 代理间歇性超时 | 代理服务不稳定，重试即可；脚本内部不做自动重试避免重复上传 |

## 限制

- **每个 Notebook 最多 50 个 Source**（硬限制，无法绕过）
- 使用非官方 Google RPC API，可能因 Google 更新而失效
- Cookie 有效期有限，可能需要定期重新登录
- API 错误不可靠：报错不代表失败，成功也不代表没问题
- 适用于个人研究和原型开发

---
> Source: [JoenHune/notebooklm-agent-skill](https://github.com/JoenHune/notebooklm-agent-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
