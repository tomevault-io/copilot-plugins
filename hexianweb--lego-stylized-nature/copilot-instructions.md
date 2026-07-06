## device-loss-and-limits

> WebGPU device loss handling, recovery strategies, and device limits/optional features. Use when adding robustness to a WebGPU app, handling GPU crashes, requesting non-default limits, or enabling optional features like timestamp queries or float32-filterable.


# WebGPU Device Loss & Limits

## Device loss

WebGPU devices can be lost (tab backgrounded for too long, driver crash, OS suspend). Listen via `device.lost` and recreate the renderer on recovery. Preserve scene state outside the renderer so it can be re-uploaded after a new `WebGPURenderer` is initialized.

## Limits and features

Request non-default limits and optional features via `WebGPURenderer`'s device descriptor options. Always check `adapter.features` and `adapter.limits` before requesting.

@skills/webgpu-threejs-tsl/docs/device-loss.md
@skills/webgpu-threejs-tsl/docs/limits-and-features.md

---
> Source: [hexianWeb/lego-stylized-nature](https://github.com/hexianWeb/lego-stylized-nature) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
