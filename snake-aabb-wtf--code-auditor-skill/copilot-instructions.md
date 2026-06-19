## code-auditor-skill

> Automated code audit using static syntax checking tools. Invoke when user requests code security audit, code quality review, static analysis, syntax checking, or compliance verification for any codebase or project.


# 代码审计技能 (Code Auditor)

教 LLM 真正理解和运用静态语法纠错工具进行代码审计，而不是只会调用预设脚本。

## 核心思想

```
┌─────────────────────────────────────────────────────────────┐
│  真正懂审计 = 理解工具原理 + 灵活构建命令 + 智能解读输出  │
│  而不是：找到脚本 → 执行 → 粘贴结果                        │
└─────────────────────────────────────────────────────────────┘
```

---

## 工作流程总览

1. **发现目标** — 搜索并定位要审计的代码文件/目录
2. **克隆仓库** — 如果是 GitHub 仓库链接，先 `git clone` 到本地再审计
3. **语言检测** — 通过扩展名、配置文件识别编程语言
4. **询问审计策略** — 主动问用户要什么深度（语法检查 / 标准审计 / 深度安全），不要默认假设
5. **工具认知** — 根据选定的策略深度，选择对应类型的工具（编译器/linter/格式化器/安全扫描）
6. **构建命令** — 根据项目结构、审计策略和环境动态构造审计命令
7. **执行审计** — 依次运行各工具的审计命令，捕获 stdout/stderr
8. **解读结果** — 智能解读工具输出，区分致命错误、潜在问题和风格建议
9. **按需读码** — 仅当工具报告语法/编译错误时，读取报错文件的源码进行复核
10. **报告呈现** — 生成结构化审计报告

---

## 步骤一：发现目标文件

根据用户描述或项目上下文，自动搜索需要审计的文件：

- 如果用户指定了文件或目录路径，直接使用
- 如果用户没有指定，根据上下文推测目标
- 使用 `Glob` 或 `LS` 等工具搜索项目文件

通过项目根目录的配置文件推断语言生态：

| 配置文件 | 推断语言 | 典型项目结构 |
|---------|---------|-------------|
| `CMakeLists.txt` / `Makefile` | C/C++ | `src/`, `include/`, `lib/` |
| `pom.xml` / `build.gradle` | Java | `src/main/java/`, `src/test/java/` |
| `Cargo.toml` | Rust | `src/main.rs`, `src/lib.rs` |
| `go.mod` | Go | 平铺或多级 package |
| `package.json` | JS/TS | `src/`, `node_modules/` |
| `pyproject.toml` / `requirements.txt` | Python | `src/`, `tests/` |
| `*.sln` / `*.csproj` | C# | 多项目 Solution |
| `mix.exs` | Elixir | `lib/`, `test/` |
| `pubspec.yaml` | Dart/Flutter | `lib/`, `test/` |
| `Package.swift` | Swift | `Sources/`, `Tests/` |

> **关键理解**：配置文件不仅告诉你用什么语言，还告诉你项目的**模块结构和构建方式**，这直接决定了你应该用什么范围的命令（单文件检查 vs 全项目检查）。

---

## 步骤二：语言检测规则

### 文件扩展名识别

```
.c/.h/.cpp/.hpp/.cc/.cxx → C/C++
.java                       → Java
.kt/.kts                    → Kotlin
.cs                         → C#
.py/.pyw                    → Python
.js/.jsx/.mjs/.cjs          → JavaScript
.ts/.tsx                    → TypeScript
.go                         → Go
.rs                         → Rust
.rb                         → Ruby
.php/.phtml                 → PHP
.sh/.bash/.zsh              → Shell
.swift                      → Swift
.dart                       → Dart/Flutter
（以及其他常见扩展名）
```

### Shebang 识别（无扩展名文件）

```
#!/usr/bin/env python    → Python
#!/bin/bash              → Shell
#!/usr/bin/ruby          → Ruby
#!/usr/bin/perl          → Perl
#!/usr/bin/php           → PHP
```

---

## 步骤三：审计策略确认（问用户，别猜）

**核心原则**：不要默认假设用户想要多深的审计。每次接到审计任务，先问用户再动手。

LLM 在完成语言检测后，应主动向用户提出以下问题来确认审计策略：

### 标准提问模板

