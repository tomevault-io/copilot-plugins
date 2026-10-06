## kelip-paper-reading

> 这个目录是一套**带人精读论文**的流程和脚本：先把论文材料拉到本地（PDF、LaTeX 源码、裁好的图、官方代码），

# 给 AI Agent 的说明

这个目录是一套**带人精读论文**的流程和脚本：先把论文材料拉到本地（PDF、LaTeX 源码、裁好的图、官方代码），
再按六站一站一站讲——动机 → 架构 → 训练 → 数据 → 输入输出信息流 → 拿一个例子对着代码走一遍。

有人让你读 / 精读 / 讲解 / 拆解一篇论文时：

1. **先读 `SKILL.md`**，按它的流程来，不要直接给一段摘要。
2. 第 0 步用 `scripts/fetch_paper.py <arXiv 链接 / PDF / 目录>` 拉材料，看一眼 `figures/_contact.png` 检查裁图。
3. 开讲前自己先读懂：读 `outline.md`、`paper.tex`，亲眼看关键的图；第一次讲之前读一遍 `references/examples.md`。
4. **讲到哪张图就把那张图显示给用户**，先说「这张图是什么」再讲；一次只讲一站，讲完问要不要继续。
5. 公式写成 Unicode + 符号表 + 一组代入的数字，不要写裸 LaTeX。
6. 有 `profile.md` 就按它调整讲法（熟的一句带过、不熟的先铺垫）。

依赖：Python 3 + PyMuPDF（`pip install pymupdf`）。代码只读，不替用户跑仓库脚本或装依赖。

---
> Source: [skJack/kelip-paper-reading](https://github.com/skJack/kelip-paper-reading) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
