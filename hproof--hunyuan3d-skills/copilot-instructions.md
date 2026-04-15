## hunyuan3d-skills

> 这是一个腾讯混元3D大模型的使用环境，通过 Python SDK 提供以下功能：

## 项目介绍

这是一个腾讯混元3D大模型的使用环境，通过 Python SDK 提供以下功能：

- 提交混元3D生成任务
- 查询任务进度
- 下载生成结果（GLB/OBJ 格式）

### 目录结构

- `venv/` - Python 虚拟环境目录
- `downloads/` - 下载的 3D 模型文件（GLB/OBJ/FBX）
- `.env` - 腾讯云凭证配置
- `.claude/skills/` - Agent skills
  - `texto3d_submit/` - 提交混元3D生成任务
  - `texto3d_query/` - 查询任务进度和结果

### .env 文件配置

在项目根目录创建 `.env` 文件：

```
TENCENT_SECRET_ID=你的SecretId
TENCENT_SECRET_KEY=你的SecretKey
TENCENT_REGION=ap-guangzhou
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hproof)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hproof)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
