## nfcproxy

> **NFCProxy** simplifies NFC passport reading. It handles platform-specific NFC interactions, enabling efficient client-server integration where cryptographic and protocol processing occurs on a secure backend server (**NFCServer**).

# NFCProxy Integration Guide

**NFCProxy** simplifies NFC passport reading. It handles platform-specific NFC interactions, enabling efficient client-server integration where cryptographic and protocol processing occurs on a secure backend server (**NFCServer**).

NFCServer is proprietary and non-open-source. Communication between NFCProxy and NFCServer is secured via WebSockets using a patent-pending protocol.

MRZ (Machine Readable Zone) scanning is required.  It's done using the bundled **MRZProxy** (iOS) or a custom component conforming to the respective platform's protocol.

NFCProxy complies with the **ICAO 9303 standard** for MRTDs, ensuring adherence to global passport and identity document verification protocols.

---

## 1. Introduction

NFCProxy facilitates:

*   **NFC Passport Scanning:** Uses platform-specific NFC APIs.
*   **Secure Client-Server Communication:** Secure data exchange with NFCServer.
*   **Simplified API:** Developer-friendly interface.
*   **Flexible MRZ Scanning:** Supports bundled or custom MRZ components.
*   **ICAO 9303 Compliance:** Adheres to international MRTD standards.

NFCProxy supports **Terminal Authentication** for accessing protected biometric data:

*   **DG3:** Fingerprints
*   **DG4:** Iris data

---

## 2. Installation and Configuration

### 2.1 iOS

#### 2.1.1 Swift Package Manager (SPM)

1.  **File → Add Packages...** in Xcode.
2.  Add repositories:
    *   **NFCProxy:** `https://github.com/iproov/NFCProxy.git`
    *   **MRZProxy:** `https://github.com/iproov/MRZProxy.git`
3.  Select version rule.
4.  **Add Package**.

#### 2.1.2 Package.swift

```swift
dependencies: [
  .package(url: "[https://github.com/iproov/NFCProxy.git](https://github.com/iproov/NFCProxy.git)", from: "1.0.0"), // Replace with actual version
  .package(url: "[https://github.com/iproov/MRZProxy.git](https://github.com/iproov/MRZProxy.git)", from: "1.0.0")  // Replace with actual version
]

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/advatar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