```markdown
已识别到项目语言为 **<语言>**，审计目标为 **<文件/目录路径>**。

请选择审计深度（三选一）：

❶ **快速语法检查** — 只检查语法错误，最快完成
   适用：提交前快速验证、CI 预检查、修改后自查
   工具：仅编译器语法检查（如 gcc -fsyntax-only / node --check）
   耗时：数秒

❷ **标准质量审计** — 语法 + 代码质量 + 风格
   适用：常规代码审查、MR/PR 审查、定期质量检查
   工具：编译器 ⚡ + Linter 🔍（如 cppcheck / eslint / ruff）
   耗时：数十秒到数分钟

❸ **深度安全审计** — 语法 + 质量 + 安全专项扫描
   适用：上线前安全检查、第三方代码审计、安全合规
   工具：编译器 ⚡ + Linter 🔍 + 安全专项 🛡️（如 shellcheck / bandit / cppcheck --enable=warning）
   耗时：数分钟到十分钟
```

> **LLM 注意**：你要根据项目的实际情况动态填充 `<语言>` 和 `<路径>`，让用户清楚你在审计什么。

### 用户的回答可能不精确

用户可能不会直接说"我选二"，而是说自然语言。LLM 应能理解意图并映射到策略：

| 用户说 | 映射到策略 | 理由 |
|-------|-----------|------|
| "就看看有没有语法错误" | ❶ 快速语法检查 | 用户明确只要语法 |
| "帮我 review 一下代码质量" | ❷ 标准质量审计 | review 通常含质量检查 |
| "马上要上线了帮我扫一遍" | ❸ 深度安全审计 | 上线前需要安全把关 |
| "随便看看" / "你看着办" | ❷ 标准质量审计 | 默认中等深度 |
| "检查有没有安全漏洞" | ❸ 深度安全审计 | 明确要安全扫描 |

### 三层策略模板详解

#### ❶ 快速语法检查

| 维度 | 说明 |
|------|------|
| **目的** | 最快速度验证代码是否能通过编译/解析 |
| **工具** | 仅用编译器语法检查命令，不用 linter |
| **检查内容** | 语法错误、缺失符号、类型不匹配 |
| **不检查** | 代码风格、潜在 bug、安全漏洞 |
| **耗时** | 数秒 |
| **适用场景** | 改了一行代码想确认没写错、CI 快速门禁 |

**典型命令组合**（以 Python 为例）：
```bash
python -m compileall -q file.py     # 仅语法检查
```
（不执行 pyflakes / ruff / flake8）

#### ❷ 标准质量审计

| 维度 | 说明 |
|------|------|
| **目的** | 平衡速度与覆盖面，常规代码审查 |
| **工具** | 编译器语法检查 + 主要 Linter |
| **检查内容** | 语法错误 + 潜在 bug + 反模式 + 代码风格 |
| **不检查** | 专项安全漏洞（如 SQL 注入、XSS） |
| **耗时** | 数十秒到数分钟 |
| **适用场景** | MR/PR 审查、定期代码巡检、日常质量把控 |

**典型命令组合**（以 Python 为例）：
```bash
python -m compileall -q .           # 语法检查
ruff check .                        # 全面质量检查（替代 pyflakes + flake8）
```
（不执行专项安全扫描）

#### ❸ 深度安全审计

| 维度 | 说明 |
|------|------|
| **目的** | 最全面的检查，不遗漏任何潜在风险 |
| **工具** | 编译器 + Linter + 安全专项工具 + 最严格参数 |
| **检查内容** | 语法错误 + 潜在 bug + 代码风格 + **安全漏洞（注入、XSS、权限、密钥泄露）** |
| **耗时** | 数分钟到十分钟 |
| **适用场景** | 上线前安全检查、第三方代码审计、安全合规审查、高危项目 |

**典型命令组合**（以 Python 为例）：
```bash
python -m compileall -q .           # 语法检查
ruff check .                        # 质量 + 安全规则（ruff 包含安全规则）
flake8 .                            # 风格补充
```
（对 Shell 脚本还会加 shellcheck，对 JS 加 eslint-plugin-security 等）

---

## 步骤四：工具理解（核心部分）

**这是整个技能的灵魂**：LLM 必须理解每类工具的**检查维度**、**适用场景**和**输出风格**，才能灵活运用而不是死记命令。

