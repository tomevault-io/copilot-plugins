## sensecap-indicator-starter

> **Generated:** 2026-04-10

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-04-10  
**Project:** SenseCAP Indicator Starter  
**Stack:** ESP-IDF 5.4+, LVGL 9, ESP32-S3, RP2040 coprocessor

## OVERVIEW
Minimal ESP-IDF starter for SenseCAP Indicator (480×480 touchscreen). Convention over framework: thin app layer with pages/ and services/, not a full OS.

## STRUCTURE
```
.
├── components/
│   ├── bsp/           # Hardware abstraction (display, touch, IO expander)
│   └── cobs/          # COBS framing library for RP2040 UART
├── main/
│   ├── main.c         # Boot sequence
│   ├── lv_port.c      # LVGL 9 integration
│   └── app/           # App layer
│       ├── app_manager.*    # Page container + data routing
│       ├── pages/           # UI pages (LVGL only)
│       └── services/        # Business logic + hardware services
├── sdkconfig.defaults       # PSRAM, BLE, partition configs
└── partitions.csv           # 8MB flash partition layout
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add UI page | `main/app/pages/` | Create `page_*.c`, register in `app_manager.c` |
| Add service | `main/app/services/` | Create `svc_*.c`, call init from `main.c` |
| Hardware config | `components/bsp/include/bsp_indicator.h` | Pin definitions, I2C/UART settings |
| WiFi/BLE provisioning | `main/app/services/svc_wifi.c` | ESP provisioning manager + QR display |
| RP2040 comms | `main/app/services/svc_rp2040.c` | UART + COBS framing |
| Partition layout | `partitions.csv` | 8MB flash: nvs, otadata, app, spiffs |

## ENTRY POINTS

**Boot sequence** (`main/main.c`):
```c
nvs_flash_init()        → NVS partition
bsp_indicator_init()    → I2C, IO expander
lv_port_init()          → LVGL + display + touch
svc_rp2040_init()       → UART to RP2040
svc_storage_init()      → Settings persistence
svc_alerts_init()       → CO2 threshold alerts
app_manager_init()      → Create tileview, register pages
svc_wifi_init()         → WiFi/BLE provisioning
```

## CONVENTIONS

### Page Pattern (main/app/pages/)
```c
// page_*.h
void page_sensor_create(lv_obj_t *parent);
void page_sensor_update(const sensor_data_t *data);  // optional
```
- **Pure LVGL**: No BSP includes
- **Register**: Add entry to `s_pages[]` in `app_manager.c`
- **Data**: Receive via `update()` callback, not direct service calls

### Service Pattern (main/app/services/)
```c
esp_err_t svc_xxx_init(void);
void svc_xxx_register_cb(callback_t cb, void *ctx);
```
- Initialize in `main.c` boot sequence
- Update app state via `app_manager_*()` functions
- Use LVGL locking when touching UI (`lv_port_lock()`)

### BSP Independence
- **Pages NEVER include BSP headers** — only LVGL + app_manager.h
- **Services** can use BSP for hardware access
- **Main.c** wires everything together

### Data Flow
```
RP2040 → UART → svc_rp2040.c → app_manager_update_sensor() → page_update()
WiFi event → svc_wifi.c → app_manager_update_wifi_info() → UI timer update
```

## ANTI-PATTERNS (THIS PROJECT)

| ❌ Don't | ✅ Do Instead |
|----------|---------------|
| Put hardware logic in pages | Use services, pass data via update() |
| Include BSP headers in pages | Use app_manager APIs only |
| Call LVGL functions without lock | Wrap in `lv_port_lock()` / `unlock()` |
| Use esp-brookesia | Use thin app_manager + pages convention |
| Write display/touch drivers | Use esp_lvgl_port, esp_lcd (no custom drivers) |
| Use COBS in BSP | Keep COBS as standalone utility component |

## KEY CONFIGS

**sdkconfig.defaults** critical settings:
```
CONFIG_SPIRAM=y                          # 8MB PSRAM
CONFIG_SPIRAM_FETCH_INSTRUCTIONS=y       # Code in PSRAM
CONFIG_BT_NIMBLE_MEM_ALLOC_MODE_EXTERNAL=y  # BLE heap in PSRAM
CONFIG_BT_NIMBLE_HOST_TASK_STACK_SIZE=5120  # BLE task stack
```

## COMMANDS

```bash
# Build
idf.py set-target esp32s3
idf.py build

# Flash & monitor
idf.py flash monitor

# Full clean rebuild
idf.py fullclean && idf.py build
```

## NOTES

- **Screen**: 480×480 RGB LCD (ST7701S), 16-bit
- **Touch**: FT6336U via I2C
- **RP2040**: UART2 @ 115200, COBS framing
- **Provisioning**: BLE with QR code, POP "sensecap123"
- **PSRAM**: Required for BLE stack (external memory allocation)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Love4yzp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
