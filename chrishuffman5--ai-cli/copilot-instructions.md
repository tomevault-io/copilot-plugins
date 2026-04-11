## ai-cli

> This guide provides step-by-step instructions for installing the Gemini CLI on Windows, macOS, and Linux. To ensure full compatibility with all skills and tools, we recommend installing Node.js, Python 3.12, and PowerShell 7.6+.

# Installing Gemini CLI

This guide provides step-by-step instructions for installing the Gemini CLI on Windows, macOS, and Linux. To ensure full compatibility with all skills and tools, we recommend installing Node.js, Python 3.12, and PowerShell 7.6+.

---

## 1. Prerequisites

Before installing the Gemini CLI, ensure your system meets the following software requirements:

### **Node.js (LTS Recommended)**
*   **Version:** 20.0.0 or higher.
*   **Installation:** [nodejs.org](https://nodejs.org/)

### **Python 3.12**
*   **Version:** 3.12.x
*   **Purpose:** Required for many advanced skills and local data processing tools.

### **PowerShell 7.6+**
*   **Version:** 7.6 or higher.
*   **Purpose:** The preferred shell for cross-platform automation and advanced CLI features.

---

## 2. OS-Specific Setup

### **Windows**
The most efficient way to install prerequisites on Windows is via **winget**.

1.  **Install Node.js, Python, and PowerShell:**
    ```powershell
    # Install Node.js LTS
    winget install OpenJS.NodeJS.LTS

    # Install Python 3.12
    winget install Python.Python.3.12

    # Install PowerShell 7.6
    winget install Microsoft.PowerShell
    ```
2.  **Restart your terminal** (or use `refreshenv` if using Chocolatey) to update path variables.

### **macOS**
The recommended method for macOS is using **Homebrew**.

1.  **Install Node.js, Python, and PowerShell:**
    ```bash
    # Install Node.js
    brew install node

    # Install Python 3.12
    brew install python@3.12

    # Install PowerShell
    brew install --cask powershell
    ```
2.  **Verify path:** Ensure `/usr/local/bin` (Intel) or `/opt/homebrew/bin` (Apple Silicon) is in your `$PATH`.

### **Linux (Ubuntu/Debian)**
Use the native package manager and Microsoft's repository for PowerShell.

1.  **Install Node.js and Python:**
    ```bash
    sudo apt update
    sudo apt install -y nodejs npm python3.12
    ```
2.  **Install PowerShell:**
    ```bash
    # Download and register the Microsoft repository
    wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb
    sudo dpkg -i packages-microsoft-prod.deb

    # Install PowerShell
    sudo apt update
    sudo apt install -y powershell
    ```

---

## 3. Installing Gemini CLI

Once your prerequisites are installed, you can install the Gemini CLI globally using npm.

### **Global Installation**
```bash
npm install -g @google/gemini-cli
```

### **Preview/Beta Version (Optional)**
If you want to access the latest experimental features:
```bash
npm install -g @google/gemini-cli@preview
```

---

## 4. Initialization & Verification

After installation, initialize the CLI to set up your authentication and preferences.

1.  **Run the CLI:**
    ```bash
    gemini
    ```
2.  **Authenticate:** Follow the browser-based login flow to connect your Google account.
3.  **Verify Versions:**
    Run these commands to ensure everything is correctly configured:
    *   **Gemini CLI:** `gemini --version`
    *   **Node.js:** `node --version` (should be v20+)
    *   **Python:** `python3 --version` (should be v3.12+)
    *   **PowerShell:** `pwsh --version` (should be v7.6+)

---

## Troubleshooting

*   **Permissions:** If you encounter `EACCES` errors during global npm installation, consider using a version manager like `nvm` or `fnm`, or run the command with `sudo` (Linux/macOS only).
*   **Path Issues:** If the `gemini` command is not found after installation, ensure your npm global binaries directory is in your system's `PATH`.
    *   Check with: `npm config get prefix`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chrishuffman5)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chrishuffman5)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
