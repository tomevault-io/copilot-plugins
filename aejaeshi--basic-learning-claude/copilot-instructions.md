## basic-learning-claude

> ไฟล์นี้คือ context หลักของโปรเจกต์ ใช้บอก Claude ในทุก session ใหม่ว่าโปรเจกต์นี้คืออะไร

# Coin Exchange Machine Project

ไฟล์นี้คือ context หลักของโปรเจกต์ ใช้บอก Claude ในทุก session ใหม่ว่าโปรเจกต์นี้คืออะไร

> **ภาษา:** ตอบเป็นภาษาไทยเป็นหลัก แต่ keyword/code/protocol ใช้ภาษาอังกฤษ

---

## 🎯 เป้าหมายโปรเจกต์

สร้าง **ตู้แลกเหรียญอัตโนมัติ** สำหรับใช้ในร้านเกม
- รับธนบัตร 20 / 50 / 100 / 500 / 1000 บาท
- ปล่อยเหรียญ 10 บาท ตามอัตรา **1 บาท = 1 เหรียญ 10 บาท** (mapping 1:10)
- แจ้งเตือนผ่าน **Discord webhook** (เหรียญใกล้หมด, เครื่องผิดปกติ, สรุปยอดรายวัน)
- ตั้งค่าจาก Web Admin: enable/disable แบงค์, set ยอดเหรียญในตู้, ดู transaction log
- **อนาคต**: รองรับ QR PromptPay (วาง architecture เผื่อไว้แล้ว)

---

## 🏗️ Architecture (สรุปสั้น)

```
[ลูกค้า] → [จอ 18" PC] ← Chromium Kiosk
              ↓
         [PC Lubuntu] ← Node.js + SQLite + Express
              ↓ USB Serial /dev/ttyUSB0 @ 115200
         [ESP32 DevKit] ← I/O controller เท่านั้น
              ↓
   [NK77] [Hopper 24V] [LG-JT02 sensor]
```

**แบ่งหน้าที่ชัด:**
- **PC** = สมอง (UI, logic, log, Discord, future QR payment)
- **ESP32** = มือ-ตา (อ่าน pulse, ขับ relay, ส่ง JSON ขึ้น PC ผ่าน USB)

ละเอียดดู [docs/03-architecture.md](docs/03-architecture.md)

---

## 🔧 Hardware ที่ใช้

| อุปกรณ์ | รุ่น | แรงดัน |
|---|---|---|
| Bill Acceptor | NK77 | 12V, pulse output (1 pulse = 10 บาท) |
| Hopper | 24V Coin Hopper ทั่วไป | 24V, ~1.5–2A peak |
| Coin Sensor | LG-JT02 | 12–24V, photoelectric (NPN/PNP — รอ confirm) |
| MCU | ESP32 DevKit V1 (38-pin) — โมดูล **WROOM-32E** (ECO V3 silicon) | 3.3V |
| PC | Core 2 Duo / Pentium, RAM ≤ 2GB | 100–240V AC |
| Display | จอ 18" ธรรมดา (ไม่มี touch) | HDMI/VGA |
| OS | Lubuntu 22.04 LXQt | — |

⚠️ **สัญญาณทุกเส้นเข้า ESP32 ต้องผ่าน optocoupler (PC817)** เพราะ NK77/LG-JT02 ใช้ 12–24V ESP32 รับ 3.3V

รายละเอียด BOM และ wiring → [docs/02-hardware.md](docs/02-hardware.md)

---

## 📡 Protocol สั้นๆ

PC ↔ ESP32 คุยกันผ่าน **JSON ทีละบรรทัด** (newline-delimited) บน USB Serial baud 115200

```json
// ESP32 → PC
{"type":"event","name":"bill_pulse","amount":10}
{"type":"event","name":"coin_dispensed","count":1}
{"type":"event","name":"heartbeat","uptime":12345}

// PC → ESP32
{"type":"cmd","name":"dispense","coins":10}
{"type":"cmd","name":"inhibit","denoms":[1000]}
```

