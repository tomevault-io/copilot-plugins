## opensonic-recorder-andriod

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains a new Android USB audio recorder application inspired by decompiled BatRecorder app analysis. The project focuses on 384kHz high-frequency USB audio recording and real-time visualization, specifically designed for ultrasonic and bat call recording applications.

## Project Structure

- `app/` - Main Android application
  - `src/main/java/com/example/usbaudio/` - Core application source code
  - `src/main/res/` - Android resources and layouts
  - `src/main/cpp/` - Native C++ code for USB and audio processing
- `release/` - Built APK files for distribution
- `tmp/BatRecorder/` - Reference decompiled application (analysis only)
  - `sources/` - Decompiled Java source files (3481+ files)  
  - `resources/` - Original app resources and native libraries
  - `BatRecorder.apk` - Original reference APK

### Core Architecture

**Main Package: `com.example.usbaudio`**

**Key Components:**
- `MainActivity.java` - Main activity with USB device detection and audio recording
- `UsbAudioService.java` - Background service for USB audio processing
- `SpectrogramView.java` - Custom view for real-time frequency spectrum visualization (0-200kHz)
- `WaveformView.java` - Real-time waveform display
- `UsbIsochronousReader.java` - USB isochronous transfer handling for audio data

**Custom Views (Optimized for 384kHz):**
- `SpectrogramView.java` - Frequency spectrum display (100 columns × 128 frequency bins, 0-192kHz range)
- `WaveformView.java` - Real-time waveform visualization 
- Audio visualization components support high-frequency ultrasonic signals

**Native Libraries:**
- Native USB audio processing (C++ integration via JNI)
- High-performance FFT processing (planned upgrade from current O(N²) to O(N log N))
- Isochronous USB transfer handling

## Key Technical Features

### USB Audio Support
- Android USB Host mode support (API 26+ minimum)
- 384kHz high-frequency audio device compatibility
- Real-time USB audio descriptor parsing
- Automatic device detection and configuration

### Audio Processing Pipeline
1. USB audio data acquisition via libusb
2. Real-time FFT processing for spectrum analysis
3. Trigger-based recording with configurable thresholds
4. Multi-threaded audio buffer management
5. Live waveform and spectrogram rendering

### Native Method Integration
Core native methods declared in MainActivity:
- `ReadFrameFromAudioBuffer()` - Real-time audio data reading
- `StartRecord()` / `StopRecord()` - Recording session management
- `analyzeSpectrum()` - Frequency domain analysis
- Various buffer and cache management methods

## Development Context

This is a new implementation inspired by reverse-engineering analysis, focused on:
- High-frequency USB audio recording (384kHz sampling rate)
- Real-time ultrasonic signal visualization and analysis
- Efficient audio processing for bat call recording applications
- Modern Android USB Host API integration
- Performance optimization for continuous high-sample-rate recording

## Current Development Status & TODOs

**Completed Features:**
- ✅ Basic USB audio recording infrastructure
- ✅ SpectrogramView with 384kHz support (0-192kHz frequency range)
- ✅ Real-time audio visualization components
- ✅ Android USB Host mode integration
- ✅ Release APK build system

**Pending Improvements:**
- 🔄 High-performance FFT library integration (RenderScript/KissFFT/FFTW)
- 🔄 USB isochronous transfer optimization
- 🔄 Audio buffer management enhancement
- 🔄 Recording file format optimization (WAV/FLAC)
- 🔄 Trigger-based recording functionality
- 🔄 Power spectrum analysis features

## Android Configuration

**Minimum Requirements:**
- Target SDK: 34 (Android 14)
- Minimum SDK: 26 (Android 8.0)

**Key Permissions:**
- `android.permission.RECORD_AUDIO`
- `android.hardware.usb.host` (USB Host mode)
- Location services for metadata
- External storage access

**USB Device Handling:**
- Intent filter for `USB_DEVICE_ATTACHED`
- Dynamic USB device permission management
- Multi-architecture native library support (arm64-v8a, armeabi-v7a, x86, x86_64)

## Development Notes

Since this is decompiled code:
- Variable names are often obfuscated (e.g., `f8702X`, `R0`, etc.)
- Some logic may be unclear due to decompilation artifacts
- Focus on understanding architectural patterns rather than exact implementation details
- Native library functionality is preserved but source code is not available

The codebase demonstrates advanced techniques for:
- Real-time USB audio processing on Android
- Custom view components for audio visualization  
- Multi-threaded audio buffer management
- Integration of native C/C++ libraries with Android Java layer

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zmk5566) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
