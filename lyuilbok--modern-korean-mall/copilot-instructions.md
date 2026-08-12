## modern-korean-mall

> This project is a single-file SaaS web application called **AgriLeitner Pro SaaS** located at `C:\Users\유일복\Desktop\word_app\leitner_app.html`. It is a vocabulary trainer based on the Leitner system for high-difficulty agricultural/engineering terminology.

# AgriLeitner Pro SaaS - Project Instructions

This project is a single-file SaaS web application called **AgriLeitner Pro SaaS** located at `C:\Users\유일복\Desktop\word_app\leitner_app.html`. It is a vocabulary trainer based on the Leitner system for high-difficulty agricultural/engineering terminology.

## Core Tech Stack
- **Frontend**: React (v18), Tailwind CSS, Babel (running in-browser via CDNs for zero-install execution).
- **Backend**: Supabase (used for Authentication and cloud Database persistence).
- **Database Tables**:
  - `public.agri_profiles`: Handles billing tiers ('Free', 'Pro') and AI generation counts.
  - `public.agri_cards`: Stores user-specific flashcards.

## API Configurations
- **Supabase URL**: `https://cfimyvvecsoqeicsjezo.supabase.co`
- **Supabase Anon Key**: `[REDACTED_SECURELY] (Refer to private project memory .gemini/tmp/project/memory/MEMORY.md)`
- **Google OAuth Client ID**: `[REDACTED_SECURELY] (Refer to private project memory .gemini/tmp/project/memory/MEMORY.md)`
- **Google OAuth Client Secret**: `[REDACTED_SECURELY] (Refer to private project memory .gemini/tmp/project/memory/MEMORY.md)`
- **Google OAuth Redirect URI**: `https://cfimyvvecsoqeicsjezo.supabase.co/auth/v1/callback`

## 🎛️ One-Stop AI Command Control Center (`command_center.py`)
To make development completely frictionless, we have built a **Unified AI Command Control Center** program. Instead of managing multiple separate agents or terminal windows, you can control the entire project from a single, interactive CLI cockpit.

### 🌟 Core Capabilities
1. **🤖 Natural Language AI Code Editor**: Type any feature request or design modification in plain Korean (e.g., *"구글페이 결제 완료 성공 모달창 더 이쁘게 다듬어줘"*). The console directly communicates with the Gemini 1.5 Flash API, surgically rewrites `leitner_app.html`, creates an automatic safety backup, and saves the file!
2. **💻 Integrated Local Server**: Launch the background local test server (`http://localhost:5000`) and automatically open the app in your default browser with a single press of a button.
3. **🐙 Automated GitOps GitHub Sync**: Instantly stage, commit, and push your entire codebase directly to your GitHub repository.
4. **💾 Safety Rollback**: Did the AI write something you want to undo? Instantly restore the previous version of your code from the automatically generated `.bak` safety file.

### 🎮 How to Launch
1. Double-click **`Run_Command_Center.bat`** in your project directory.
2. Enter your Gemini API Key on first launch (saved locally to `gemini_key.txt`).
3. Speak, test, and sync your application to GitHub with absolute zero friction!

## 🧠 Connect AI Lite - Autonomous Agent Studio (`connect_ai_lite.py`)
Inspired by the open-source **Connect AI** agent in `D:\ai lab`, we have built an elite, lightweight, and extremely high-efficiency **Autonomous AI Coding Agent Engine** from scratch in pure Python.

### 🌟 Core Capabilities
- **Reason-Act-Observe Loop**: Give the agent a broad natural language goal, and it will autonomously use tools in a step-by-step thinking loop until the objective is reached.
- **Integrated Tool Suite**:
  1. `read_file`: Reads code from any file.
  2. `write_file`: Surgically writes modified code, automatically creating `.bak` backups of the original file.
  3. `list_dir`: Scans folder directories.
  4. `run_command`: Runs local shell/terminal commands (e.g. `npx cap sync`, `python -m http.server 5000`).
- **🔒 Complete Safety Controls**:
  - Out of safety, the agent will *never* run command-line scripts without displaying them and asking for your explicit authorization (`[y/n]` prompt), giving you complete control over your system.
- **🔌 Dual AI Engine Support (Hybrid Agent)**:
  - **Gemini Engine (Cloud)**: Connected to Gemini 1.5 Flash for high-speed, incredibly smart, and super cheap cloud-based edits.
  - **Ollama Engine (100% Local/Free)**: Connects to your local Ollama server (e.g., `qwen2.5-coder:7b`, `llama3`, `codegemma`) for a completely offline, zero-cost development loop.

### 🎮 How to Launch
1. Double-click **`Run_Connect_AI_Lite.bat`** in your project directory.
2. Select **`[1] 자율 행동 AI 에이전트`** and type your goal. Watch the AI think, analyze, call its own tools, and deliver the final results in real-time!
3. Select **`[2] AI 에이전트 환경 설정`** to toggle between Gemini and local Ollama models instantly.

## 🎮 Unity Game-Based AI Space Integration (Metaspace Cockpit)
We have laid the foundation for an immersive, game-based AI collaboration tool using the **Unity Game Engine** as a 3D/2D frontend visual cockpit, and **Node.js** as the backend developer brain.

### 🔌 Hybrid Architecture
- **Backend (`unity_server.js`)**: A lightweight, zero-dependency Node.js HTTP server listening on port `5001`. It runs the autonomous Gemini Agent loop, executes file I/O, runs local commands, and tracks real-time state (`status`, `thought`, `action`, `observation`).
- **Frontend (`AIWorkspaceManager.cs`)**: A professional Unity C# network script that polls the Node.js agent status and maps states (e.g., `THINKING`, `TYPING`, `SUCCESS`) directly to Unity 3D character animator controller parameters (`isThinking`, `isTyping`, `isCheering`).

### 👥 Dual-Collaboration Strategy (Gemini CLI vs. Google Gems)
How should you build this game-based cockpit? The absolute best approach is **co-collaborating with BOTH Gemini CLI and Google Gems**:
1. **Gemini CLI (Your Chief Builder & Engineer)**:
   - *Why*: I have direct local access to your filesystem, terminals, and compilers. I will do the heavy lifting of generating raw C# scripts, structuring Unity directories, managing background API servers, and committing code.
2. **Google Gems (Your Creative Directors & Game Designers)**:
   - *Why*: Gems are fantastic for rapid brainstorming, UI/UX design mockups, character design, writing individual gameplay/movement C# scripts, and asset sourcing guides in a conversational window.

### 🎮 How to Launch the Unity Backend
1. Double-click **`Run_Unity_Backend.bat`** to start the local REST API server at `http://localhost:5001`.
2. Drag and drop **`AIWorkspaceManager.cs`** into your Unity Project's `Assets` folder and attach it to an empty GameObject in your 3D Scene!

---
> Source: [LyuIlBok/modern-korean-mall](https://github.com/LyuIlBok/modern-korean-mall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
