## mineru-skill

> |


# MinerU 文档处理技能

通过 MinerU API 将 PDF/DOCX/PPTX/图片/HTML 转换为 Markdown + 图片。

## 目录约定

- skill 根目录: 当前 `SKILL.md` 所在目录
- 脚本目录: `<skill-root>/scripts`
- Python 虚拟环境: `<skill-root>/scripts/.venv`
- 默认数据目录: `~/.mineru`
- 可选环境变量: `MINERU_DATA_DIR`，可覆盖默认数据目录

如果你是在仓库根目录直接使用本 skill，下面命令可以直接运行：

```bash
python scripts/setup_env.py
```

环境初始化完成后，Python 解释器位于：

```text
Windows: scripts\.venv\Scripts\python.exe
Unix:    scripts/.venv/bin/python3
```

以下用 `$PY` 代指该解释器，`$SD` 代指脚本目录。

## 环境准备

首次使用需完成以下步骤：

```bash
# 1. 初始化环境（创建 venv、安装依赖、安装 Playwright 浏览器）
python $SD/setup_env.py

# 2. 编辑账户配置（填入 MinerU 账号密码）
#    默认文件位置: ~/.mineru/accounts.yaml
#    如果设置了 MINERU_DATA_DIR，则位于 $MINERU_DATA_DIR/accounts.yaml

# 3. 批量登录获取 Token
$PY $SD/batch_login.py
```

## 核心工作流

**处理文档前必须先检查 Token 有效性。** 完整流程:

1. 运行 `check_tokens.py`，根据退出码判断:
   - 退出码 0 → Token 有效，继续处理
   - 退出码 1 → Token 过期，先运行 `batch_login.py` 续期
2. Token 有效后，运行 `process_document.py` 或 `process_batch.py` 处理文档

## 各脚本用法

### check_tokens.py — 检查 Token 状态

```bash
$PY $SD/check_tokens.py
```

退出码: 0=全部有效，1=有过期或缺失。

### batch_login.py — Token 续期

```bash
# headless 模式（默认，无需显示器）
$PY $SD/batch_login.py

# 调试模式（打开浏览器界面）
$PY $SD/batch_login.py --headed
```

自动完成: 登录 MinerU → 删除旧 Token → 创建新 Token → 保存到 `~/.mineru/all_tokens.json`。

### process_document.py — 单文档处理

```bash
# 处理本地文件（输出到源文件同目录）
$PY $SD/process_document.py <文件路径>

# 指定输出目录
$PY $SD/process_document.py <文件路径> --output-dir <目录>

# 指定模型
$PY $SD/process_document.py <文件路径> --model vlm

# 处理 URL
$PY $SD/process_document.py https://example.com/doc.pdf
```

支持格式: PDF, DOC, DOCX, PPT, PPTX, PNG, JPG, JPEG, HTML。
输出: `{文件名}.md` + `{文件名}_images/` 目录。

### process_batch.py — 批量处理

```bash
# 处理目录下所有 PDF
$PY $SD/process_batch.py <目录>

# 指定文件类型
$PY $SD/process_batch.py <目录> --pattern "*.docx"

# 控制并发数
$PY $SD/process_batch.py <目录> --max-workers 3

# 递归扫描子目录
$PY $SD/process_batch.py <目录> --recursive
```

## 常见场景

| 用户请求 | 操作 |
|----------|------|
| "帮我处理这个 PDF" | check_tokens → process_document.py |
| "转换这个目录下所有文档" | check_tokens → process_batch.py |
| "查看 Token 状态" | check_tokens.py |
| "续期/刷新 Token" | batch_login.py |
| "处理这个网页/URL" | check_tokens → process_document.py URL |

## 注意事项

- 文件大小限制 200MB，超过会报错
- 模型自动选择: PDF/DOC/PPT/图片 → vlm，HTML → MinerU-HTML
- Token 有效期约 90 天，过期需运行 batch_login.py 续期
- 数据目录默认是 `~/.mineru/`，可通过 `MINERU_DATA_DIR` 自定义
- 详细 API 参数和错误排查参考 `references/api-reference.md`

---
> Source: [Lucent-Snow/mineru-skill](https://github.com/Lucent-Snow/mineru-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