Full spec → [docs/04-protocol.md](docs/04-protocol.md)

---

## 📁 โครงสร้างไฟล์ที่จะมี

```
D:\Basic_Learning_Claude\
├── CLAUDE.md                ← ไฟล์นี้
├── SKILL.md                 ← วิธีที่ Claude ควรช่วย
├── docs/
│   ├── 01-overview.md       ← ภาพรวมระบบ
│   ├── 02-hardware.md       ← BOM + wiring + แรงดัน
│   ├── 03-architecture.md   ← software architecture
│   ├── 04-protocol.md       ← JSON serial protocol
│   ├── 05-workflow.md       ← state machine การแลก
│   └── 06-safety.md         ← safety, watchdog, error handling
├── firmware/                ← (ยังไม่สร้าง) ESP32 code
│   └── src/
└── server/                  ← (ยังไม่สร้าง) PC Node.js code
    ├── src/
    └── public/              ← UI ลูกค้า + admin
```

---

## ✅ Decisions ที่ตกลงแล้ว

- เหรียญที่ปล่อย: **10 บาทเท่านั้น** (mapping 1 pulse = 1 เหรียญ)
- Bill denominations: 20, 50, 100, 500, 1000
- การแจ้งเตือน: **Discord webhook** (ไม่ใช่ LINE — LINE Notify ปิดบริการ 31 มี.ค. 2025)
- ESP32 board: **DevKit V1 (38-pin) + โมดูล WROOM-32E** (Espressif official, ECO V3 silicon — เสถียร 24/7 ดีกว่า WROOM-32S)
- การสื่อสาร PC↔ESP32: **USB Serial** (ไม่ใช่ WiFi)
- จอ: **18" ไม่มี touch** → UI เป็น display-only ลูกค้าไม่ต้องกด
- DB: **SQLite** (ไฟล์เดียว ไม่ต้องตั้ง server)
- Frontend stack: **HTML + Tailwind CSS + Vanilla JS** (ไม่ใช้ React — PC เก่ารับไม่ไหว)
- Backend stack: **Node.js + Express + serialport**

---

## ❓ ที่ยังต้อง confirm

- [ ] LG-JT02 เป็น NPN หรือ PNP output? (กระทบวงจร pull-up/down)
- [ ] ต้องการปุ่มกายภาพ (Reset / Manual dispense / Admin key) ไหม?
- [ ] ต้องการ UPS สำหรับ PC ไหม? (กันไฟดับกลาง transaction)
- [ ] Discord webhook URL ใครเป็นคนทำ?
- [ ] ติดตั้งจริงที่ไหน? (มี WiFi ไหม, มีกุญแจล็อกตู้ยังไง)

---

## 🚨 Rules สำคัญสำหรับ Claude

1. **เตือนทันทีถ้าเห็นความเสี่ยง** — ไฟฟ้า, ความปลอดภัย, security, ข้อผิดพลาดที่จะทำให้พัง
2. **อย่าให้ ESP32 ต่อตรงกับสัญญาณ 12V/24V** — ต้องผ่าน optocoupler เสมอ
3. **อย่าให้ ESP32 ขับ Hopper 24V โดยตรง** — ต้องผ่าน MOSFET/Relay + flyback diode
4. **ทุก transaction ต้อง log ก่อน dispense** — เผื่อไฟดับจะได้รู้ว่าค้างเหรียญเท่าไหร่
5. **ถ้า PC ↔ ESP32 ขาดการสื่อสาร > 5 วินาที** → ESP32 ต้อง inhibit NK77 และหยุด hopper ทันที
6. **อธิบายโค้ดให้เข้าใจ** — เจ้าของโปรเจกต์อยากเรียนรู้ไปด้วย
7. **ห้ามใช้ `delay()` blocking ใน firmware** — ใช้ `millis()` non-blocking pattern

---
> Source: [Aejaeshi/Basic_Learning_Claude](https://github.com/Aejaeshi/Basic_Learning_Claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
