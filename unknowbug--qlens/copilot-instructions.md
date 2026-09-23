## qlens

> > 本仓库引入 **Anchorlaw 协议**（[E:\PYTHON\Anchorlaw](E:\PYTHON\Anchorlaw)）——"任何声称都必须有可验证的实践锚点"。

# QLens AGENTS.md（项目级常驻指令）

> 本仓库引入 **Anchorlaw 协议**（[E:\PYTHON\Anchorlaw](E:\PYTHON\Anchorlaw)）——"任何声称都必须有可验证的实践锚点"。
> 本文件是**索引**——协议知识按需加载（Anchorlaw v0.18 §14 Skill Manifest），铁律正文在对应 skill/文档，不在本文件常驻。

## 〇、开始工作前（每个 session 必做）

1. `git status` 确认工作区状态（远程 = `github.com/unknowbug/qlens`，push 需代理 `127.0.0.1:9199`，commit 必须 `--author="unknowbug <unknowbug@users.noreply.github.com>"`）
2. 若改动了 src/ 下的公开函数：新增/变更函数必须带 anchor 标注（`// @anchor.test` 或 `// @anchor.idk`），新增界面文本必须走 `T()`/`I18n::Get()` + 补 .po
3. 涉及核心逻辑改动后：跑 `python -m anchorlaw_scanner check src` 确认不新增 ERROR 级模式

## 一、Anchorlaw 协议索引（权威正文在 Anchorlaw 仓库）

| 主题 | 位置 |
|------|------|
| 协议规范 v0.18（语言无关） | `E:\PYTHON\Anchorlaw\spec\protocol-v0.18.md` |
| 扫描器（P1-P6 + severity） | `E:\PYTHON\Anchorlaw\python\anchorlaw-scanner\`（已装，`python -m anchorlaw_scanner check <dir>`；v0.16 起 `--lang {python,cpp,go,java}` 原生支持 comment-form 语言的 anchor 提取） |
| Skill 参考实现 | `E:\PYTHON\Anchorlaw\dsh\skills\anchor-*\`（`.reasonix\skills\` 已归档） |
| 本仓库验证协议（构建型适配） | [docs/08-anchorlaw.md](docs/08-anchorlaw.md) |

## 二、本仓库验证规则（速查，正文见 docs/08-anchorlaw.md）

**C++ 标注语法**（行注释，inert——删除协议零代码改动）：

```cpp
// @anchor.test("TagStore 组合查询 AND 语义", source="probe:tag_store_probe#001")
QStringList comboSearch(...);

// @anchor.idk("超大目录（>10 万图）扫描的内存边界未验证", source="static: 未在测试中覆盖大输入")
void loadFolder(...);
```

**门禁**（v0.18 §15.4 判据先行）：
- 公开函数必须有 `@anchor.test` 或 `@anchor.idk`（纯逻辑 = WARNING 级；I/O 重 = INFO 级；`_` 前缀/测试函数 = SKIP）
- `@anchor.test` 必须带 `source` 字段（验证载体引用：probe/测试/观察记录）
- `@anchor.idk` 描述具体未知边界——TODO ≠ idk（"我知道该做但没做" ≠ "我不知道正确行为"）
- ERROR 级模式（吞噬异常等）不允许新增；存量逐步清零

## 三、与其他铁律的关系

- **i18n 铁律**：界面文本必须走 `T()`/`I18n::Get()`——与 anchor 无关，各自独立遵守
- **崩溃日志铁律**：交付程序必须带全局崩溃捕获（crashlog 模块）——这是运行期证据，与 anchor 的验证载体互补
- **不放弃原则**：QLens 不豁免——任务持续推进直到用户命令停止

## 四、验证命令

```powershell
# 扫描防御性模式（基线：36 个——0 ERROR + 18 WARN + 18 INFO，集中在 src/mcp）
python -m anchorlaw_scanner check src

# comment-form 语言 anchor 标注提取/校验（v0.16 起原生支持，取代本地脚本）
python -m anchorlaw_scanner check src --lang cpp
```

---
> Source: [unknowbug/qlens](https://github.com/unknowbug/qlens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
