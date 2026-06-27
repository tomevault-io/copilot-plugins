## ytlite

> Fork of YTLite (also known as YouTube Plus / YTPlus), an iOS tweak for YouTube built with Theos/Logos (Objective-C runtime hooking). The public repo contains the open-source feature code; the developer's subscription gate was compiled into their closed binary and has been removed from this fork's build pipeline.

# YTLite — Claude Session Guide

## Project Overview

Fork of YTLite (also known as YouTube Plus / YTPlus), an iOS tweak for YouTube built with Theos/Logos (Objective-C runtime hooking). The public repo contains the open-source feature code; the developer's subscription gate was compiled into their closed binary and has been removed from this fork's build pipeline.

**Active branch:** `main` (Patreon gate already stripped)  
**Build pipeline:** `.github/workflows/_build_tweaks.yml` — compiles from source via Theos on GitHub Actions (macOS runner), never downloads the developer's pre-built binary.  
**Tweak version in source:** `PACKAGE_VERSION = 3.0.1` (Makefile)  
**Target:** ARM64 iOS 13.0+, compiled against iPhoneOS 16.5 SDK

## Key Files

| File | Purpose |
|---|---|
| `YTLite.x` | Main tweak — all YouTube hooks (~1400 lines) |
| `Settings.x` | Settings UI in YouTube settings panel (~650 lines) |
| `Sideloading.x` | SSO/keychain/app-group patches for sideloaded IPA |
| `Makefile` | Theos build config |
| `YTLite.h` | Shared header |
| `YouTubeHeaders.h` | YouTube private class declarations |
| `Utils/` | Helper utilities (NSBundle category, Reachability, UserDefaults) |
| `layout/Library/Application Support/YTLite.bundle/` | Resources bundle (images, localization strings, audio) |
| `.github/workflows/_build_tweaks.yml` | CI build pipeline |

## Current Task Context

A binary dump of the latest v5.2.1 IPA build exists separately (path TBD by user). The goal is to either:

**Option A — Patch the Patreon DRM out of the compiled binary:**
The Patreon gate strings are: `"Log-inViaPatreon"`, `"FeaturesNotActivated"`, `"SkipLogInAlert"`, `"AccessDenied"`, `"ManageAccount"`. Find the method(s) referencing these, then NOP the gate check or patch the return value.

**Option B — Forward-port v5.2.1 features into this fork's source:**
Use binary analysis tools to enumerate hooks/classes in the v5.2.1 dylib that are absent from the `.x` source files, then re-implement them in Logos syntax.

## Binary Analysis — Available Tools

This is a Linux (x86-64) environment. The binary dump will be ARM64 Mach-O.

### Standard (no install needed)
```bash
file <binary>                        # identify file type
strings -a <binary>                  # extract all strings
strings -a <binary> | grep -i "patreon\|subscription\|FeaturesNot"
nm -gU <binary>                      # list exported symbols (Obj-C method names)
nm -a <binary> | grep -i "patreon"
objdump -d <binary>                  # disassemble (GNU objdump, handles Mach-O ARM64)
objdump -t <binary>                  # symbol table
objdump --macho --objc-meta-data <binary>  # Obj-C class/method dump (if LLVM objdump)
readelf -a <binary>                  # ELF fallback info
grep -a "patreon" <binary>           # binary string grep
perl -pi -e 's/PATTERN/REPLACE/g' <binary>  # binary patch
dpkg-deb -x pkg.deb outdir/         # extract .deb package
python3 -c "..."                     # scripting / hex patching
```

### Install on demand (pip3)
```bash
pip3 install lief        # best Mach-O parser on Linux — class/method/segment analysis
pip3 install capstone    # ARM64 disassembler library
pip3 install keystone-engine  # ARM64 assembler (for generating patch bytes)
```

**lief quick-start for Mach-O:**
```python
import lief
binary = lief.parse("YTLite.dylib")
# List Obj-C classes
for cls in binary.objc_classes:
    print(cls.name, [m.name for m in cls.methods])
# Find string references
for s in binary.strings:
    if "patreon" in s.lower():
        print(hex(s.virtual_address), s)
```

**capstone quick-start for ARM64 disassembly:**
```python
from capstone import *
md = Cs(CS_ARCH_ARM64, CS_MODE_ARM)
for insn in md.disasm(code_bytes, start_address):
    print(f"0x{insn.address:x}: {insn.mnemonic} {insn.op_str}")
```

### Mach-O structure notes (ARM64)
- Obj-C method implementations live in `__TEXT,__text`
- Obj-C class/method metadata lives in `__DATA,__objc_classlist`, `__DATA,__objc_methnames`
- String literals live in `__TEXT,__cstring`
- To find a gate check: locate the `__cstring` offset of `"FeaturesNotActivated"`, then find the `adrp`/`add` pair that loads it in `__text` — the surrounding branch is the gate

## Build System (Theos — runs on GitHub Actions macOS runner, not locally here)

```makefile
# Makefile variables
TWEAK_NAME = YTLite
ARCHS = arm64
TARGET = iphone:clang:16.5:13.0
$(TWEAK_NAME)_FILES = $(wildcard *.x Utils/*.m)
$(TWEAK_NAME)_FRAMEWORKS = UIKit Foundation SystemConfiguration
```

Build command: `make clean package DEBUG=0 FINALPACKAGE=1`  
Output: `packages/com.dvntm.ytlite_<version>_iphoneos-arm.deb`

Logos preprocessor syntax reference:
```objc
%hook ClassName
- (ReturnType)methodName:(ArgType)arg { return %orig; }  // call original
- (ReturnType)methodName:(ArgType)arg { %orig; }         // call original (void)
- (ReturnType)methodName:(ArgType)arg { return VALUE; }  // replace entirely
%end

%hook ClassName
%new
- (void)newMethod { ... }  // add new method
%end
```

## GitHub / MCP

Repo: `the-big-mini/ytlite`  
All GitHub interactions (PRs, issues, CI) use `mcp__github__*` tools — do NOT use `gh` CLI.

## Workflow Tips

- Always commit and push to `claude/<branch-name>` — never push directly to `main`
- After pushing, create a PR via `mcp__github__create_pull_request`
- The stop hook will remind you to commit if you forget
- When patching binaries, always keep a `.bak` of the original before modifying

---
> Source: [The-Big-Mini/YTLite](https://github.com/The-Big-Mini/YTLite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
