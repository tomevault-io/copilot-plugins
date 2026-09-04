## openclaw-pdf-to-cad-agent-skill

> - 代码、路径、命令、API、字段名、配置名保持英文。

# AGENTS

## 默认沟通

- 默认使用中文总结工作结果。
- 代码、路径、命令、API、字段名、配置名保持英文。
- 每次修改后说明 `changed files`、`implementation`、`verification`、`next steps`。

## 开始修改前

- 先读取 `README.md`、`skills/openclaw-pdf-to-cad/SKILL.md` 和相关 `docs/`。
- 先确认本仓库是 Agent Skill 项目，不要把它改成传统网站、Next.js 项目或 FastAPI 平台仓库。
- 如果当前目录不是 Git 仓库，必须先提醒用户，并且不得声称已经完成 `git diff`。
- 如果没有 Git 仓库，大任务修改前必须创建文件级备份。
- 不要大规模重写核心转换引擎，除非用户明确要求。
- 大任务前必须确认是否处于 Git 仓库；如果不是 Git 仓库，只能做文件级备份和本地验证，不能声称具备版本回滚。

## 商业与能力边界

- 不要过度承诺图纸转换能力。
- 对外表达应为：优先支持 PDF 工程图转 DXF / DWG 交付包。
- 扫描图、JPG、PNG、模糊图纸、缺失尺寸或 OCR 结果必须标记 `needs_review` 并要求人工复核。
- 不要承诺 JPG / PNG / 扫描 PDF 一键精准转 CAD。
- 不要承诺制造级完全自动重建。
- 不要说本工具可以替代工程师最终判断。
- 不要猜测缺失尺寸、标注、材料、比例、标题栏或工艺信息。
- 商业化文档必须明确区分“已实现 / 规划 / TODO”，不能把 API、收费、授权、批量平台、私有化部署或社群功能写成已完成。
- 避免在多个文档重复堆叠同一大段边界说明；优先引用 `docs/business/commercialization-model.md#commercial-boundary-checklist`。

## 数据与敏感信息

- 不要引入真实客户图纸、供应商图纸、企业样本或真实交付包。
- 不要提交 API key、license key、token、密码、SSH key、私有桥接配置或本机敏感路径。
- 普通客户图纸默认只用于本次转换。
- 授权脱敏后才可用于内部优化。
- 企业客户图纸默认完全隔离，不进入公共训练池，不用于其他客户优化。

## Git 安全

- 禁止 `git push`。
- 禁止 `git reset --hard`。
- 禁止 force push。
- 禁止删除分支。
- 禁止改写历史。
- 不要擅自更改 `LICENSE`；如发现许可证不利于商业化，只在文档中提出建议。

## 验证

- 优先运行可用测试，例如 `pytest -q`。
- 商业化文档修改后必须运行 `python3 scripts/validate_commercial_assets.py`。
- 如果无法运行测试，至少检查 Markdown 链接、目录结构和 JSON schema 有效性。
- 修改文档和模板时，确认没有新增真实客户数据或敏感文件。

---
> Source: [CavanJB/openclaw-pdf-to-cad-agent-skill](https://github.com/CavanJB/openclaw-pdf-to-cad-agent-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
