## go-cpabe

> 本项目是一个 CP-ABE 加密文件共享系统，用于验证密文策略属性基加密 CP-ABE 在细粒度

# AGENTS.md

## 项目说明

本项目是一个 CP-ABE 加密文件共享系统，用于验证密文策略属性基加密 CP-ABE 在细粒度
访问控制场景中的工程应用价值。

系统目标是构建一个可演示、可测试、可解释的完整闭环：

- 用户拥有属性。
- 文件拥有访问策略。
- 文件内容使用 AES-GCM 加密。
- 数据密钥 DEK 使用 RSA 或 CP-ABE 封装。
- 只有属性满足策略的用户才能完成解密。
- 系统展示 RSA 与 CP-ABE 的性能对比、访问树和线性秘密共享方案 LSSS 可视化。

## 项目宪章

所有工作必须优先遵守：

```text
.specify/memory/constitution.md
```

如果代码、规格、计划、任务或文档与项目宪章冲突，必须先修正下游产物，或明确提出
宪章修订。

## SpecKit 开发流程

核心功能必须遵循以下流程：

```text
spec -> plan -> tasks -> implementation
```

不得在没有规格说明的情况下直接生成大段业务代码。

## 语言与文档规范

所有 SpecKit 生成的文档必须使用简体中文，包括但不限于：

- `spec.md`
- `plan.md`
- `tasks.md`
- `research.md`
- `data-model.md`
- `quickstart.md`
- `checklist.md`
- `contracts/` 目录下的接口说明文档
- `README.md`
- 其他项目说明文档

除代码标识符、文件名、路径、命令、API 路径、JSON 字段名、数据库表名、Go 类型名、
TypeScript 类型名、第三方库名称和密码学算法名称外，不允许大段使用英文。

技术术语可以采用“中文 + 英文缩写”的形式，例如：

- 数据密钥 DEK
- 访问结构 Access Structure
- 线性秘密共享方案 LSSS
- 密文策略属性基加密 CP-ABE

如果模板中存在英文标题或英文说明，生成时也必须翻译为简体中文。

代码中的变量名、函数名、接口名和包名仍然使用英文，避免破坏工程规范。

## AI 协作与代码注释规范

本项目使用 SpecKit + Codex / AI 辅助开发。所有 AI 新增或修改的核心业务代码必须包含
必要中文注释。

核心业务代码包括 Service、Repository、Handler/Middleware 中涉及业务规则的部分，以及
Crypto、Policy、Benchmark、Audit、认证授权、文件上传和访问控制相关实现。简单 DTO、
常量、字段映射、路由注册和显而易见的胶水代码可以少写解释性块注释，但不能规避关键逻辑说明。

Go 后端代码必须遵守以下硬性注释规则：

- 每一个函数、方法前都必须有注释，包括导出函数、私有函数和测试辅助函数。
- 导出的函数、方法、类型、接口、常量和变量必须符合 GoDoc 规范，注释必须以标识符名称开头。
- 私有函数和私有方法必须说明职责、调用场景、关键输入输出、边界条件和可能副作用。
- 数据库实体、DAO、Model 和 Domain 字段必须说明业务含义；关键字段需要说明来源、可空性、
  是否敏感、是否参与权限判断或外部响应。
- Handler、Service、Repository、Middleware 注释必须说明业务语义、副作用、鉴权/租户上下文、
  错误边界和安全边界。

注释重点解释“为什么这么设计”，不得简单复述代码做了什么。禁止每行都写注释，避免制造
阅读噪音；禁止“获取用户”“调用函数”“返回结果”等无意义注释。

对后端新手不容易理解的逻辑必须补充说明。涉及安全、认证、权限、Token、密码、文件上传、
加密算法、访问控制的代码，必须解释关键设计原因和安全边界。

AI 完成实现后，必须进行一次“关键注释和可读性检查”，确认注释足够解释风险点和取舍，
同时没有无意义注释。

涉及新增或修改 Go 业务代码的 `plan.md` 必须说明注释策略，`tasks.md` 必须包含“关键注释
和可读性检查”任务；交付说明必须明确函数/方法注释、GoDoc 前缀、核心模块业务语义注释和
安全边界检查是否完成。

