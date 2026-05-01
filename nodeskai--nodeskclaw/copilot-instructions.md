## gene-skill-sync

> 工作区/Agent 行为变更时，检查 Gene/Skill 是否需要同步更新并推送到 GeneHub


# Gene / Skill 同步规则

## 触发条件

以下变更完成后，必须评估是否需要更新 Gene 或 Skill：

- 工作区功能变更（黑板、任务、拓扑、消息、协作流程等）
- Agent 行为变更（system prompt、collaboration、工具调用方式等）
- Channel Plugin 变更（新增/修改 tool、新增事件类型等）
- 后端 API 变更（影响 Agent 可调用的端点）

## 检查流程

### 1. 判断是否涉及 Gene/Skill

对照变更内容，检查 `nodeskclaw-backend/app/data/gene_templates/` 中的 JSON 模板：

- `manifest.skill.content`（SKILL.md 内容）是否需要更新
- `manifest.tool_allow`（工具白名单）是否需要调整
- `manifest.mcp_servers`（MCP 配置）是否需要变更
- `manifest.openclaw_config`（OpenClaw 配置合并项）是否受影响

### 2. 更新 Gene 模板

修改对应 JSON 模板文件，同步更新 `version` 字段。

### 3. 推送到 GeneHub

模板更新后，提醒用户执行推送：

```bash
export GENEHUB_REGISTRY_URL=<GeneHub 地址>
export GENEHUB_API_KEY=<API Key>
python scripts/upload_seeds_to_genehub.py --dry-run  # 先预览
python scripts/upload_seeds_to_genehub.py             # 确认后推送
```

### 4. 同步已安装实例

已通过 `InstanceGene` 安装了该 Gene 的 AI 员工，其 Pod 上的 `.openclaw/skills/{skill_name}/SKILL.md` 需要更新。提醒用户：

- 重新部署相关实例，或
- 通过后端 `gene_service.install_gene()` 触发重新安装

## 禁止

- **禁止改了 Agent 行为不检查 Gene 模板**
- **禁止更新了 Gene 模板不提醒推送 GeneHub**
- **禁止忽略已安装实例的同步**

---
> Source: [NoDeskAI/nodeskclaw](https://github.com/NoDeskAI/nodeskclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