### 工具类型总览

所有静态审计工具按功能分为三大类：

| 类型 | 本质 | 检查什么 | 速度 | 误报率 |
|------|------|---------|------|-------|
| **编译器语法检查** | 模拟编译，只解析语法树 | 语法错误、类型不匹配、未定义引用 | 最快 | 极低（都是真错误） |
| **Linter** | 静态分析 AST/CFG | 潜在 bug、反模式、安全漏洞、代码坏味 | 中等 | 中等（部分规则较主观） |
| **格式化器检查** | 解析→格式化→对比 | 缩进、空格、换行、命名风格 | 最快 | 极低（风格约定而已） |

> **理解这一点很重要**：编译器的"错误"是**确定性的**——代码就是错的。Linter 的"警告"是**概率性的**——可能有问题，需要人工判断。格式化器的"问题"是**约定性的**——不影响逻辑，只影响可读性。

### 如何为每种语言选择工具

LLM 应该根据**步骤三确定的策略 + 以下三个维度**综合决策：

```
维度1：项目范围
  - 单个文件 → 文件级命令（如 gcc -fsyntax-only file.c）
  - 完整项目 → 项目级命令（如 cargo check --workspace）

维度2：策略深度（由步骤三的询问结果决定）
  - ❶ 快速语法检查 → 仅编译器语法检查
  - ❷ 标准质量审计 → 编译器 + Linter
  - ❸ 深度安全审计 → 编译器 + Linter + 安全专项

维度3：环境限制
  - Windows 环境 → 优先用跨平台工具（eslint / ruff / cppcheck）
  - Linux 环境 → 所有工具都可用
  - 容器环境 → 检查工具是否已预装
```

### 各语言的工具家族

#### C/C++ 工具家族
```markdown
工具       | 类型         | 检查维度                    | 典型命令
-----------|-------------|----------------------------|-----------------------
gcc -fsyntax-only | 编译器语法 | 语法错误、语义错误     | gcc -fsyntax-only file.c
clang -fsyntax-only | 编译器语法 | 语法错误、更友好的错误信息 | clang -fsyntax-only file.cpp
cppcheck   | Linter      | 内存泄漏、越界、空指针、未初始化变量 | cppcheck --enable=all src/
clang-tidy | Linter      | 代码坏味、性能问题、安全漏洞 | clang-tidy file.cpp --
sparse     | Linter      | 内核风格语义检查（Linux 专用） | sparse file.c
```

**LLM 应当理解**：
- `gcc -fsyntax-only` 只做语法解析，不生成机器码，所以最快也最安全
- `cppcheck --enable=all` 会启用所有检查器（性能、风格、可移植性、信息），如果你只关心安全问题，可以只 `--enable=warning`
- `clang-tidy` 的 `--` 表示后面不再有编译器选项，这是一个 Unix 惯例

#### Java 工具家族
```markdown
工具    | 类型    | 检查维度                    | 典型命令
--------|--------|----------------------------|-----------------------
javac -Xlint:all | 编译器语法 | 语法、泛型、未检查转换、fall-through | javac -Xlint:all -proc:none file.java
pmd     | Linter | 空 catch、过长方法、重复代码、导入问题 | pmd check -d src/ -R rulesets/java/quickstart.xml
```

**LLM 应当理解**：
- `-Xlint:all` 启用 javac 的所有编译警告，`-proc:none` 禁用注解处理以避免干扰
- PMD 的 `-R` 指定规则集，`quickstart.xml` 是入门级规则，还有 `unusedcode.xml`, `braces.xml` 等专项规则集

#### Python 工具家族
```markdown
工具          | 类型    | 检查维度                    | 典型命令
--------------|--------|----------------------------|-----------------------
python -m compileall | 编译器语法 | 语法错误、缩进错误     | python -m compileall -q .
pyflakes      | Linter | 未使用变量/导入、未定义变量、重复定义 | pyflakes .
ruff          | Linter | 600+ 规则，覆盖 pyflakes+flake8+isort | ruff check .
flake8        | Linter | PEP8 风格、逻辑错误、复杂度警告 | flake8 .
```

