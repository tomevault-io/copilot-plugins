## cxdec-tools

> 本文件给后续 agent 快速了解 `cxdec-tools` 的结构和修改约束。当前工具主要用于恢复/导出 KiriKiri XP3 封包，重点适配从游戏 EXE/BOOTSTRAP 静态提取参数的 CX/HX/PackinOne 链路；不要恢复外部 `formats.json` 配置依赖。

# Agent Notes

本文件给后续 agent 快速了解 `cxdec-tools` 的结构和修改约束。当前工具主要用于恢复/导出 KiriKiri XP3 封包，重点适配从游戏 EXE/BOOTSTRAP 静态提取参数的 CX/HX/PackinOne 链路；不要恢复外部 `formats.json` 配置依赖。

## Build And Check

- `cargo fmt`：提交前格式化 Rust 代码。
- `cargo test`：运行单元测试和集成到库里的解析/哈希测试。
- `cargo run -- recover <hash_domain> <input_dir> <output_dir>`：完整恢复流程。
- `cargo run -- dump <input_dir> <output_dir>`：按当前可解析名称直接导出封包内容。
- 常用 hash domain 是 `xp3hnp`，但不要在代码里把它写死到通用逻辑。

## Project Layout

- `Cargo.toml` / `Cargo.lock`：Rust 包配置和依赖锁定。
- `README.md`：面向用户的简短说明。
- `report*.md`：逆向分析记录，不参与构建。
- `src/lib.rs`：库入口，导出 crypto、format、io、pipeline、process、struct 模块；不要重新加入 `game` 模块或外部 `formats.json` 配置入口。
- `src/main.rs`：CLI 入口，解析 `recover`/`dump` 参数，初始化 tracing 日志。
- `src/io.rs`：二进制读写 helper，集中放固定宽度整数读写、Endian、内存 cursor 和 `std::io` 读写适配；其他 Rust 文件不要重新实现 `read_u32` / `write_u32` 等低层 helper。
- `src/pipeline.rs`：完整恢复流程编排。当前顺序是准备输入输出、读取 EXE 内嵌资源并派生 bootstrap keys、构造运行时 HxCrypt 方案、挂载 XP3、收集 EXE startup TJS、TJS 字符串、SCN 字符串、普通文本、PBD 图层、TLGref 字符串，最后恢复剩余条目。STARTUP.TJS 中 `_bootStrap(...)` 调用识别和 runtime key 派生编排放在这里；不要在这里放 BOOTSTRAP PE 字节模式扫描。
- `src/format.rs`：按文件头识别 XP3 条目类型，包括 TLG/TLGref/TLGqoi、图片、音频、TJS bytecode、PBD、PSB、脚本和纯文本。

## Crypto Modules

- `src/crypto/bootstrap_alg.rs`：Cxdec bootstrap key derivation 的底层算法实现，包括 Argon2i、CxSponge/Keccak、deriveKeyBlock、control block 变换、archiveUniqueKey 和 HChaCha20 table keys。这里应只放可复用加解密/派生算法，不放 EXE 扫描或 STARTUP.TJS 业务识别。
- `src/crypto/bootstrap_key.rs`：bootstrap runtime key 派生接口。输入是已经提取好的 `BootstrapStaticData` 和 bootstrap input 字符串，输出 `BootstrapDerivedKeys`；不要在这里读取 PE、解析 STARTUP.TJS 或搜索 `_bootStrap`。
- `src/crypto/cx_hash.rs`：PackinOne 文件名/目录名 hash。注意 `file_hash` 和 `path_hash` 是两套算法，文件名恢复不能误用目录 hash。
- `src/crypto/exe_resource.rs`：EXE/BOOTSTRAP 资源解密和静态材料提取相关的低层工具，包括资源 salt、127/TEXT startup filter path、资源 ChaCha 派生流、BOOTSTRAP KV 表 payload 提取、`PARAMS` 解析、`WARNING`/`UNIQUE` payload 识别和 BOOTSTRAP PE 中 `archiveUniqueKey` 静态扫描。不要在这里做候选 EXE 选择、STARTUP.TJS 调用链识别或恢复流程编排。
- `src/crypto/hxv4_compute.rs`：解析 HXv4 索引、名称表、过滤器参数和条目级解密辅助逻辑。
- `src/crypto/hxv4_shellcode.rs`：模拟 CX shellcode 的加解密核心，包括 TPM 控制块、随机数和条目过滤器。
- `src/crypto/mod.rs`：crypto 子模块导出。

