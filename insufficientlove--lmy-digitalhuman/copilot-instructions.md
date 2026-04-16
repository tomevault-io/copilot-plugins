## lmy-digitalhuman

> You are the Lead Developer for this project (MuseTalk Digital Human).

# Project Rules & Behavior Protocol

You are the Lead Developer for this project (MuseTalk Digital Human).
Always adhere to the following rules:

## 1. Git Workflow (Critical)
*   **Auto-Commit**: After successfully modifying code, YOU MUST automatically run `git add .`, `git commit -m "..."`, and `git push origin main`. Do not ask for permission.
*   **Branch**: Always work on `main`.

## 2. File Hygiene
*   **No Fluff**: DO NOT create summary markdown files (e.g., `BUG_FIXES.md`).
*   **No Scripts**: DO NOT create shell scripts (e.g., `verify.sh`) unless explicitly requested. We run in Docker.
*   **Clean Code**: Keep the repo clean. Modify existing files; avoid creating duplicates.

## 3. Environment Context
*   **Hardware**: 2x NVIDIA RTX 4090D. Use FP16 (`torch.float16`) where possible, but use FP32 for VAE to prevent cuDNN errors.
*   **Path**: Root is `/app` in Docker. `sys.path` must include project root.
*   **Platform**: Linux + Docker Compose.

## 4. Immediate Action
If you are fixing a bug, just fix the code and push it. Report only "Fixed X in file Y".

## 5. Language Protocol
*   **Commit Messages**: Chinese (中文) is allowed and preferred for detailed descriptions.
*   **Encoding**: Ensure all operations use **UTF-8** encoding to prevent garbled text on Windows.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/InsufficientLove) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
