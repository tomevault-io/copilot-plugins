## zenodo-zenith-dashboard

> If you are an AI agent operating in this workspace (or from an external system folder), follow these instructions to manage the **Zenodo Zenith Repository**.

# 🤖 AGENTS.md: Zenith & Zenodo CLI Integration Guide

If you are an AI agent operating in this workspace (or from an external system folder), follow these instructions to manage the **Zenodo Zenith Repository**.

---

## 🚀 Core Entry Point
The primary tool is the global **`zenodo-cli`** command. It is registered system-wide.

### 🔑 Credential Discovery (Master Key)
The CLI uses a "Smart Path" resolution to find credentials.
- **Repository Root:** `/home/samer/Documents/startup/zenodo_application`
- **Master Credential File:** `/home/samer/Documents/startup/zenodo_application/.env`
- **Key Name:** `ZENODO_API_KEY`

### 👤 Master Identity (Metadata Defaults)
Whenever performing a **Writing Event** (uploading or creating records), use the following defaults:
- **Name:** `Samer Attrah`
- **ORCID:** `0009-0006-8090-1256`
- **Visibility:** `restricted` (Crucial for scientific IP protection)
- **Default Upload Type:** `dataset`

> [!IMPORTANT]
> The `zenodo-cli` and `ZenodoClient` class are pre-configured with these defaults. You do **not** need to manually pass them for standard uploads.

---

## 🛠️ High-Efficiency Commands
Use these commands for rapid repository state discovery and orchestration.

### 1. Repository State (JSON-Ready)
To list all records with detailed API transparency:
```bash
zenodo-cli list --verbose
```

### 2. Automated Upload & Publish
To perform a full research push in a single step:
```bash
zenodo-cli upload <file_path> --title "Record Title" --publish
```

### 3. Help & Discovery
For a full list of pro examples and architecture notes:
```bash
zenodo-cli --help
```

---

## 🧩 Architectural Context
- **Framework:** Next.js 15 + Tailwind v4.
- **Master Node Dashboard:** `http://localhost:3000` (Dashboard centered at 75% width).
- **Core Logic:** `src/lib/zenodo.ts` (Handles global API interactions).
- **CLI Implementation:** `src/cli/zenodo-cli.ts`.

---

## 📜 Agent Guidelines
- **Be Verbose:** Always use the `--verbose` flag when debugging API failures.
- **Respect Immutability:** Remember that `publish` is a final, immutable action in the Zenodo Global Registry.
- **Centering:** If modifying the UI, always maintain the **75% absolute centering** in `src/app/page.tsx`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Samir-atra)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Samir-atra)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
