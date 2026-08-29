## shulihuazixuecongshu

> - `books/` 下的 17 个“`书名 - 数理化自学丛书编委会.md`”文件是规范正文源；根目录不放置单册电子书文件。

# 仓库维护约定

- `books/` 下的 17 个“`书名 - 数理化自学丛书编委会.md`”文件是规范正文源；根目录不放置单册电子书文件。
- 图片统一存放于 `books/assets/`；移动或重命名资源时必须同步更新全部 Markdown 引用。
- 书名是每册唯一 H1；章节从 H2 开始。
- 不凭空补写原扫描缺失内容；证据不足时保留透明校注。
- 公式内容不得经过会破坏 LaTeX 的全角标点替换。
- 批量修改前先在仓库外建立备份；不得执行破坏工作区状态的 Git 操作。
- 提交前运行 `make audit` 和 `make privacy`；推送前运行 `make pre-push`。
- 修改构建链后运行 `make all` 和 `make verify`。
- `dist/`、`.build/` 和动态 JSON 报告属于生成物，不纳入版本控制。

---
> Source: [tradecatlabs/shulihuazixuecongshu](https://github.com/tradecatlabs/shulihuazixuecongshu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
