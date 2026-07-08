## 16-houdini-tools

> Houdini/Solaris tools — structure, UI, LOP rules, dev sync


# Houdini Tools (MonoFX)

## Structure

- **Entry**: `tools/fx/<tool>/` — `config.py`, `logic.py` (no `hou`), `ui.py` (PySide6 MONOS), `controller.py`
- **DCC layer**: `apps/houdini/` — chỉ layer này import `hou`
- **Shared LOP helpers**: `apps/houdini/lop_usd_helpers.py` — cấu hình Sublayer/Reference, không log qua `anim_publish_loader`
- **Shelf**: thêm tool vào `toolbar/MonoFX.shelf` + icon `config/Icons/<name>.svg`

## UI

- **Không** dùng `hou.ui.selectFile` / `selectFromList` / `displayMessage` cho tool chính — dùng PySide6 theo `04_ui_standard.md`
- `config.WINDOW_TITLE` → `ui.setWindowTitle(...)`
- Controller: `importlib.reload` adapter + ui (+ module `apps/houdini/*` khi dev) trước khi chạy

## Solaris / LOP

- LOP node **không có** `isRenderFlagSet()` / `setRenderFlag()` — chỉ copy **display flag**
- Batch load USD (`usd_batch_loader`):
  - **Không** wire / chain / merge — mỗi LOP độc lập trong network editor đang mở
  - Tên node = stem file sau **Strip name** (comma-separated tokens, xóa substring)
  - Layout: hàng ngang, căn giữa `visibleBounds()`
- Tool generic **không** import `apps.houdini.hda.anim_publish_loader` (chỉ dùng cho HDA anim publish)

## Dev sync (bắt buộc khi test shelf)

Houdini shelf thường load từ `Documents/houdini21.0/monofx/`, **không** tự lấy repo dev.

Sau khi sửa code, sync file liên quan sang:
`%USERPROFILE%\Documents\houdini21.0\monofx\`

Hoặc set `MONOFX_SUITE` / package trỏ thẳng repo. Restart Houdini hoặc xóa module cache + `importlib.reload` trước khi test.

## Anti-patterns

- Logic thuần trong `apps/houdini/` thay vì `tools/fx/*/logic.py`
- Warning/log prefix `[MonoFX anim_publish_loader]` từ tool không liên quan anim publish
- Tạo merge LOP khi user chỉ cần chain sublayer/reference

---
> Source: [simplekile/MonoFXSuite](https://github.com/simplekile/MonoFXSuite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
