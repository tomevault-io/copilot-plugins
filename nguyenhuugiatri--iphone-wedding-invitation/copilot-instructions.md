## iphone-wedding-invitation

> - Toàn bộ UI và animation phải match iOS / Apple HIG.

# CLAUDE.md

## Design style

- Toàn bộ UI và animation phải match iOS / Apple HIG.
  - Spacing, corner radius, typography, hierarchy: theo iOS lock-screen / Music / Control Center.
  - Glass/blur backgrounds (vibrancy, subtle inner highlight, soft shadow).
  - Animation: easing kiểu iOS (`cubic-bezier(0.4, 0, 0.2, 1)` hoặc spring nhẹ), duration ngắn (150-250ms cho micro-interaction), tránh bounce mạnh.
  - Icon swap = crossfade + scale nhẹ (0.7 → 1.0), không cắt cứng.
  - Tap feedback: scale 0.95-0.98, không ripple kiểu Material.
- Khi chưa rõ, tham chiếu hành vi app gốc của iOS (Music, Mail, Photos, Control Center) thay vì web convention.

---
> Source: [nguyenhuugiatri/iphone-wedding-invitation](https://github.com/nguyenhuugiatri/iphone-wedding-invitation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