**LLM 应当理解**：
- `python -m compileall` 将 Python 源码编译为字节码，语法错误会在这里暴露，速度极快
- `ruff` 是 Rust 实现的 linter，速度比 flake8 快 10-100 倍，覆盖面也更广。如果项目里有 `pyproject.toml` 或 `.ruff.toml`，ruff 会自动读取配置
- `pyflakes` 只检查逻辑错误，不检查风格。`flake8` = pyflakes + pycodestyle + mccabe（复杂度）。`ruff` 可以替代以上全部

#### JavaScript 工具家族
```markdown
工具           | 类型    | 检查维度                    | 典型命令
---------------|--------|----------------------------|-----------------------
node --check   | 编译器语法 | 语法错误                 | node --check file.js
eslint         | Linter | 潜在 bug、反模式、风格一致性 | eslint '**/*.js'
jshint         | Linter | 语法警告、全局变量泄漏、未定义变量 | jshint .
quick-lint-js  | Linter | 超快速语法+逻辑检查        | quick-lint-js file.js
```

**LLM 应当理解**：
- `node --check` 只做语法解析，不执行代码，最快最安全
- ESLint 是最主流的 JS linter，项目通常有 `.eslintrc` / `eslint.config.js` 配置。如果配置文件存在，ESLint 会自动加载，不需要传额外参数
- `quick-lint-js` 用 C++ 实现，检查速度比 ESLint 快几个数量级，适合大型项目快速扫描

#### TypeScript 工具家族
```markdown
工具    | 类型    | 检查维度                    | 典型命令
--------|--------|----------------------------|-----------------------
tsc --noEmit | 类型检查器 | 类型错误、语法错误、模块解析 | tsc --noEmit
eslint  | Linter | TS 特有的 lint 规则（如 @typescript-eslint） | eslint '**/*.ts'
```

**LLM 应当理解**：
- `tsc --noEmit` 是最关键的 TypeScript 检查——它运行完整的类型检查器，这是 ESLint 做不到的
- `tsc` 会自动读取 `tsconfig.json`，所以应该在项目根目录执行
- ESLint 需要用 `@typescript-eslint` 插件才能检查 TS 文件

#### Go 工具家族
```markdown
工具           | 类型    | 检查维度                    | 典型命令
---------------|--------|----------------------------|-----------------------
go vet         | 编译器级 | 可疑构造、不可达代码、锁误用 | go vet ./...
gofmt -e       | 格式化器 | 代码格式是否符合 gofmt 标准 | gofmt -e file.go
golangci-lint  | Linter 集合 | 数十个 linter 的集成运行器 | golangci-lint run ./...
staticcheck    | Linter | 高级静态分析、性能建议、死代码 | staticcheck ./...
```

**LLM 应当理解**：
- `go vet` 是 Go 团队官方维护的，"证书级别"的检查——它的警告基本上都是真问题
- `gofmt` 的格式标准是 Go 社区的硬性约定，`-e` 参数会输出所有错误（包括非致命错误）
- `golangci-lint` 是一个 linter 的"瑞士军刀"——它内部集成了 `staticcheck`、`govet` 等数十个 linter
- `./...` 表示递归所有子目录，这是 Go 工具链的惯例

#### Rust 工具家族
```markdown
工具          | 类型    | 检查维度                    | 典型命令
--------------|--------|----------------------------|-----------------------
cargo check   | 编译器级 | 类型检查、借用检查、生命周期 | cargo check --workspace
rustc --emit metadata | 编译器级 | 快速元数据检查（无代码生成） | rustc --emit metadata file.rs
rustfmt --check | 格式化器 | 代码格式是否符合 rustfmt 标准 | rustfmt --check src/**/*.rs
```

**LLM 应当理解**：
- Rust 的编译器本身就是一个极其严格的静态分析器——它的借用检查器是其他语言没有的
- `cargo check` 比 `cargo build` 快得多，因为它不做代码生成和链接
- `--workspace` 检查工作区中所有包，对多 crate 项目很重要

#### Shell 工具家族
```markdown
工具       | 类型    | 检查维度                    | 典型命令
-----------|--------|----------------------------|-----------------------
bash -n    | 编译器语法 | 语法错误（缺失引号、括号不匹配） | bash -n file.sh
shellcheck | Linter | 未引用变量、注入风险、POSIX 兼容性 | shellcheck file.sh
```

