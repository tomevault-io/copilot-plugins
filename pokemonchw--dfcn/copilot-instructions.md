## dfcn

> - 用户自定义的小队名称（`squad.alias`）必须原样显示，包括英文、数字组合和中文；不得翻译、音译或按词义改写。

# 项目协作规范

## 自定义小队名称保持原样

- 用户自定义的小队名称（`squad.alias`）必须原样显示，包括英文、数字组合和中文；不得翻译、音译或按词义改写。
- 截图中的英文自定义小队名称不属于漏译，不得以“未翻译”为由添加别名翻译词条或覆盖原样显示逻辑。
- 自动生成的小队名称和界面提示使用各自的现有翻译逻辑；缺少名称来源信息时，不凭拼写推断自定义名称并进行翻译。

## 仅修改数据文件时不编译（优先规则）

- 仅修改词表、TOML、TSV、配置或其他数据文件，且无需变更代码时，不执行 `build-deploy.sh`、`build-deploy.ps1` 或任何编译部署；现有数据文件原地使用。
- 不为数据修改单独运行生成器，也不直接调用 `tools/build.py` 或另设入口绕过本规则。
- 只有确需构建代码时，才按实际宿主系统使用下述唯一编译部署入口。后文的构建、部署和失败重试规定均以此为前提，不得因词表或规则数据更新而触发编译。

## 按实际宿主系统使用唯一编译部署入口（必须遵守）

以当前会话的实际操作系统和工作区为准，不得把历史 Windows 路径当作所有会话的宿主环境。
Linux 使用 Linux 原生工具和 `.so` 部署；Windows 使用 Windows 原生工具和 DLL 部署，不交叉调用另一系统的入口。
两个入口均无须参数，内部调用同一 `tools/build.py`，完成数据生成、核心与加载器编译及正式部署；不另行复制产物或单独运行生成器。

### Linux

Linux 工作区为 `/home/lulika/.local/share/Steam/steamapps/common/Dwarf Fortress/dfcn`。
任务确需构建代码时，直接执行根目录 `build-deploy.sh`；仅修改数据文件时不执行。从任意目录均可执行：

```bash
/bin/bash '/home/lulika/.local/share/Steam/steamapps/common/Dwarf Fortress/dfcn/build-deploy.sh'
```

- 固定 Python：`$HOME/.cache/codex-runtimes/codex-primary-runtime/dependencies/python/bin/python3`。
- 固定原生编译器：`/usr/bin/g++`；固定依赖查询工具：`/usr/bin/pkg-config`。
- SDL2、FreeType、Fontconfig 使用当前 Linux 已安装的开发依赖，由上述 `pkg-config` 读取 `sdl2 freetype2 fontconfig`。
- 正式产物为 `dfcn/libdfcn_core.so`、`dfcn/libdfhooks.so`、游戏根目录 `libdfhooks.so`。

### Windows

Windows 当前源码工作区为 `D:\workspace\dfcn`，游戏目录与源码分开。
本机游戏及官网版映像、官网版部署开关和首次安装用的 `dfhooks.dll` 来源由
`data/runtime/native-pe-images.json` 配置；Steam 游戏目录为
`D:\program\steam\steamapps\common\Dwarf Fortress`。
任务确需构建代码且宿主为 Windows 时，执行根目录 `build-deploy.ps1`；仅修改数据文件时不执行。从任意目录均可执行：

```powershell
& "$env:SystemRoot\System32\WindowsPowerShell\v1.0\powershell.exe" -NoLogo -NoProfile -NonInteractive -ExecutionPolicy Bypass -File 'D:\workspace\dfcn\build-deploy.ps1'
```

- 固定 Python：`C:\Users\WIN11\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe`。
- 固定工具链：`C:\Users\WIN11\AppData\Local\Programs\dfcn-toolchains\w64devkit-2.9.1\w64devkit`，使用其中的 `bin\g++.exe`。
- 固定 SDL2 SDK：`C:\Users\WIN11\AppData\Local\Programs\dfcn-toolchains\SDL2-2.30.11\x86_64-w64-mingw32`。已从当前可用 SDK 安装到此持久目录，不依赖 `%TEMP%`。
- 正式产物固定为 `dfcn\dfcn_core.dll`、`dfcn\dfhooks_dfcn.dll`。Windows 游戏根目录只放引导所需的 `dfhooks.dll` 和 `dfhooks_dfcn.ini`，不再部署 `dfhooks_dfcn.dll`；入口清理旧版遗留的根目录同名 DLL。现有配置、词表和规则资源原地使用。
- 源码工作区保留本机编译产物；独立游戏安装所需的运行数据和 DLL 由同一入口部署到其 `dfcn`。首次安装缺少的 `dfhooks.dll` 从本机配置来源安装，`dfhooks_dfcn.ini` 指向该游戏安装的 `dfcn/dfhooks_dfcn.dll`。

