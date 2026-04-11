## game-civilization2

> I have created the **GEMINI.md** file based on the technical details and repository structure found in the sources. You can copy the markdown content below to add it to your project.

I have created the **GEMINI.md** file based on the technical details and repository structure found in the sources. You can copy the markdown content below to add it to your project.

***

# GEMINI.md - Civilization II Repository Documentation

This document provides essential technical information for maintaining and running this version of **Civilization II**, including scenario management and modern compatibility fixes.

## 🚀 Getting Started

### Recommended Launcher
To ensure modern enhancements and bug fixes are active, do not use the standard `CIV2.EXE`.
*   **Launcher:** Run the game via **`Civ2UIALauncher.exe`**.
*   **Requirements:** This launcher must be in the same directory as `CIV2.EXE` and `Civ2UIA.dll` to successfully inject the **User Interface Add-on (UIA)** into the game process.

### Video & Cinematic Fixes
Modern Windows systems often crash during intro movies or High Council segments due to outdated "Video for Windows" (VfW) codecs.
*   **The Fix:** Locate **`Vfwfix.reg`** in the root directory.
*   **Action:** Double-click or right-click and select **Merge** to update your registry. This maps legacy codecs to modern equivalents.

---

## 📂 Repository Organization
This repository follows a specific structure to distinguish between core game files and development/test assets:
*   **`_bin` / `_test`**: Contains binaries and testing utilities.
*   **`_docs`**: Project documentation, including `INSTALL.LOG`.
*   **`PEDIA`**: Core Civilopedia documentation files.
*   **`SOUND` / `Video`**: Multimedia assets for music, sound effects, and cinematics.
*   **File Naming:** Non-standard folders use a **leading underscore** for easy identification.

---

## 🗺️ Scenario Management (Fantastic Worlds)
When working with or playing scenarios from *Fantastic Worlds*, strict directory rules apply:
1.  **Directory Integrity:** Scenario files **MUST** stay in their specific sub-directories (e.g., `scenario/Mars/`). Moving files will break the game.
2.  **Save Files:** When a scenario is saved, the `.sav` file is automatically placed in that scenario's directory, not the main game folder.
3.  **Loading:** From the game menu, select **'Begin Scenario'**, then navigate into the `scenario` folder and the specific sub-folder for your chosen game.
4.  **Custom Units:** Scenario builders have 8 new unit 'slots' available, with corresponding sound files named `extra1.wav` through `extra8.wav`.

---

## 🎮 Multiplayer & Versioning
This repository supports both **Classic (2.7.8)** and **Multiplayer Gold Edition (MGE 1.3)** logic.
*   **Multiplayer Compatibility:** All players in a session must use **Patch 1.3** to prevent "cheat issues" and game imbalances.
*   **Network Play:** Use the **TCP/IP** option within the Multiplayer Game menu for modern network sessions.
*   **Version Preference:** While MGE supports multiplayer, the Classic 2.7.8 version is often preferred for **scenario creation** because the tools are considered more robust and it does not require a CD to be present.

---

## 🛠️ Technical Environment Note
The project was configured in a **Linux/Wine** environment. If deploying on Linux, ensure the mapping reflects the structure in `INSTALL.LOG`, specifically pointing to `~/.wine/drive_c/MPS/CIV2/` to ensure all `.dll` and `.wav` files are correctly located.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/devsecfranklin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/devsecfranklin)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