**LLM 应当理解**：
- `bash -n` 只做语法解析（`-n` = no execution），不会执行脚本，所以非常安全
- ShellCheck 的每条警告都有一个编号（如 SC2086），可以通过编号查文档了解详细原因

#### PHP 工具家族
```markdown
工具           | 类型    | 检查维度                    | 典型命令
---------------|--------|----------------------------|-----------------------
php -l         | 编译器语法 | 语法错误（Parse error）    | php -l file.php
parallel-lint  | 编译器语法 | 并行语法检查（多文件）     | parallel-lint src/
phpcs          | Linter | 编码标准（PSR-1/PSR-12 等） | phpcs --standard=Generic src/
```

**LLM 应当理解**：
- `php -l`（`-l` = lint）是 PHP 的内置语法检查器，对每个文件只检查语法
- `parallel-lint` 是对 `php -l` 的并行封装，适合检查整个目录
- `phpcs` 的 `--standard` 指定编码标准，常见的有 `Generic`、`PSR1`、`PSR12`、`Squiz`

#### Ruby 工具家族
```markdown
工具    | 类型    | 检查维度                    | 典型命令
--------|--------|----------------------------|-----------------------
ruby -c | 编译器语法 | 语法错误                   | ruby -c file.rb
rubocop | Linter | 风格、代码坏味、Metrics、安全性 | rubocop
```

**LLM 应当理解**：
- `ruby -c`（`-c` = check syntax）只检查语法，不执行代码
- RuboCop 的检查分为多个"部门"：Style、Lint、Metrics、Naming、Security 等

#### 其他语言工具速查

```markdown
语言          | 编译器语法检查命令                        | Linter/其他
--------------|----------------------------------------|---------------------------
Kotlin        | kotlinc -Werror file.kt                 | ktlint, detekt
Scala         | scalac -Ystop-after:parser file.scala   | scalafmt --test
C#            | dotnet build --no-restore               | dotnet format --verify-no-changes
Swift         | swiftc -parse file.swift                | swiftlint lint --strict
Dart/Flutter  | dart analyze                             | dart format --set-exit-if-changed .
Lua           | luac -p file.lua                        | luacheck .
Perl          | perl -c file.pl                         | perlcritic --brutal .
Haskell       | ghc -fno-code file.hs                   | hlint .
Erlang        | erlc -o /tmp -Wall file.erl             | （内置 -Wall）
Elixir        | mix compile --warnings-as-errors        | mix credo --strict
R             | Rscript -e 'parse(file="script.R")'     | （社区包 lintr）
Fortran       | gfortran -fsyntax-only file.f90         | （无主流 linter）
Ada           | gnatmake -gnats file.ada               | （GNAT 内置）
COBOL         | cobc -fsyntax-only file.cbl             | （无主流 linter）
Pascal        | fpc -s file.pas                         | （无主流 linter）
VHDL          | ghdl -s file.vhd                        | （GHDL 内置）
Verilog/SV    | iverilog -s file.v / verilator --lint-only | （Verilator 更强）
Groovy        | groovyc -o /tmp/out file.groovy         | （社区工具 CodeNarc）
MATLAB        | matlab -batch "checkcode('file.m')"     | （MATLAB 内置）
SQL           | （无标准编译器语法检查）               | sqlfluff lint models/
HTML          | （无标准编译器语法检查）               | tidy -e file.html, vnu.jar
CSS           | （无标准编译器语法检查）               | stylelint '**/*.css'
```

> **对于这些语言，LLM 的理解重点**：大部分语言的编译器本身就是最好的语法检查器。如果该语言有官方 linter（如 SwiftLint、RuboCop），优先使用；如果没有，编译器的 `-Wall -Werror` 就是最高级别的检查。

---

## 步骤五：构建和执行审计命令

### 命令构建方法论

LLM 应根据以下要素动态构造命令：

```
命令 = 工具路径 + 检查模式 + 目标路径 + 特殊参数
```

**要素1：检查模式**
- `-fsyntax-only`（GCC/Clang）— 只做语法解析
- `--check`（node）、`-c`（ruby）、`-l`（php）— 检查语法
- `--noEmit`（tsc）— 只做类型检查，不输出文件
- `lint` / `check` — linter 的检查子命令

