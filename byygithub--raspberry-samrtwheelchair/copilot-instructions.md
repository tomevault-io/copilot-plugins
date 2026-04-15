## raspberry-samrtwheelchair

> This project is a **Multi-modal Smart Voice-Controlled Wheelchair System** designed for individuals with mobility impairments. It follows a "Sense-Cloud-Control" tri-layer architecture, splitting high-level AI tasks and low-level real-time control between a Raspberry Pi and an STM32 microcontroller.

# Gemini Project Context: Raspberry Smart Wheelchair

This project is a **Multi-modal Smart Voice-Controlled Wheelchair System** designed for individuals with mobility impairments. It follows a "Sense-Cloud-Control" tri-layer architecture, splitting high-level AI tasks and low-level real-time control between a Raspberry Pi and an STM32 microcontroller.

## 🏗️ System Architecture

### 1. Perception & Intelligence Layer (Raspberry Pi 4B)
- **Voice Control (`vosk_v2.py`)**: Uses the `Vosk` offline engine for speech-to-text. Implements a fuzzy matching algorithm using `pypinyin` to handle accents and dialects (e.g., mapping "钱进" to "前进").
- **Vision Control (`Pi_opencv/`)**: A C++/Qt application running `YOLOv11n` for head pose detection (`up`, `down`, `left`, `right`, `front`). It translates poses into movement commands.
- **Cloud Integration (`onenet_mqtt_final.py`)**: Connects to the China Mobile **OneNET** platform via MQTT for remote monitoring and control.

### 2. Execution & Control Layer (STM32F407)
- **Motor Drive**: Located in `stm32/WheelchairControl/`. It receives UART commands (`F`, `B`, `L`, `R`, `S`, `A`, `D`) and outputs analog voltages via **DAC** to emulate a physical joystick.
- **Manual Override**: The STM32 continuously monitors the physical joystick via **ADC**. If manual movement is detected, it overrides AI/Voice commands for safety.
- **Safety Features**: Includes speed gear management (3 levels) and logic for safe braking.

### 3. User Interaction Layer (uni-app)
- **WeChat Mini-Program (`weixin_app/`)**: Developed with `uni-app`, providing a remote control interface, speed adjustment, and real-time status viewing (Health data, Direction).

---

## 📂 Key Directory Structure

- `/raspberry`: Core intelligence modules.
    - `vosk_v2.py`: Main voice recognition script.
    - `onenet_mqtt_final.py`: IoT cloud communication.
    - `Pi_opencv/`: C++/Qt vision project (requires OpenCV & ONNX Runtime).
    - `weixin_app/`: Source code for the WeChat Mini-program.
- `/stm32/WheelchairControl`: STM32CubeMX + Keil project for the motor controller.
- `/docs`: Documentation, including the `Defense_Guide.md` with technical insights.

---

## 🛠️ Development & Deployment

### Raspberry Pi (Python)
- **Environment**: Python 3.9+
- **Dependencies**: `vosk`, `pypinyin`, `paho-mqtt`, `pyserial`, `pyaudio`.
- **Run**: 
  ```bash
  python3 raspberry/vosk_v2.py
  python3 raspberry/onenet_mqtt_final.py
  ```

### Raspberry Pi (C++/Qt Vision)
- **Environment**: Qt 5.15+, OpenCV 4.8.0.
- **Build**: 
  ```bash
  cd raspberry/Pi_opencv
  qmake Pi_opencv.pro
  make
  sudo ./OpenCV_CameraMonitor
  ```

### STM32 (C)
- **Environment**: Keil uVision5 or STM32CubeIDE.
- **UART Config**: 115200 baud, 8N1. Connected to Raspberry Pi via `USART2`.

---

## 📡 Communication Protocol (UART)

| Character | Command | Description |
|-----------|---------|-------------|
| `F`       | Forward | Move forward |
| `B`       | Backward| Move backward |
| `L`       | Left    | Turn left |
| `R`       | Right   | Turn right |
| `S`       | Stop    | Stop all movement |
| `A`       | Accel   | Increase speed gear |
| `D`       | Decel   | Decrease speed gear |

---

## 💡 Development Conventions
- **Safety First**: Bottom-layer (STM32) manual override always takes precedence over top-layer (Pi) commands.
- **Modular Design**: Keep AI logic (Python/C++) on the Pi and time-critical hardware control on the STM32.
- **Fuzzy Mapping**: When adding new voice commands, update the `command_config` dictionary in `vosk_v2.py` with pypinyin aliases.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/byygithub)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/byygithub)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