### 所有结果都继续使用当前系统的同一入口

- 编译参数固定沿用 C++20、`-O2` 及该系统现有原生链接配置，不加入 `-g` 调试信息。脚本隔离外部编译变量，不采用 PATH 上的同名编译器，不自动下载或回退工具链。现有配置、词表和规则资源原地使用。
- 退出码 `0`：编译部署完成，本次临时文件清理完成。
- 退出码 `2`：本次应部署的正式文件全部部署完成，仅替换产生的旧动态库临时文件尚未清理；输出保留精确路径和原始错误。报告此状态即可，不重编、不换工具链、不强杀游戏、不以此要求用户批准新增检查。游戏占用的文件待正常退出或原有热重载释放；退出码 `2` 不表示清理已经完成。
- 退出码 `1`（或其他非零值，除 `2`）：编译、数据生成、正式部署或候选文件清理失败。沿本入口报告的具体错误定位、修复，再执行同一命令；不得把真实失败当作成功。
- 固定依赖缺失时，只修复上面明确列出的路径或本入口的固定配置；无法恢复则报告具体缺失项。不得扫描磁盘寻找替代编译器、下载别的工具链或尝试不同构建系统。
- 无论首次构建、修改核心、修改加载器、失败重试还是上下文丢失，都禁止直接调用裸 `python` / `python3`、`make`、`g++`、MSVC、CMake、Ninja 或旧手写编译命令。仅更新词表或规则数据不触发构建。禁止直接调用 `tools/build.py` 绕过当前系统入口，禁止通过 WSL、Git Bash、Wine 或 Proton 绕用另一系统的编译链，禁止使用 `%TEMP%\codex-world-compute-w64devkit` 中缺少标准头的缓存工具链，禁止将 `--clean` 当作修复编译错误的手段。
- README、Makefile、旧部署记录或历史对话中的其他构建命令均不得替代当前系统入口。宿主切换时自然选用对应入口，不需要为正常的 Linux／Windows 原生部署再次询问；不要在同一宿主内另选工具链或设计其他编译路径。
- 不自动启动、终止或重启游戏。仅核心逻辑变化且加载器及核心 ABI 未变时，可用原有 `Shift+F10` 关闭再开启汉化；已关闭时按一次。加载器或 ABI 改变时需正常重启游戏。编译部署完成不代表已观察到游戏内最终效果。

## 禁止新增检查项

用户已明确要求：清除这次排版任务的专项检查，禁止后续再新增所谓的检查项。

- 后续任务不得自行新增测试、自测、专项回归、smoke test、验收检查项，以及为其创建的检查脚本、测试样例、测试夹具、测试断言、结果文件或检查报告。
- 不得恢复、复制或换名重建已清理的排版专项检查，也不得继续运行或扩展这些检查。
- 工作直接围绕问题定位、修改代码或数据，以及确需构建代码时的正常编译部署展开；不得将额外检查设为交付前置步骤，不用检查数量或通过率作为完成工作的依据。
- 如需验证游戏内的最终显示效果，应如实说明是否实际观察到结果，不以模拟检查结果代替实机效果。
- 只有用户后续明确要求具体检查或测试时，才按该次明确要求执行；不得主动要求用户批准新增检查来绕过本规范。

## 不依赖人工操作辅助

- 用户反馈“没有变化”“仍然一样”或修复未生效时，一律以新核心已经加载为前提，直接定位代码、数据或界面识别逻辑的问题。不得再核对是否加载新核心，不得为此读取加载日志、比对构建时间或模块版本，也不得询问、提醒或要求用户重新加载、热重载或重启游戏。

- 以用户提供的截图为准，通过截图、代码和已有资料自行分析解决；禁止操作用户的桌面和窗口，包括切换或激活窗口、模拟键鼠输入、切换游戏页面、触发热重载及主动截取桌面或窗口。不得以定位或观察效果为由绕过，也不得转而要求用户代做操作。

- 后续任务不得依赖用户手动操作来辅助定位、修改、部署或观察效果，不得要求用户切换页面、点击控件、按快捷键、触发热重载、停留在指定界面或补做操作步骤。
- 在已有权限和工具能力内，自行完成代码或数据定位、修改，以及确需构建代码时的正常编译部署；继续遵守不操作桌面和窗口、不自动启动、终止或重启游戏，以及禁止新增检查项的规定。
- 如工具或当前环境确实无法完成某项操作，应说明具体限制及尚未观察到的结果，不将人工操作设为交付前置条件，不以编译成功或模拟结果冒充游戏内效果。

---
> Source: [pokemonchw/dfcn](https://github.com/pokemonchw/dfcn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
