## txlock-skill

> TXLock交互式加解密工具。支持加密(enc)、解密(dec)、验证(verify)、状态查看(status)。使用BIP39助记词+index进行离线加解密。


# TXLock Skill

执行TXLock加解密操作。参数格式：`<operation> [file] [-index N]`

## 环境信息

- 项目根目录: !`pwd`
- 二进制状态: !`which txlock-enc txlock-dec 2>/dev/null || echo "未找到"`

## 执行流程

### 1. 解析参数

从`$ARGUMENTS`提取：操作类型、文件路径、index（默认777）

### 2. 检测二进制

检查顺序：
1. `which txlock-enc` (PATH)
2. `test -x ./bin/txlock-enc` (项目本地)

未找到时显示安装指引：
- 从源码编译 `https://github.com/vcvvvc/TXLock.git`

### 3. 执行操作

默认配置：
- 助记词环境变量：MNEM
- 默认index：777

#### enc（加密）

```bash
txlock-enc -in <file> -mnemonic-env <MNEMONIC_ENV> -index <index>
```

输出：文件路径、index、结果（成功/失败）、输出文件位置

#### dec（解密）

```bash
txlock-dec -in <file> -mnemonic-env <MNEMONIC_ENV> -index <index>
```

输出：文件路径、index、结果、输出文件位置

失败时提示可能原因：Index错误、助记词错误、文件损坏

#### verify（验证）

三步流程：
1. 加密文件
2. 解密文件
3. 字节级比较（`cmp -s`）

输出：每步状态、最终结论（成功/失败）

#### status（状态）

显示：
- lockfile/lock/ 文件列表和数量
- lockfile/unlock/ 文件列表和数量

如果目录不存在，提示首次使用时自动创建。

## 错误处理

### 环境变量未设置

必须完整提示两种设置方法：
- 临时：`export <MNEMONIC_ENV>="..."`
- 持久：询问是否自动创建`~/.txlock-env.sh`（权限600）

### 文件不存在

显示：`❌ 错误：文件不存在: <file>`

### 未知操作

列出支持的操作：enc、dec、verify、status

## 安全原则

- 不在命令行输入助记词
- 不询问用户输入助记词
- 配置文件只存储环境变量名

---
> Source: [vcvvvc/txlock-skill](https://github.com/vcvvvc/txlock-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
