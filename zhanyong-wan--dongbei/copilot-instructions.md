## dongbei

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目简介

dongbei 是一门以东北方言词汇为关键字的编程语言，基于 Python 3 实现。源文件扩展名为 `.dongbei`，解释器将其转译成 Python 后执行。

## 常用命令

```bash
# 安装依赖
pip install -r requirements.txt

# 跑某个 .dongbei 源文件
src/dongbei.py <文件>.dongbei

# 跑内置案例（demo/ 目录下的）
src/dongbei.py --bihua 磨叽

# 打印对应生成的 Python 代码（调试用）
src/dongbei.py --xudao <文件>.dongbei

# 启动交互式 REPL
src/dongbei.py

# 跑全部测试（单元测试 + 所有 demo）
bash test/test_all

# 只跑单元测试（所有 *_test.py）
python3 -m unittest discover -s test -p "*_test.py"

# 只跑某个测试文件
python3 -m unittest test.dongbei_test
python3 -m unittest test.dongbei_add_test

# 跑单个测试类或测试方法
python3 -m unittest test.dongbei_test.DongbeiParseExprTest
python3 -m unittest test.dongbei_test.DongbeiParseExprTest.test_parse_number
```

## 代码架构

整个解释器只有一个源文件：`src/dongbei.py`（约 2100 行）。

### 执行流水线

```
.dongbei 源码
  → DongbeiParser.tokenize()       # 词法分析 → Token 列表
  → DongbeiParser.translate_tokens_to_statements()  # 语法分析 → Statement AST
  → translate_statement_to_python()   # 代码生成 → Python 字符串
  → exec()                         # 执行生成的 Python
```

入口函数：`translate_and_run()`，对外暴露完整的"翻译+执行"流程。

### 关键数据结构

- **常量 `KW_*`**：所有东北话关键字的字符串常量（约 70 个）。`KEYWORDS` 元组的顺序决定词法分析匹配优先级，较长/较具体的关键字必须排在其前缀之前（注释中有说明）。
- **`Token`**：词法单元，含 `kind`（`TK_*` 常量）、`value`、`loc`（源码位置）。
- **`Expr` 及其子类**：表达式 AST 节点，每个子类实现 `to_python()` 和 `to_dongbei()`。
- **`Statement`**：语句节点，`kind` 为 `STMT_*` 常量，`value` 存放操作数。
- **`DongbeiParser`**：包含 `tokenize()` 和 `translate_tokens_to_statements()` 两个核心方法。

### 代码生成

`translate_statement_to_python(stmt, indent)` 是一个大型分派函数，根据 `stmt.kind` 生成对应 Python 代码。每个 `Expr` 子类通过 `to_python()` 自行生成表达式代码。

生成的 Python 代码通过 `exec(py_code, globals(), globals())` 执行——全局/局部命名空间共用同一个字典，以支持递归函数定义。

### 自举文件

`src/dongbei.dongbei` 是用东北话写的东北话解释器（套娃）。

## 开发规范

一定要注意幽默感！注释、文档和PR说明要用东北方言，保证读者和用户会心一笑。

每个 PR **必须** 同时包含：
1. 文档更新（`README.md` 和 `doc/cheatsheet.md`）
2. 对应的测试案例（`test/dongbei_test.py`）

新加测试案例时，**每个测试方法只整一件事**——别把好几件不同的事塞一个方法里。一个萝卜一个坑，清清楚楚，省得将来查问题抓瞎。

所有函数名（包括方法名）**必须**用 snake_case 命名，e.g. `translate_and_run()`，不用 PascalCase 或 camelCase。

## 发布流程

1. 更新 `setup.cfg` 中的 `version`
2. 更新 `src/dongbei.py` 中的 `DONGBEI_VERSION`
3. 在 `CHANGES.md` 记录变更
4. `git tag -a x.y.z -m "发布 x.y.z"` 并 `git push --tags origin master`（CD 自动发布到 PyPI）

---
> Source: [zhanyong-wan/dongbei](https://github.com/zhanyong-wan/dongbei) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
