## linuxprintermaster

> LinuxPrinterMaster is a desktop application designed to manage printers on Linux systems. It is built using **Python 3** and **PyQt5**.

# Project Context: LinuxPrinterMaster

## Overview
LinuxPrinterMaster is a desktop application designed to manage printers on Linux systems. It is built using **Python 3** and **PyQt5**.
The application is targeted to run on **Ubuntu 20.04+**, **UOS (UnionTech OS)**, **Kylin V10**, and **CASCO (Zhongke Fangde)**.

## Architecture
The project follows a simple MVC-like structure:
- **`src/main.py`**: Application entry point.
- **`src/ui/`**: Contains PyQt5 UI components.
  - `main_window.py`: The primary application window and interaction logic.
- **`src/utils/`**: Backend logic and system interactions.
  - `scanner.py`: Handles printer discovery using system tools (`lpinfo`, `lsusb`).

## Dependencies
### Python Packages
- `PyQt5`: GUI framework.

### System Dependencies
The application relies on the following Linux system utilities:
- `cups-client` (provides `lpinfo`): Preferred method for printer discovery.
- `usbutils` (provides `lsusb`): Fallback method for detecting connected USB devices.

## Development
- **Current Status**: Initial prototype capable of listing connected printers and showing details.
- **Mock Mode**: If no physical printers are detected (e.g., dev environment), the application generates a "Demo Printer" for UI testing.

## Target Environment Notes
- **Ubuntu 20.04/22.04/24.04**: Standard support.
  - Note: On Ubuntu 22.04, the `python3.10-venv` package is required to create virtual environments: `sudo apt install python3.10-venv`.
- **UOS / Kylin / CASCO**: Ensure generic drivers and `pyqt5` system packages are compatible. Prefer `apt install python3-pyqt5` over pip where possible to match system Qt libraries.

## Deployment

To deploy and run LinuxPrinterMaster on a target Linux system (Ubuntu 20.04+, UOS, Kylin V10, CASCO), follow these steps:

### Prerequisites

1.  **Python 3**: Ensure Python 3 (preferably 3.8+) is installed.
2.  **System Dependencies**: Install necessary system packages for GUI, USB detection, and printer management tools.
    ```bash
    sudo apt update
    sudo apt install python3-pyqt5 python3-pip python3-venv cups-client usbutils
    # For Ubuntu 22.04 specifically:
    sudo apt install python3.10-venv
    ```
    *Note: `python3-pyqt5` is preferred over pip installation where available to ensure better system integration. `cups-client` provides `lpstat`, `lpoptions`, `usbutils` provides `lsusb`, which are crucial for printer detection.*

### Installation

1.  **Copy Application Files**: Transfer the entire `LinuxPrinterMaster` project directory to the target machine (e.g., `/home/user/LinuxPrinterMaster`).
    ```bash
    # Example:
    scp -r /path/to/local/LinuxPrinterMaster user@target_ip:/home/user/
    ```

2.  **Navigate to Project Directory**:
    ```bash
    cd /home/user/LinuxPrinterMaster
    ```

3.  **Set up Python Virtual Environment (Recommended)**:
    This isolates project dependencies from your system Python packages.
    ```bash
    python3 -m venv venv
    source venv/bin/activate
    ```

4.  **Install Python Packages**:
    Install the required Python libraries using pip:
    ```bash
    pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
    ```

### Running the Application

After installation, you can launch the application. The display language will adapt based on your system's `LANG` environment variable.

-   **To run in English (default behavior if `LANG` is not set to Chinese):**
    ```bash
    source venv/bin/activate # If using venv
    LANG=en_US.UTF-8 python3 src/main.py
    ```

-   **To run in Chinese (Simplified):**
    ```bash
    source venv/bin/activate # If using venv
    LANG=zh_CN.UTF-8 python3 src/main.py
    ```
    *Note: You may need to ensure the `zh_CN.UTF-8` locale is generated on your system: `sudo locale-gen zh_CN.UTF-8`.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/purplepalmdash)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/purplepalmdash)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
