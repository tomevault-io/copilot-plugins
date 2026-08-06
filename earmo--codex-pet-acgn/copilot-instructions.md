## codex-pet-acgn

> 本项目是面向 Codex Desktop 的 ACGN 主题自定义宠物合集。宠物运行时文件遵循 Codex v2 精灵图规范，安装目标为个人 Codex 目录下的 `~/.codex/pets/<pet-id>`。

# Codex Pet ACGN 项目协作说明

## 项目定位

本项目是面向 Codex Desktop 的 ACGN 主题自定义宠物合集。宠物运行时文件遵循 Codex v2 精灵图规范，安装目标为个人 Codex 目录下的 `~/.codex/pets/<pet-id>`。

## 目录约定

- `pets/<pet-id>/pet.json`：运行时元数据，必须包含正确的 `id` 和 `spriteVersionNumber`。
- `pets/<pet-id>/spritesheet.webp`：透明精灵图；v2 图集为 `1536 × 2288`，8 列 × 11 行。
- `pets/<pet-id>/preview.gif`、`preview.png`：README 和人工检查使用的预览文件。
- `pets/<pet-id>/checksums.sha256`：远程安装器下载校验清单，只校验 `pet.json` 和 `spritesheet.webp`。
- `pets.json`：远程安装器使用的宠物目录清单；新增或删除宠物时同步更新。
- `scripts/install-pet.ps1`、`scripts/install-pet.sh`：跨平台远程安装器。
- `pets/*/work/`：制作和 QA 中间产物，已被 Git 忽略，不应提交。

## 修改与新增宠物

1. 新宠物目录名使用小写 kebab-case，并与 `pet.json` 的 `id` 保持一致。
2. 保留 `pet.json`、`spritesheet.webp` 和必要的预览文件；不要提交提示词、解码帧、临时渲染结果或完整 QA 工作目录。
3. 更新根目录 `pets.json`。
4. 任何运行文件变更后重新生成对应的 `checksums.sha256`。
5. 同步更新 `README.md` 的宠物介绍、版权说明和安装示例。
6. 新增角色或使用第三方素材时，先确认许可范围，并在 README 或宠物元数据中保留必要的版权说明。

## 安装器约定

- 默认远程源为 GitHub `main` 分支的 Raw 地址；可用 `CODEX_PET_RAW_BASE` 覆盖测试源。
- 最终只安装 `pet.json` 和 `spritesheet.webp`，校验清单不复制到用户目录。
- PowerShell 和 Bash 安装器必须保持参数语义一致：宠物 ID、列出宠物、覆盖 `CODEX_HOME`。
- 下载后先完成文件校验和 `pet.json.id` 校验，再写入目标目录。
- 不要在安装器中加入遥测、联网统计或额外文件写入，除非得到明确需求。

## 验证

提交前至少运行：

```powershell
# PowerShell 语法检查
$tokens = $null
$errors = $null
[System.Management.Automation.Language.Parser]::ParseFile(
  (Resolve-Path scripts/install-pet.ps1),
  [ref]$tokens,
  [ref]$errors
) | Out-Null
if ($errors.Count -gt 0) { $errors | Format-List *; exit 1 }

# 清单与校验文件检查
Get-Content -Raw pets.json | ConvertFrom-Json | Out-Null
Get-ChildItem pets -Directory | ForEach-Object {
  Push-Location $_.FullName
  try { sha256sum -c checksums.sha256 }
  finally { Pop-Location }
}

# Bash 语法检查（在 Git Bash、WSL 或 macOS/Linux 中运行）
bash -n scripts/install-pet.sh
```

如果修改了安装器，最好使用本地静态 HTTP 服务实际测试 `-RawBase` / `CODEX_PET_RAW_BASE`，并确认目标目录只出现 `pet.json` 与 `spritesheet.webp`。

## 提交边界

- 不要提交 `.idea/`、`pets/*/work/`、临时安装目录或生成缓存。
- 不要为了新增宠物大范围重排 README 或修改无关文件。
- 当前项目尚未发布 npm 包；除非明确提出，否则不要把 `package.json`、npm 发布流程或 npx 命令当作已存在能力。

---
> Source: [Earmo/codex-pet-acgn](https://github.com/Earmo/codex-pet-acgn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
