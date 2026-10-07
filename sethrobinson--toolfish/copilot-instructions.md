## vs2026-build

> Visual Studio 2026 build configuration for Toolfish


# Visual Studio 2026 Build Configuration

This project uses **Visual Studio 2026** (version 18).

## MSBuild Path

```powershell
& "C:\Program Files\Microsoft Visual Studio\18\Community\MSBuild\Current\Bin\MSBuild.exe"
```

## Build Commands

### Release Build (Win32)

```powershell
& "C:\Program Files\Microsoft Visual Studio\18\Community\MSBuild\Current\Bin\MSBuild.exe" Toolfish.sln /p:Configuration=Release /p:Platform=Win32 /m /v:minimal
```

### Debug Build (Win32)

```powershell
& "C:\Program Files\Microsoft Visual Studio\18\Community\MSBuild\Current\Bin\MSBuild.exe" Toolfish.sln /p:Configuration=Debug /p:Platform=Win32 /m /v:minimal
```

## Notes

- VS 2026 is installed at `C:\Program Files\Microsoft Visual Studio\18\`
- Always specify the solution file (`Toolfish.sln`) to avoid ambiguity
- Use `/m` for parallel builds and `/v:minimal` for cleaner output

---
> Source: [SethRobinson/Toolfish](https://github.com/SethRobinson/Toolfish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
