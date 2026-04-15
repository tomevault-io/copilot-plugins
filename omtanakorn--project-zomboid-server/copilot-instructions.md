## project-zomboid-server

> This project provides a robust, Dockerized environment for running a **Project Zomboid Build 42 Unstable** dedicated server. It is optimized for performance using host networking and tuned JVM arguments (ZGC).

# GEMINI.md - Project Zomboid Build 42 Server

## Project Overview
This project provides a robust, Dockerized environment for running a **Project Zomboid Build 42 Unstable** dedicated server. It is optimized for performance using host networking and tuned JVM arguments (ZGC).

### Main Technologies
- **Docker & Docker Compose**: For containerization and orchestration.
- **SteamCMD**: Utilized via `cm2network/steamcmd` to manage game binaries.
- **Project Zomboid (App ID 380870)**: Specifically targeting the `unstable` beta branch for Build 42.

### Architecture
- **Networking**: `host` mode to minimize NAT overhead and improve multiplayer latency.
- **Persistence**: Local directory mapping for game data (`pz-data/`) and server configuration (`pz-config/`).
- **JVM Optimization**: Configured with 12GB heap size and G1 Garbage Collector (G1GC) with tuned parameters for low-latency performance (`-Xms12g -Xmx12g -XX:+UseG1GC -XX:MaxGCPauseMillis=50 -XX:G1HeapRegionSize=32M -XX:+ParallelRefProcEnabled -XX:ParallelGCThreads=8`).

---

## Building and Running

### Initial Setup
1. **Prepare Directories**:
   ```bash
   mkdir -p pz-data pz-config
   sudo chown -R 1000:1000 .
   ```
2. **Initial Download**:
   ```bash
   docker compose up
   ```
   *Wait for `*** SERVER STARTED ****`, then `Ctrl+C` and `docker compose down`.*

### Production Boot
- **Start Detached**: `docker compose up -d`
- **Monitor Logs**: `docker compose logs -f pzserver`
- **Stop Server**: `docker compose down`

---

## Key Files and Configuration

- **`docker-compose.yml`**: Primary service definition. Uses `network_mode: host` and maps volumes. Resources are tuned with 12GB memory reservation and 16GB limit.
- **`custom-start.sh`**: Custom entrypoint script for automatic updates and optimized JVM (G1GC) startup parameters.
- **`pz-data/ProjectZomboid64.json`**: Original JVM configuration (Note: Overridden by `custom-start.sh` arguments).
- **`pz-config/Server/b42coop.ini`**: Game-level settings (MaxPlayers, Mods, WorkshopItems, Maps).
- **`.gitignore`**: Configured to exclude large game binaries and local server data from version control.

---

## 🗺️ Directory & File Map (AI Reference)

เพื่อให้ AI หรือผู้ดูแลระบบเข้าถึงไฟล์ได้อย่างถูกต้องในโครงสร้าง Build 42 นี้:

### 1. Server Configurations (`pz-config/Server/`)
- **Main Settings**: `b42coop.ini` (จัดการ Mods, WorkshopItems, Admin Password)
- **Sandbox Settings**: `b42coop_SandboxVars.lua` (จัดการกฎเกณฑ์โลก, XP, Point ตัวละคร)
- **Spawn Regions**: `b42coop_spawnregions.lua` (จุดเกิดของผู้เล่น)

### 2. Game Data & Mods (`pz-data/`)
- **Workshop Content**: `pz-data/steamapps/workshop/content/108600/` (ไฟล์ Mod ที่โหลดมา)
- **Vanilla Lua Scripts**: `pz-data/media/lua/` (ไฟล์ระบบพื้นฐานของเกม)
- **Java Binaries**: `pz-data/java/`

### 3. Runtime Info (`pz-config/`)
- **Logs**: `pz-config/Logs/` (ตรวจสอบ Error และสถานะเซิร์ฟเวอร์)
- **Saves**: `pz-config/Saves/Multiplayer/b42coop/` (ข้อมูลตัวละครและโลก)
- **Database**: `pz-config/db/` (ข้อมูลผู้เล่นและ White-list)

