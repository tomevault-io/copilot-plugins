## test

> ysyxSoC 构建和 mill 工具使用注意事项


# ysyxSoC 构建注意事项

## Mill 构建工具问题

### 问题现象
在 WSL 环境中执行 `make verilog` 或直接运行 mill 时，可能出现：
- 大量无关的 GUI 相关输出（libEGL、flutter、Sanmill 等）
- 构建挂起或失败
- 找不到生成的 `ysyxSoCTop.sv` 文件

### 根本原因
1. WSL 环境中设置了 `DISPLAY` 环境变量
2. 系统中可能安装了名为 "mill" 的 snap 应用（Sanmill 棋类游戏），与 Scala 构建工具 mill 产生冲突
3. mill 构建工具在有 DISPLAY 时会尝试初始化 GUI 组件

### 解决方案

**推荐方式**：在运行 mill 前禁用 DISPLAY 环境变量

```bash
# 方式1：直接设置环境变量运行
DISPLAY="" CHISEL_FIRTOOL_PATH=/path/to/firtool/bin ./mill -i ysyxsoc.runMain ysyx.Elaborate --target-dir build

# 方式2：在 ysyxSoC 目录下完整构建流程
cd ysyxSoC
rm -rf build  # 清理旧文件（如果需要强制重新生成）
mkdir -p build
DISPLAY="" CHISEL_FIRTOOL_PATH=$(pwd)/patch/firtool/firtool-1.105.0/bin ./mill -i ysyxsoc.runMain ysyx.Elaborate --target-dir build

# 后处理步骤（必须）
mv build/ysyxSoCTop.sv build/ysyxSoCFull.v
sed -i -e 's/_\(aw\|ar\|w\|r\|b\)_\(\|bits_\)/_\1/g' build/ysyxSoCFull.v
sed -i '/firrtl_black_box_resource_files.f/, $d' build/ysyxSoCFull.v
```

**不要直接使用** `make verilog`，因为 Makefile 没有设置 `DISPLAY=""`。

## 修改 ysyxSoC 配置后的重新生成

当修改以下文件后，需要重新生成 verilog：
- `ysyxSoC/src/Top.scala`（如 `sdramUseAXI` 配置）
- `ysyxSoC/src/SoC.scala`
- `ysyxSoC/src/amba/*.scala`

检查是否需要重新生成：
```bash
# 比较时间戳
ls -la ysyxSoC/build/ysyxSoCFull.v
ls -la ysyxSoC/src/Top.scala
# 如果 scala 文件比 verilog 新，则需要重新生成
```

## 外设 Verilog 文件修改

当 ysyxSoC 生成的代码引用新的 AXI4 端口时，需要同步修改外设文件：
- `ysyxSoC/perip/amba/axi4_delayer.v` - 添加 lock/cache/prot 端口
- `ysyxSoC/perip/sdram/sdram_top_axi.v` - 添加 lock/cache/prot 端口

这些端口通常可以直接透传或忽略（用于 ysyxSoC 内部互联）。

## NPC 编译

ysyxSoC 模式下编译 NPC：
```bash
cd npc
make -f Makefile.soc
# 可执行文件在 build_soc/ysyxSoCFull
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/haoyu1377871089-code) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