**要素2：目标路径**
- 单文件：直接传文件路径
- 整个目录：传目录路径或通配符（`./...`、`src/**/*.js`、`.`）
- 完整项目：在项目根目录执行（自动读取项目配置）

**要素3：特殊参数**
- `--enable=all` — 启用所有检查（cppcheck）
- `--workspace` — 检查工作区所有包（cargo）
- `-Xlint:all` — 启用全部编译器警告（javac）
- `--strict` — 严格模式（swiftlint, mix credo）
- `--brutal` — 最严格级别（perlcritic）
- `--error-exitcode=1` — 有警告也返回非零退出码

### 典型场景的命令构建示例

**场景1：审计单个 Python 文件**
```
分析：
  - 语言：Python
  - 范围：单个文件
  - 环境：可能是 Windows

构建命令：
  1. python -m compileall -q file.py    ← 语法检查（最快）
  2. pyflakes file.py                    ← 逻辑检查（无风格噪声）
  3. ruff check file.py                  ← 全面检查（速度快）
  4. flake8 file.py                      ← 风格检查（PEP8）

注意：
  - 对单文件不需要用 .（当前目录），直接传文件名
  - ruff 的默认规则集已经很全面，不需要额外参数
```

**场景2：审计一个 Cargo 项目**
```
分析：
  - 语言：Rust
  - 项目结构：有 Cargo.toml，多模块
  - 范围：完整项目

构建命令：
  1. cargo check --workspace 2>&1    ← 借用检查 + 类型检查（全项目）
  2. rustfmt --check src/**/*.rs     ← 格式检查

注意：
  - cargo check 必须在有 Cargo.toml 的目录执行
  - 不要在子目录执行，会找不到 Cargo.toml
  - 用 2>&1 重定向 stderr，因为 Rust 的错误信息走 stderr
```

**场景3：审计一个 Express 后端项目（JS）**
```
分析：
  - 语言：JavaScript
  - 项目结构：src/routes/, src/models/, src/middleware/
  - 配置文件：package.json, 可能 .eslintrc.js
  
构建命令：
  1. node --check src/app.js        ← 先检查入口文件语法
  2. eslint 'src/**/*.js'           ← 如果有 eslint 配置会自动加载
  3. jshint src/                    ← 快速补充检查

注意：
  - 如果项目很大（数百个文件），node --check 可以只检查入口
  - eslint 会自动向上查找配置文件，在项目根目录执行即可
```

### Windows 环境注意事项

```markdown
在 Windows 上执行命令时要注意：
  1. 路径分隔符用 \ 或 / —— PowerShell 两者都支持
  2. 某些工具没有 Windows 版本（如 sparse）—— 跳过并在报告中注明
  3. 某些命令可能需要 PowerShell 语法调整：
       - 重定向：2>&1   （和 bash 一样）
       - 多命令：用 ; 分隔或用 try/catch
       - 路径引号：如果路径有空格，用双引号包裹
  4. 跨平台工具首选：eslint, ruff, cppcheck, shellcheck（都有 Windows 版）
```

### 退出码的语义理解

LLM 必须理解工具退出码的含义，不能只看输出：

```markdown
退出码 0     → 无问题（即使有一些 info 级别的输出）
退出码 1     → 有错误（语法错误、编译失败）
退出码 2     → 工具有效但检查失败（通常表示有警告被计为错误）
非零退出码    → 需要检查输出内容，排除工具本身的问题

重要：
  - 某些工具（如 cppcheck --error-exitcode=1）强制让警告也触发非零退出码
  - 退出码非零不一定是"工具报错"，可能是"代码有问题"
  - 区分"工具不存在"（command not found）和"工具运行但发现代码问题"
```

---

## 步骤六：结果解读（智能解析）

### 通用输出模式识别

几乎所有编译器和 linter 的输出遵循类似的模式：

```markdown
模式1：标准位置格式
  file:line:col: severity: message
  示例：src/main.c:42:5: error: expected ';' before 'return'

模式2：简化位置格式
  file:line: severity: message
  示例：app.py:15: SyntaxError: invalid syntax

模式3：编号警告格式
  file:line:line: WARNING: code: message
  示例：script.sh:20:22: warning: SC2086: Double quote to prevent globbing
```

### 严重程度判定标准

