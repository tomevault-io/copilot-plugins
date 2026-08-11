## open-nuyoah-skill

> - 本仓库只保存可公开、可复用、不依赖南鸢个人资产的 Agent Skills。

# open-nuyoah-skill 仓库规则

## 定位

- 本仓库只保存可公开、可复用、不依赖南鸢个人资产的 Agent Skills。
- 公共 Skill 必须让使用者建立、拥有并维护自己的项目资产。
- 本仓库不接收南鸢个人角色图、情绪图、完整个人角色 Prompt、Eagle ID、本机绝对路径、确认记录或私有测试素材。

## 目录

- `skills/`：正式公开 Skill，每个 Skill 一个英文 kebab-case 目录。
- `docs/`：仅保存仓库级维护或发布说明。
- 根目录：只放 `AGENTS.md`、`README.md`、`LICENSE`、`.gitignore` 等仓库入口文件。

## Skill 约束

- 每个 Skill 的 `SKILL.md` 只保留启动时必须知道的流程。
- 详细方法进入 `references/`；重复且确定的逻辑进入 `scripts/`。
- 不在已安装 Skill 目录中保存用户生成资产；项目资产默认写入当前项目的隐藏目录。
- 没有真实图片或工具输出时，不得伪造资产登记、视觉检查或成功状态。
- 候选角色只有在用户明确确认后才能登记为 `adopted`。
- 公共版本不得提供会让所有用户静默复用同一个作者角色的默认角色包。

## 修改与验证

- 修改行为前记录基线和保护范围；未触及的规则保持不变。
- 新增或修改脚本后必须真实运行。
- 每个正式 Skill 至少通过结构校验、合同测试、正例/反例/近邻触发检查和一个可复现案例。
- 发布前执行隐私泄漏检查，至少扫描 `nuyoah`、`南鸢`、`Eagle`、`/Users/`、Token、密钥与常见凭证格式。
- 真实生图一致性只有在调用图像模型并完成人工视觉检查后才能声称通过。

## GitHub

- `main` 是当前公开可用版本。
- 只提交本轮相关文件，不使用无范围确认的 `git add .` 或 `git add -A`。
- 推送前检查仓库可见性、LICENSE、README、图片资源路径和敏感信息。
- 不创建 Release、协作者或自动化发布，除非用户另行明确要求。

---
> Source: [nuyoah-ai-works/open-nuyoah-skill](https://github.com/nuyoah-ai-works/open-nuyoah-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