---

## 🛠️ Build 42 Configuration Notes

- **XP Multipliers**: อยู่ใน `b42coop_SandboxVars.lua` ภายใต้เทเบิล `MultiplierConfig = { ... }`
- **Free Points**: ใช้ตัวแปร `CharacterFreePoints` (ไม่ใช่ `FreeTraitPoints`)
- **Workshop App ID**: `108600`
- **Clothing Logic**: หากเกิดบัคเสื้อผ้าหลุด ให้ตรวจสอบ `ISClothingExtraAction.lua` ในระบบพื้นฐาน

---

## How to Add Workshop Items and Mods

ในการเพิ่ม Mod ใหม่ๆ ให้กับเซิร์ฟเวอร์ ให้แก้ไขไฟล์ `pz-config/Server/b42coop.ini` โดยทำตามขั้นตอนดังนี้:

1. **ระบุ Workshop ID และ Mod ID**: ค้นหา ID จากหน้า Steam Workshop ของ Mod นั้นๆ
2. **แก้ไข `WorkshopItems=`**: เพิ่ม Workshop ID ต่อท้ายรายการเดิม โดยคั่นด้วยเครื่องหมายอัฒภาค (`;`)
   - ตัวอย่าง: `WorkshopItems=2256623447;3171167894;[WORKSHOP_ID_ใหม่]`
3. **แก้ไข `Mods=`**: เพิ่ม Mod ID ต่อท้ายรายการเดิม โดยคั่นด้วยเครื่องหมายอัฒภาค (`;`)
   - ตัวอย่าง: `Mods=firearmmod;damnlib;[MOD_ID_ใหม่]`
4. **Restart Server**: เพื่อให้เซิร์ฟเวอร์ทำการดาวน์โหลดและติดตั้ง Mod ใหม่
   ```bash
   docker compose restart pzserver
   ```
*หมายเหตุ: ต้องตรวจสอบให้แน่ใจว่า Workshop ID และ Mod ID มีการจับคู่กันอย่างถูกต้องและครบถ้วน*

---

## Troubleshooting: Accessing Ignored Files

สาเหตุที่คุณอาจเห็นว่า Gemini (หรือเครื่องมืออื่นๆ) "เข้าถึงไม่ได้" หรือ "มองไม่เห็น" ไฟล์บางอย่าง เช่น `pz-config/` หรือ `pz-data/` เป็นเพราะ:

1. **Git Ignore Protection**: ไฟล์และโฟลเดอร์เหล่านี้ถูกระบุไว้ใน `.gitignore` เพื่อไม่ให้ไฟล์ขนาดใหญ่หรือข้อมูลส่วนตัวถูกอัปโหลดขึ้น Git
2. **Default Tool Behavior**: เครื่องมือค้นหาไฟล์ส่วนใหญ่ (รวมถึงเครื่องมือที่ Gemini ใช้) จะข้ามไฟล์ที่อยู่ใน `.gitignore` โดยอัตโนมัติเพื่อลดความซ้ำซ้อนและเพิ่มประสิทธิภาพ
3. **การเข้าถึง**: หากต้องการให้ Gemini อ่านไฟล์ที่ถูก Ignore สามารถระบุให้ใช้ `run_shell_command` (เช่น `cat` หรือ `grep`) เพื่อเข้าถึงไฟล์เหล่านั้นได้โดยตรงโดยข้ามระบบ Git protection ครับ

---

## Development Conventions

- **Surgical Edits**: When modifying `b42coop.ini`, ensure `WorkshopItems` and `Mods` are synchronized to prevent startup errors.
- **Build 42 Focus**: The current setup is strictly for Build 42. Switching to Build 41 requires changing the beta branch and potentially re-generating configuration files.
---

## Interaction Guidelines
- **Language**: Always respond and provide explanations in **Thai** (ภาษาไทย) as the primary language, unless technical terms or specific code examples require English.
- **Role**: Senior Software Engineer assisting with Project Zomboid server management.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OmTanakorn)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/OmTanakorn)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
