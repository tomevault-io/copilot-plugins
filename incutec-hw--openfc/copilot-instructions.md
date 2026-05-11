## openfc

> Open-source Betaflight flight controller. 30x30mm, 4-layer, 2S-6S. Integrated ELRS receiver (break-off tab).

# OpenFC — Project Instructions

## What This Is
Open-source Betaflight flight controller. 30x30mm, 4-layer, 2S-6S. Integrated ELRS receiver (break-off tab).

## Key ICs
| IC | Part | LCSC | Bus |
|----|------|------|-----|
| MCU | RP2354B (QFN-80, 2MB flash) | C39843328 | — |
| IMU | LSM6DSV16XTR | C5267406 | SPI0 (GPIO18-20), CS=GPIO14, INT=GPIO13 |
| Baro | BMP388 | C779278 | I2C0 (GPIO16/17) |
| Blackbox | BY25Q128ASWIG | C394872 | SPI1 (GPIO42-44), CS=GPIO46 |
| ELRS MCU | ESP32-C3FH4 | C2858491 | UART1 (GPIO21 RX, GPIO22 TX funcsel 11) |
| ELRS Radio | SX1281IMLTRT | C2151551 | SPI to ESP32 |
| 12V Buck | LMR51420YDDCR/YFDDCR | C5383002/C7296200 | — |
| 3.3V LDO | LP5912-3.3DRVR | C524780 | — |
| 1.8V Gyro LDO | NCV8187AMT180TAG | C893189 | — |
| ELRS 3.3V LDO | WL9005D4-33 | C5357206 | — |
| Power Mux | TPS2116DRLR | C3235557 | — |
| OSD Mux | PI3A223ZMEX | — | GPIO32-37 |

## GPIO Map (RP2354B → Betaflight)
- UART0: TX=GPIO0, RX=GPIO1
- UART1 (ELRS): TX=GPIO22 (funcsel 11 UART_AUX), RX=GPIO21
- PIO UART0: TX=GPIO2, RX=GPIO3
- PIO UART1: TX=GPIO26, RX=GPIO27
- Motors: M1=GPIO31, M2=GPIO30, M3=GPIO29, M4=GPIO28 (PIO0 DShot)
- OSD: VBLACK=GPIO32, SYNC_IN=GPIO33, PIXEL_SEL=GPIO34, VWHITE=GPIO35, COLOR_SEL=GPIO36, SYNCLVL=GPIO37
- ADC: VBAT=GPIO41, CURRENT=GPIO40, RSSI=GPIO45, EXT=GPIO47
- LED strip: GPIO23 (PIO2), Status LED: GPIO12
- 10V enable: GPIO11, Beeper: GPIO6
- ESP control: EN=GPIO24, BOOT=GPIO25

## Power Tree
+BATT → LMR51420 → +12V (switchable via GPIO11) / +5V_BUCK (always on)
+5V_BUCK + VBUS → TPS2116 mux → +5V → LP5912 → +3.3V → RP2354B VREG → +1.1V
+5V → WL9005D4 → +3.3V_ELRS | +5V → NCV8187 → +1.8V_GYRO

## PIO Allocation
PIO0: DShot (motors), PIO1: PIO UARTs, PIO2: LED strip + OSD

## Critical Open Issues (pre-production review 2026-03-01)
1. **CRIT-1**: SX1281 VDD_IN disconnected — no power to radio
2. **CRIT-2**: Motor pin numbering M1-M4 reversed vs BF config
3. **CRIT-3**: OSD needs 3 consecutive pins for FB_OSD (remap GPIO33=W, 34=EN, 35=SYNC)
4. **CRIT-4**: No battery reverse polarity protection
5. **CRIT-5**: ELRS LDO EN tied to own output (bootstrap problem)

## Schematic Structure
`hardware/` — KiCad 9 hierarchical: root `OpenFC.kicad_sch` + 12 sub-sheets:
rp2350a, elrs, power, pads, connectors, osd, Radioschematic, leds, imu, baro, blackbox, compass (DNP)

## Rules
- **NEVER** raw-edit `.kicad_sch`, `.kicad_pcb`, `.kicad_pro` — use Python scripts with kicad-skip or pcbnew API
- Python tools live in `hardware/tools/`
- Libraries are project-local: `hardware/lib.kicad_sym`, `hardware/lib.pretty/`, `hardware/lib.3dshapes/`
- Production exports in `hardware/production/` — use KiCad Fabrication Toolkit for JLCPCB
- JLCPCB assembly, prefer LCSC basic parts
- Full review in `OpenFC_preproduction_review.md`

## Firmware
- Target: Betaflight RP2350B (PICO platform)
- No firmware in this repo yet
- RP2354B uses Pico SDK (C/C++), PlatformIO
- ESP32-C3FH4 runs ExpressLRS firmware (separate build)

## README Discrepancies (fix these)
- README says BMI388 → actual is BMP388
- README says W25Q128JVPIQTR → actual is BY25Q128ASWIG
- 10V rail is actually ~12.1V per feedback divider

---
> Source: [incutec-hw/OpenFC](https://github.com/incutec-hw/OpenFC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