## 技术栈约束

前端使用：

```text
Electron + TypeScript
```

后端使用：

```text
Go + Gin + Gorm
```

数据库使用：

```text
MySQL
```

缓存使用：

```text
Redis
```

文件内容加密使用：

```text
AES-GCM
```

RSA 方案使用：

```text
RSA-OAEP
```

CP-ABE 方案优先使用：

```text
Cloudflare CIRCL TKN20 CP-ABE
```

## 密码学实现约束

必须使用混合加密：

```text
文件内容 -> AES-GCM 加密
DEK -> RSA / CP-ABE 加密
```

RSA 和 CP-ABE 只能用于封装或解封装数据密钥 DEK，不得直接加密大文件内容。

CP-ABE 必须使用真实 Go 密码学库完成加密和解密。不得使用数学模拟结果、伪随机结果或
手写示例逻辑冒充真实 CP-ABE 加解密结果。

访问树和 LSSS 只能用于策略表达、策略校验和教学可视化，不得作为真实 CP-ABE 加密实现。

## 模块边界

后端至少应保持以下模块边界：

- User 模块：用户注册、登录、用户属性管理。
- File 模块：文件上传、下载、文件元数据管理。
- Policy 模块：访问策略解析、校验、访问树生成和 LSSS 可视化。
- Crypto 模块：AES-GCM、RSA-OAEP、CP-ABE 的统一封装。
- Benchmark 模块：加密耗时、解密耗时、密文大小和策略复杂度记录。
- Audit 模块：解密成功、解密失败、访问拒绝等操作日志记录。

业务代码不得将密码算法逻辑散落在 Controller、Service 或 Handler 中。所有密码算法能力
必须通过 Crypto 模块统一调用。

## 算法对比规范

RSA 是传统公钥加密基线，不是 CP-ABE 的替代品。

不得简单得出以下结论：

```text
CP-ABE 一定比 RSA 更快
RSA 一定比 CP-ABE 更差
```

Benchmark 必须区分：

- 文件内容 AES-GCM 加密耗时
- DEK 封装耗时
- DEK 解封装耗时
- 文件内容 AES-GCM 解密耗时

RSA 与 CP-ABE 的核心对比重点是 DEK 封装和解封装，不得将大文件 AES 加密耗时混入算法
对比结论中。

## 开发优先级

优先级顺序如下：

1. 文件加密共享主链路
2. RSA 与 CP-ABE 算法对比
3. 访问策略解析和可视化
4. Benchmark 与审计日志
5. 高级 CP-ABE 扩展能力

在主链路闭环完成前，不得优先投入大量时间实现用户撤销、多授权机构、策略隐藏、
区块链审计等高级能力。

## Git 提交规范

提交信息必须始终使用简体中文。

严格遵循 Conventional Commits 规范，格式必须为：

```text
<type>(<scope>): <subject>
```

常用的 `type` 必须从以下值中选择：

```text
feat, fix, docs, style, refactor, perf, test, chore
```

标题中必须包含适当的 gitmoji，例如：

```text
✨ feat(api): 新增文件加密接口
🐛 fix(crypto): 修复 DEK 解封装失败处理
📝 docs(spec): 更新语言规范说明
```

第一行标题必须限制在 50 个字符以内。

如果代码更改较复杂，第一行之后必须空一行，并使用条目列表详细解释 WHY 和 WHAT：

- WHY：为什么要这么改。
- WHAT：改了什么具体逻辑。

提交正文必须解释变更动机和具体影响，不得只复述代码本身。

## 安全边界

本项目用于学习、验证和演示 CP-ABE 应用场景。

系统不得声称自研 CP-ABE 代码具备生产级安全性。真实 CP-ABE 加解密必须依赖真实 Go
CP-ABE 库。

本项目只承诺工程演示能力，不承诺生产环境安全能力。如果用于生产环境，必须经过专业
密码学安全审计、密钥管理审计和系统安全审计。

---
> Source: [QunfengLiao/go_cpabe](https://github.com/QunfengLiao/go_cpabe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
