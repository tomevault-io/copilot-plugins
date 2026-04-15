## pnt-presentation

> **By: Bharath (23BCAR0252)**

# NavIC Powered Publish-Subscribe Network for Road Accident Alert

**By: Bharath (23BCAR0252)**
**Date: 01-11-2025**

---

## The Problem: A Critical Gap in Road Safety

*   **High Fatality Rate:** Over 150,000 deaths annually on Indian roads.
*   **Delayed Emergency Response:** Slow response times are a major contributor to preventable deaths.
*   **The Connectivity Barrier:**
    *   Emergency systems rely on cellular networks.
    *   These networks are unreliable in rural and remote areas.
    *   Accidents can damage communication infrastructure, rendering systems useless.
*   **Accessibility Issue:** Advanced safety systems are expensive and limited to high-end vehicles.

---

## The Solution: A Connected Emergency Alert Network

We are developing a cost-effective, connected system that:
1.  **Autonomously Detects Crashes:** Using on-board sensors with threshold-based detection algorithms.
2.  **Connects via Mobile Device:** The crash detection device communicates with the driver's mobile phone via Wi-Fi or Bluetooth.
3.  **Publishes Alert via MQTT:** The mobile phone acts as a publisher, sending crash alerts and NavIC location data to an MQTT broker.
4.  **Distributes to Emergency Services:** The broker forwards alerts to subscribers (police, hospitals, rescue services) for immediate response.

**Theme:** Building a scalable, publish-subscribe emergency response network using indigenous NavIC positioning.

---

## System Architecture

*[This slide would ideally be a diagram showing the MQTT publish-subscribe network flow.]*

**Crash Detection Device (In Vehicle):**
*   **Sensors:** MPU6050 (Motion) & SW-420 (Vibration)
*   **Processing:** ESP32 Microcontroller with threshold-based crash detection
*   **Communication:** Wi-Fi/Bluetooth to mobile device

**Mobile Device (Publisher):**
*   **Positioning:** NavIC GPS for precise location
*   **Role:** Acts as MQTT publisher
*   **Function:** Publishes crash alerts with location data to broker

**MQTT Broker (Central Hub):**
*   **Role:** Message distribution center
*   **Function:** Receives crash alerts from publishers and forwards to subscribers

**Emergency Services (Subscribers):**
*   **Police Stations**
*   **Hospitals & Ambulances**
*   **Rescue Services**
*   **Role:** Receive real-time crash alerts with precise location data

---

## Key Technologies

*   **NAVIC (Navigation with Indian Constellation):**
    *   **Why:** Indigenous, reliable, and accurate positioning. Reduces dependency on foreign systems.
    *   **Role:** Provides precise GPS coordinates integrated with mobile device for location accuracy.

*   **MQTT (Message Queuing Telemetry Transport) Protocol:**
    *   **Why:** Lightweight, reliable publish-subscribe messaging protocol ideal for IoT applications.
    *   **Role:** Enables scalable communication between crash detection systems and emergency services.

*   **Publish-Subscribe Architecture:**
    *   **Why:** Decoupled, scalable system where publishers and subscribers operate independently.
    *   **Role:** Allows multiple emergency services to receive alerts simultaneously without direct connections.

*   **Threshold-Based Detection Algorithm:**
    *   **Why:** Simple, reliable, and efficient crash detection using mathematical threshold comparisons.
    *   **Role:** Analyzes sensor data on the ESP32 to distinguish crashes from normal driving events using predefined threshold values.

---

## The Prototype: RC Car MQTT Network Demonstration

*   **Why an RC Car?**
    *   Safe and controlled testing environment for crash simulation.
    *   Cost-effective platform for demonstrating the complete MQTT network.
    *   Allows testing of various crash scenarios and network responses.

*   **Three-Computer Demonstration Setup:**
    *   **Computer 1:** Acts as the crash detection device and mobile publisher
    *   **Computer 2:** Runs the MQTT broker (message distribution center)
    *   **Computer 3:** Simulates emergency service subscribers (police, hospital, rescue)

*   **Components:**
    *   RC car with integrated ESP32, sensors for crash detection
    *   MQTT implementation for publish-subscribe communication
    *   NavIC positioning integration for precise location data

---

## Core Objectives & Deliverables

**Primary Goals:**
*   Autonomous Crash Detection using Threshold-Based Algorithms
*   NAVIC Integration for precise positioning
*   MQTT-based Publish-Subscribe Network Implementation
*   Scalable Emergency Response System
*   Real-time alert distribution to multiple emergency services

**Key Deliverables:**
*   Functional RC car prototype with crash detection
*   Complete MQTT broker and client implementation
*   Three-computer demonstration of the publish-subscribe network
*   Threshold-based crash detection algorithm implementation
*   NavIC integration with mobile device positioning
*   Performance analysis and technical documentation

---

## Challenges & Our Approach

*   **Challenge:** Sensor noise and avoiding false positives in crash detection.
    *   **Solution:** Multi-sensor approach with threshold-based algorithms tuned to recognize genuine crash patterns vs. normal driving events.

*   **Challenge:** Reliable mobile device connectivity.
    *   **Solution:** Dual connectivity options (Wi-Fi/Bluetooth) between crash detection device and mobile phone.

*   **Challenge:** Network scalability and message reliability.
    *   **Solution:** MQTT's built-in Quality of Service (QoS) levels ensure reliable message delivery to all subscribers.

*   **Challenge:** Scaling from RC car prototype to real vehicle implementation.
    *   **Solution:** Focus on physics-based crash detection algorithms that are scale-independent and platform-agnostic.

---

## Conclusion & Next Steps

This project addresses the critical need for a scalable, connected emergency alert system that can rapidly distribute accident information to multiple emergency services simultaneously.

**Current Status:**
*   Actively developing MQTT protocol implementation for broker and client communication.
*   Three-computer demonstration setup being prepared for network testing.
*   RC car prototype integration planned upon hardware arrival.

**Next Steps:**
1.  Complete MQTT codebase development and testing.
2.  Integrate hardware components with RC car platform.
3.  Conduct extensive testing of crash detection and data collection.
4.  Demonstrate full publish-subscribe network with three-computer setup.
5.  Validate NavIC positioning accuracy and MQTT message reliability.

---

## Thank You & Q/A

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/8harath)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/8harath)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