## Struct Parsers

- `src/struct/xp3.rs`：XP3 索引、条目、segment、文件名映射和 HX 元数据绑定；也负责安全输出名清洗。
- `src/struct/tjs.rs`：TJS2 bytecode 结构解析、常量池读取和轻量 disasm/常量传播。这里保持格式层职责；不要放 STARTUP.TJS、`_bootStrap`、Magalumina 或密钥派生相关业务逻辑。
- `src/struct/pbd.rs`：PackinOne PBD 解析、解密、LZ4 解包和图层图片引用提取。
- `src/struct/psb.rs`：PSB 解析，读取对象、名称 trie 和字符串表。
- `src/struct/tlg.rs`：TLGref/TLGqoi 轻量解析，提取 QRef 目标和内嵌字符串。
- `src/struct/mod.rs`：struct 子模块导出。

## Process Stages

- `src/process/common.rs`：恢复上下文、运行时 `GameScheme`/HxCrypt scheme 数据、XP3 挂载、名称提示注册、hash 索引、候选路径匹配、输出路径和安全清空输出目录。
- `src/process/dump.rs`：dump 模式实现，按当前可读的 archive path 输出文件。
- `src/process/exe.rs`：EXE 候选选择、内嵌 `STARTUP.TJS`/`BOOTSTRAP` 资源解密、BOOTSTRAP zlib 解包和资源 dump。该模块只返回解密后的资源，不派生 bootstrap keys；key 派生由 pipeline/dump 编排层调用。
- `src/process/tjs.rs`：扫描 TJS bytecode 和普通文本，从常量池、字符串字面量、文本 token 注册名称提示。
- `src/process/scn.rs`：解析场景列表和 SCN 常量池，生成 `.scn` 相关候选名。
- `src/process/pbd.rs`：从 PBD 图层数据推导同名或分层 TLG 图片候选。
- `src/process/tlg.rs`：从 TLGref/TLGqoi 中收集引用字符串并注册为名称提示。
- `src/process/script.rs`：KiriKiri 加密/压缩脚本文本的解码处理。
- `src/process/mod.rs`：`ProcessedFile` 结构和按文件类型分派的处理入口。

## Coding Conventions

- 所有函数前保留英文注释；新增函数也必须补英文注释。
- 代码和注释默认使用 ASCII。只有游戏原始文件名、字符串、路径示例等确实需要时才使用非 ASCII。
- 保持现有错误风格：CLI/流水线层通常返回 `Result<T, Box<dyn std::error::Error>>`，解析器内部可使用本地错误类型。
- 路径 hash 逻辑要非常谨慎：归一化 archive path 使用小写和 `/`；目录名通常带结尾 `/`；文件名 hash 只针对文件叶子名。
- 名称提示注册应保持确定性和去重，避免让同一个 hash 反复产生无意义候选。
- 新增恢复阶段时，应接入 `src/pipeline.rs` 的阶段日志，并在 `RecoveryContext` 里复用已有 hash/name 注册工具。
- 新增文件类型时，同时检查 `src/format.rs`、`src/process/mod.rs` 和需要的 `src/struct/*` 解析器是否都接上。
- 涉及 EXE/BOOTSTRAP/STARTUP 的改动要按职责放置：候选 EXE 选择和资源 dump 放 `process/exe.rs`，PE 资源解密、BOOTSTRAP KV payload 提取和 BOOTSTRAP PE 静态材料扫描放 `crypto/exe_resource.rs`，STARTUP 调用链识别和 key 派生编排放 `pipeline.rs`，纯 key derivation 算法放 `crypto/bootstrap_alg.rs`/`bootstrap_key.rs`，TJS 格式解析只放 `struct/tjs.rs`。
- Rust 测试统一放在对应文件最底部的 `#[cfg(test)]` 区块里，不要在业务代码中间插入测试模块。
- 不要绕过 `prepare_output_dir` 的安全检查，也不要加入会清空输入目录或当前目录的逻辑。
- 修改解析器、hash 或候选名生成时，优先添加聚焦测试；测试放在对应模块的 `#[cfg(test)]` 区块里即可。

---
> Source: [1F1E33-float32/Cxdec_Tools](https://github.com/1F1E33-float32/Cxdec_Tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
