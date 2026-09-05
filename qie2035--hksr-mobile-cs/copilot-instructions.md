## hksr-mobile-cs

> Windows 版崩坏：星穹铁道 MobileUI 启动器（.NET 10 / x64）。以挂起状态创建游戏进程 → 注入 GameAssembly.dll → 在 il2cpp 节中

# AGENTS.md — hksr-mobile (C#)

Windows 版崩坏：星穹铁道 MobileUI 启动器（.NET 10 / x64）。以挂起状态创建游戏进程 → 注入 GameAssembly.dll → 在 il2cpp 节中
AOB pattern scan 定位地址 → 写入值 `2`（启用 MobileUI）→ 恢复主线程。注入写入发生在游戏代码启动之前。

## 构建 / 运行

```powershell
dotnet build            # Debug
dotnet build -c Release # Release
dotnet run -- -h        # 帮助
```

产物：`bin\Release\net10.0-windows\HksrMobile.exe`。x64（`PlatformTarget`），`net10.0-windows`。

运行时需管理员权限（注入）。本工具自带 UAC 提权（`Admin.EnsureAdmin()`），普通权限启动会自动弹 UAC 以管理员身份重启自身。
游戏必须由本工具拉起（挂起启动），不能手动启动后附加。 **无自动化测试**——验证方式 = 编译 + 真机注入（检查日志出现
`MobileUI 已启用`）。

## 架构

- `Program.cs` — 入口：解析 CLI → 初始化 Serilog（`-v` 开 Debug）→ `Admin.EnsureAdmin()`（非管理员 UAC 提权重启）→ 加载配置 →
  解析游戏路径（CLI > 配置 > 注册表）→ 终止残留进程询问 → `SuspendedProcess.Create` 挂起 → `Injector.InjectDll` →
  `Pe.Parse` 定位 il2cpp 节 → `MobileUi.EnableMobileUi` → `Resume()`
- `Cli.cs` — System.CommandLine 解析 `--game-path` / `--config` / `-v`；`-h`/`--help` 打印后退出
- `Config.cs` — JSON 配置（`System.Text.Json`，命名 camelCase）读写；注册表回退用 `Microsoft.Win32.Registry`（
  `Software\{vendor}\HYP\1_<n>\{game_key}\GameInstallPath`，CN 优先）
- `Core/` — 领域逻辑：
    - `Injector.cs` — Iced 生成 64 位 shellcode（`call LoadLibraryW` + 回写基址），远程分配/RWX/`CreateRemoteThread` 驱动
    - `MobileUi.cs` — AOB pattern scan（`??` 通配符），`MobileUiPatterns` 按最新→最旧匹配，写入值 2
    - `Pe.cs` — `System.Reflection.PortableExecutable` 解析 PE 头，定位 `il2cpp`/`.il2cpp` 节
    - `ProcessOps.cs` — `EnumProcesses` 枚举 + `QueryFullProcessImageNameW` 匹配路径，询问是否终止
- `Native/` — Win32 封装：`Admin.cs`（UAC）、`ProcessMemory.cs`（远程内存读写 + `RemoteAlloc` RAII）、`SuspendedProcess.cs`
  （挂起进程 RAII，未 `Resume` 时 `Dispose` 自动终止）
- `NativeMethods.txt` — CsWin32 声明。 **全部 Win32 API 经 CsWin32 生成，禁止手写 `DllImport`**

## 约定

- 源码注释、日志、控制台输出用中文；日志走 Serilog（info/warn/error/debug），仅交互询问用 `Console.WriteLine`/`ReadLine`
- 现代 C#：`Nullable` + `ImplicitUsings`，file-scoped namespace，`record`/`record struct`；Win32 用 CsWin32 强类型（`HANDLE`/
  `PCWSTR`/`BOOL`），需要 `unsafe` 用指针
- **注入/PE/scan 逻辑改动后必须真机验证**——潜在 bug 表现为游戏闪退或 MobileUI 未生效
- **本机反作弊环境下 `CreateToolhelp32Snapshot`（Process32/Module32）会卡死**，进程枚举一律 `EnumProcesses`（psapi）；模块基址用
  shellcode 回写，不依赖模块枚举
- Windows bash 坑：`timeout` 是 Windows 的 `timeout.exe`（别用来限时），`find` 在非 ASCII 路径有转义问题，改用 `ls`/`grep`
- 依赖一律通过 `dotnet add package`（或直接编辑 csproj 的 PackageReference）添加；AOB 签名随游戏版本变化，`MobileUiPatterns`
  需随版本更新

## 备注

- 配置文件 `hksr-mobile.json`（含本机游戏路径）被 `.gitignore` 忽略；`bin/`、`obj/`、`.idea/` 同样忽略
- 游戏版本更新后 pattern 失效时，更新 `Core/MobileUi.cs` 中 `MobileUiPatterns`

---
> Source: [QiE2035/hksr-mobile-cs](https://github.com/QiE2035/hksr-mobile-cs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
