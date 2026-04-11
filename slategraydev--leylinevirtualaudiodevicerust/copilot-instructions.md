## leylinevirtualaudiodevicerust

> **Product Name**: Leyline Audio Driver

# GEMINI.MD: Leyline Audio Driver Technical Specification

**Product Name**: Leyline Audio Driver
**Version**: 0.1.0
**Platform**: Windows 10/11 (x64)
**Framework**: Rust-native WaveRT

## Current Status: Phase 2 (Endpoint Visibility)

### Status Summary
The driver installs and PortCls initializes successfully. All source files have undergone a full documentation audit to meet 'Sleepy Architect' standards, and all compiler warnings have been resolved. The Audio Endpoint Builder (AEB) currently fails to enumerate endpoints on Hyper-V; investigation into missing `PKEY` properties and INF policy overrides is the primary focus.

### Completed
- **Kernel Core**: DriverEntry and PcInitializeAdapterDriver are functional.
- **WaveRT Registration**: StartDevice successfully registers the streaming filter.
- **Topology**: Port, bridge pins, and internal routing are defined.
- **Binary Integrity**: Subsystem 1 (Native) is enforced for all kernel binaries.
- **Environment**: Build containment established via eWDK and LaunchBuildEnv.ps1.
- **Documentation Audit**: Full project source (Kernel, Shared, APO, HSA) and README.md updated to 'Sleepy Architect' standards.
- **Warning Resolution**: Zero-warning state achieved across all Rust crates.
- **Modularization**: Descriptor structures refactored into dedicated modules (common, render, capture).
- **Diagnostics**: Integrated ETW provider registration for kernel-mode tracing.
- **Control**: Implemented Control Device Object (CDO) dispatchers for custom IOCTL handling.
- **IP Protection**: Standardized headers applied to all source files.

### In Progress
- **Endpoint Visibility**: Diagnosing why AEB ignores the topology on Hyper-V.
- **Trait Implementation**: Mapping foundational structures to wdk traits.
- **HSA Integration**: Development of the WinUI 3 Hardware Support App bridge.

---

## 0. Core Objective
The objective is a production-grade WaveRT virtual audio driver.
- **Core**: WDM-based implementation using zero-copy buffering.
- **Endpoints**: Two discrete nodes (Input and Output).
- **Latency**: Targeted at <10ms to meet professional requirements.
- **Control**: WinUI 3 bridge for real-time routing.

---


## 1. Technical Specification

### 1.1. Build Environment
- **Compiler**: Enterprise WDK (eWDK) 26H1/28000.
- **Rust Toolchain**: 1.75+ (no_std), cargo-wdk, and cargo-make.
- **LLVM**: 17.0.6 for stable bindgen output.

### 1.2. Driver Architecture
- **Framework**: windows-drivers-rs (wdk and wdk-sys).
- **Safety**: Unsafe blocks require // SAFETY: comments justifying invariants.
- **Low Latency**: Implementing WASAPI Exclusive Mode and IAudioClient3 small-buffer paths.

### 1.3. Hyper-V Constraints
- **AEB Behavior**: Endpoint Builder may remain idle on headless VMs.
- **Symptom**: MMDeviceEnumerator returns zero endpoints despite successful load.
- **Investigation**: Testing PKEY_AudioEndpoint_FormFactor and INF policy overrides.

---

## 2. Developer Protocols

### 2.1. Zero-Warning Policy
- **Requirement**: Errors and warnings must be resolved before a session ends.
- **Clippy**: cargo clippy --workspace --all-targets -- -D warnings is mandatory.
- **Verification**: Confirm zero warnings in the final build log.

### 2.2. Source Control Protocol
- **Commits**: Do not execute git commit without explicit instruction.
- **Style**: Conventional Commits (feat, fix, docs) with technical context.

### 2.3. Artifact Management
- **Hygiene**: Keep the repository free of dll, sys, obj, and package folders.
- **Exclusions**: Maintain .gitignore for temporary build files.

---

## 3. Reference Documentation
- [Introducing the WaveRT Port Driver](https://learn.microsoft.com/en-us/windows-hardware/drivers/audio/introducing-the-wavert-port-driver)
- [Developing a WaveRT Miniport Driver](https://learn.microsoft.com/en-us/windows-hardware/drivers/audio/developing-a-wavert-miniport-driver)
- [Low Latency Audio Guidelines](https://learn.microsoft.com/en-us/windows-hardware/drivers/audio/low-latency-audio)

---
**Last Updated**: February 28, 2026

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/slategraydev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/slategraydev)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
