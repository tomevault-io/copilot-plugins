## utilitylab

> 本文件供编码代理阅读。完整规则见 [RULES.md](RULES.md)，此处只摘录硬性约束。

# CLAUDE.md

本文件供编码代理阅读。完整规则见 [RULES.md](RULES.md)，此处只摘录硬性约束。

## 硬性约束

1. **不要提交凭据。** 不把 API Key、令牌、密码、私钥、证书或真实配置写进
   源码、文档、测试数据或注释。需要示例值时使用脱敏模板
   （`.env.example` / `.env.sample` / `.env.template` / `.env.dist`）。

2. **不要提交数字创意资产、归档和编译产物。** 包括模型、动画、贴图、音视频、
   字体、`.zip`/`.7z` 等归档，以及 `.exe`/`.dll`/`.wasm` 等二进制。

3. **单文件 50 MB 上限不可豁免。** 没有任何绕过方式，不要尝试绕过。

4. **不要动 `AnimotionHub/`。** 该目录约 38 GB 数字创意资产，被 `.gitignore`
   整体排除。不要把它加入 Git，不要在其中生成新文件，也不要为了"让检查通过"
   而修改排除规则。

5. **项目目录边界。** 每个实验工具一个独立目录；跨项目共享代码放
   `shared/<库名>/`。不要把源码、生成物或迁移中间文件散落在仓库根目录。
   新增根目录文件需同步更新 `RULES.md` 第 3 节与 `README.md` 的目录约定。

## 提交前

运行策略检查（已安装 hook 时 `git commit` 会自动执行）：

```bash
powershell -NoProfile -ExecutionPolicy Bypass -File ./scripts/git/check-repository-policy.ps1
```

检查失败时**不要**用 `--no-verify` 绕过，也不要为了通过检查而放宽
`.gitignore` 或策略脚本。正确做法是移除违规内容；确属必要的例外，在
`.policy-allow` 中登记并写明来源、许可证与必要性（`size` 规则除外，它不可豁免）。

提交信息格式：`<类型>(<范围>): <祈使句摘要>`，类型取 `feat`/`fix`/`docs`/
`refactor`/`test`/`chore`/`build`，范围通常是项目目录名，仓库级改动用 `repo`。

## 修改策略工具时

- `.gitignore` 与 `scripts/git/check-repository-policy.ps1` 中的扩展名清单
  必须同步修改。
- `scripts/git/*.ps1` 必须保存为 **UTF-8 with BOM**（Windows PowerShell 5.1
  会把无 BOM 的 `.ps1` 按 ANSI 代码页解码，损坏中文字面量）。
- 策略脚本需兼容 Windows PowerShell 5.1：不可使用 `$IsLinux`/`$IsMacOS`
  自动变量、`` `u{} `` 转义，也不要对原生命令用 `2>&1`
  （在 `$ErrorActionPreference='Stop'` 下会抛 `NativeCommandError`）。
- `scripts/git/hooks/` 下的文件必须是 LF 行尾。

---
> Source: [Ogurijay/UtilityLab](https://github.com/Ogurijay/UtilityLab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