```markdown
真·错误（必须读码）    → error, ERROR, SyntaxError, Fatal, ParseError
  含义：代码无法通过编译/解析，逻辑一定有问题

潜在问题（统计即可）  → warning, WARN, W, caution, advice
  含义：代码可能有 bug 或反模式，但不影响编译

风格建议（统计即可）  → note, info, style, refactoring, convention
  含义：代码风格/可读性改进建议

工具环境缺失          → command not found, not recognized, 不是内部或外部命令
  含义：工具未安装，跳过并报告
```

### 输出过滤与噪音消除

```markdown
有些工具的输出包含大量噪音，LLM 应该能过滤：
  - progress/status 行（如 cppcheck 的 "Checking file..."）
  - 工具自身的 banner/版本信息（如 "PMD 6.55.0"）
  - 重复的相同警告（同一个错误可能在多个工具中重复报告）
  - 第三方代码的警告（node_modules/, vendor/, .cargo/ 中的代码）

过滤方法：
  - 识别和跳过以 [progress] 开头或包含百分比的行
  - 合并不同工具报告的相同位置的问题
  - 排除依赖目录中的警告（除非用户要求检查依赖）
```

---

## 步骤七：按需读码

**只有 error 级别的问题才需要 LLM 读取源码进行人工复核。**

### 读码策略

```markdown
当工具报告了语法错误/编译错误后：
  1. 从工具输出提取：文件路径、行号、列号、错误描述
  2. 用 Read 读取报错文件，以报错行为中心（前后各 10~15 行）
  3. 结合工具的错误信息和源码上下文，判断：
       - 这是个什么错误？（缺少分号？类型不匹配？未定义变量？）
       - 错误的根因是什么？（手误？API 变更？重构遗留？）
       - 建议的修复方案是什么？

当工具只报告了 warning / style 问题时：
  不读取源码。统计数量并在报告中列出即可。
  理由：warning 级别的检查是概率性的，LLM 读码判断的性价比很低。
```

---

## 步骤八：审计报告生成

将以上步骤的结果汇总为结构化审计报告：

```markdown
## 代码审计报告

**目标**: <路径>
**语言**: <识别的语言>
**策略**: <快速语法检查 / 标准质量审计 / 深度安全审计>
**工具链**: <实际执行的工具列表>

---

### 审计摘要

| 指标 | 数值 |
|------|------|
| 审计文件 | N |
| 运行工具 | N |
| 语法错误 | N |
| 编译警告 | N |
| 风格问题 | N |
| 工具缺失 | N |

---

### 各工具详情

#### 1. <工具名> — ✅ 已安装
```
<工具输出>
```

#### 2. <工具名> — ❌ 未安装（跳过）
```
---
```
...

---

### 问题汇总

| # | 严重程度 | 工具 | 文件 | 行号 | 描述 |
|---|---------|------|------|------|------|
| 1 | 高危 | gcc | main.c | L42 | 缺少分号 |

---

### 工具环境

| 工具 | 状态 |
|------|------|
| gcc | ✅ 可用 |
| cppcheck | ❌ 未安装 |

---

### 审计结论

**总体评价**: ...
**高风险问题**: ...
**改进建议**: ...
```

---

## 重要原则

1. **先问再动手**：每次审计前先问用户要什么深度（快速语法检查 / 标准质量审计 / 深度安全审计），不要默认假设。用户用自然语言回答也没关系，映射到对应策略即可
2. **真正理解工具**：LLM 必须知道每个工具是什么类型的检查器（编译器/linter/格式化器/安全扫描）、它查什么、不查什么，而不是记住命令字符串
3. **灵活构建命令**：根据策略深度、项目结构（单文件 vs 完整项目）、语言版本（Python 2 vs 3、C++11 vs 20）、环境（Windows vs Linux）动态调整参数
4. **智能解读输出**：能区分真正的错误和风格建议，能过滤噪音，能理解退出码的语义
5. **按需读码**：只有语法/编译错误值得读码分析，warning 和 style 问题仅做统计
6. **容错执行**：工具未安装时跳过并报告，不影响后续工具的执行
7. **非破坏性**：所有审计命令都是只读的，不会修改任何文件
8. **混合语言处理**：对前后端混合项目，分别对每种语言执行对应的审计工具链

---
> Source: [snake-aabb-wtf/code-auditor-skill](https://github.com/snake-aabb-wtf/code-auditor-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
