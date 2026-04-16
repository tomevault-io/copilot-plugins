## tuxtests

> This document defines the interface paradigm between TuxTests raw hardware data and the Google Gemini 3.1 Pro engine.

# Gemini Hardware-to-AI Pipeline

This document defines the interface paradigm between TuxTests raw hardware data and the Google Gemini 3.1 Pro engine.

## 1. Context Assembly Schema

Hardware scan data (from `sysinfo`, `udev`, `lsblk`, `smartctl`) and retrieved logs are collected into a unified JSON blob before transmittal to the AI. This structure is strictly enforced by the Rust `TuxPayload` model, utilizing `BTreeMap` for benchmarks to ensure deterministic, alphabetized serialization for the LLM:

```json
{
  "system": {
    "cpu": "AMD Ryzen 9 5900X",
    "ram_gb": 64
  },
  "drives": [
    {
      "name": "nvme0n1",
      "type": "NVMe",
      "capacity_gb": 2048,
      "usage_percent": 92,
      "health_ok": true,
      "connection": "PCIe 4.0 x4",
      "physical_path": "/devices/pci0000:00/0000:00:02.4/0000:03:00.0/nvme/nvme0/nvme0n1"
    },
    {
      "name": "sda",
      "type": "HDD",
      "capacity_gb": 4096,
      "usage_percent": 15,
      "health_ok": true,
      "connection": "USB 2.0 (High-Speed)",
      "physical_path": "/devices/pci0000:00/0000:00:14.0/usb2/2-1"
    }
  ],
  "benchmarks": {
    "nvme0n1": {"write_mb_s": 5400},
    "sda": {"write_mb_s": 35}
  },
  "kernel_anomalies": [
    "usb 2-1: reset high-speed USB device number 3",
    "blk_update_request: I/O error, dev sdb, sector 1234"
  ]
}
```

## 2. Agent Prompts

TuxTests uses the following system prompt injected ahead of the data payload:

> "You are an expert Linux diagnostics agent. Analyze the provided JSON representing a Linux machine's hardware layout. Identify specific bottlenecks (e.g., drives at >90% capacity, high-speed SSDs bottlenecked by physical USB 2.0 connections) and provide 3 concrete, actionable upgrade or mitigation suggestions. Format output strictly in Markdown."

## 3. Invocation Strategy

`src/ai/gemini.rs` handles standard POST invocation against the `v1beta` Google AI endpoint. It strictly carries a **60-second timeout** and structures the JSON payload to include `system_instruction` alongside the hardware context.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/StarTuz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
