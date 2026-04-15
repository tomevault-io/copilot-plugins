## aletheia-api

> **Aletheia Edge** is a high-performance, standalone biometric facial verification and document enrollment engine written in **C++**. It is optimized for Brazilian identity documents (CNH and RG) and designed to run as a single, ultra-fast binary without external dependencies like RabbitMQ or PostgreSQL.

# GEMINI.md - Aletheia Edge API Context (C++)

## Project Overview
**Aletheia Edge** is a high-performance, standalone biometric facial verification and document enrollment engine written in **C++**. It is optimized for Brazilian identity documents (CNH and RG) and designed to run as a single, ultra-fast binary without external dependencies like RabbitMQ or PostgreSQL.

## Core Architecture
The system follows a synchronous, monolithic edge architecture:
1.  **Embedded HTTP Server (`cpp-httplib`)**: Handles incoming multipart/form-data requests for verification and enrollment.
2.  **Inference Engine**:
    - **Document Classifier (ONNX)**: Validates if the document is a legitimate Brazilian ID (MobileNetV3).
    - **Biometric Matching (InspireFace SDK)**: Performs 1:1 face matching between selfies and documents.
3.  **Local Biometric Search (HNSWLib)**: Performs 1:N deduplication to ensure a user is not already enrolled.
4.  **Local Audit (SQLite3)**: Stores request metadata, identifiers (CPF/RG), and audit trails in a local `.db` file.
5.  **Local Storage**: Saves encrypted/obfuscated audit images on the filesystem.

## Technical Stack
- **Language**: C++17
- **Inference**: ONNX Runtime & InspireFace SDK
- **Image Processing**: OpenCV
- **Vector Search**: HNSWLib (In-memory index)
- **Database**: SQLite3
- **Web**: `cpp-httplib` (Header-only)
- **JSON**: `nlohmann/json`

## Project Structure
- `app/src/`: Core C++ source code (Engine, Database, Biometrics, API).
- `app/tests/`: Unit tests using Catch2.
- `app/third_party/`: Integrated libraries (HNSWLib, httplib, JSON, InspireFace SDK).
- `models/`: Pre-trained ONNX models and InspireFace resource files.
- `Examples/`: Sample images for testing and verification.
- `setup.sh`: Automated script for installing dependencies and building the project.

## Building and Running

### Prerequisites
- GCC 9+ or Clang (C++17)
- CMake 3.10+
- `wget` and `unzip` for the setup script.

### Automated Setup & Build
```bash
bash setup.sh
```

### Manual Build
```bash
cd app
mkdir build && cd build
cmake ..
make -j$(nproc)
```

### Running the API
```bash
cd app/build
./aletheia_edge
```

## Development Conventions
- **Synchronous Logic**: Unlike the old Python version, the C++ engine is fully synchronous for maximum performance and lower complexity.
- **Request Tracking**: Every request generates a `request_id` logged in the SQLite audit table.
- **Biometric Privacy**: Face embeddings are handled locally; no biometric data leaves the host system.
- **Code Style**: Standard C++17 conventions; use CMake for build management.
- **Testing**: All new features must be accompanied by a Catch2 test case in `app/tests/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IvelOt)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/IvelOt)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
