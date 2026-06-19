## lumi-diary

> Check for milestone dates matching today. 检查今日里程碑。


# 🧚 Lumi Diary

**Your local-first memory guardian and cyber bestie.**

> *Lumi isn't a cold cloud drive or a mechanical habit tracker. She's a tiny spirit living on your device who speaks your squad's slang, drops memes from months ago at the perfect moment, and stitches everyone's messy moments into a stunning memory scroll.*
>
> *Lumi 不是一个冷冰冰的网盘，也不是机械的打卡助手。她是一个住在你设备里、懂你们圈子黑话、会接梗，还能把日常碎片拼成灿烂画卷的赛博精灵。*

---

## ✨ Features

### 🔀 Three-Context Architecture

| Mode | Trigger | Lumi's Role |
|------|---------|-------------|
| **👤 Solo** | 1-on-1 chat | Personal assistant & warm confidant |
| **🫂 Circle** | Long-running group chat | Low-key historian & keepsake curator |
| **🚩 Event** | "Start the trip!" | Hype photographer & vibe commander |

### 🧩 Core Capabilities

- **Annotation Stitching** — Multiple perspectives on the same moment, linked and rendered as flip cards
- **Portraits System** — Remembers names, tracks milestones (birthdays, anniversaries), evolving impressions
- **Content-Addressed Media** — Images, videos, and audio stored with Git-style hash sharding (zero duplicates)
- **Fragment CRUD** — Search, view, update, or delete any recorded fragment through conversation
- **Keepsakes** — Archives legendary moments for lethal callbacks months later
- **Time Echoes** — Proactively reminds you of milestones and generates exclusive canvases

### 🎨 Canvas & Export

- **Interactive HTML Scroll** — Star-trail timeline, flip cards, keepsakes gallery, 10 vibe themes
- **Capsule Export** — ZIP-based `.lumi` capsule with real media files for full portability
- **Social Sharing** — Export as vertical long PNG + `.lumi` capsule + HTML
- **Multi-Language** — Full EN/ZH support for all rendered output

### 🔒 Privacy

- **100% local** — All data stays in `Lumi_Vault/` on your device
- **No cloud, no telemetry** — Lumi never phones home

---

## 📂 Vault Structure

```
Lumi_Vault/
├── 👤 Solo/
│   ├── Daily/          # Monthly journals (YYYY-MM.md)
│   └── Projects/       # Serious material (ProjectName.md)
├── 🫂 Circles/         # Group archives (GroupName_YYYY-MM.md)
├── 🚩 Events/          # Trip/event scrolls (YYYY-MM-EventName.md)
├── 📁 Assets/
│   └── <xx>/           # Git-style 2-char hash sharding (a1/a1b2c3...jpg)
└── 🧠 Brain/
    ├── Portraits.json        # Owner + contacts + milestones + impressions
    ├── fragment_index.json   # Searchable fragment index
    ├── Keepsakes.json        # Legendary moments archive
    └── exports/              # .lumi capsules + PNG screenshots
```

---

## 🛠 Tools Summary

| Tool | Purpose |
|------|---------|
| `record_group_fragment` | Record a life fragment with auto-routing |
| `manage_identity` | Owner setup, contact registration, rename |
| `manage_event` | Start / stop / query event scrolls |
| `update_portrait` | Record traits, impressions, milestones |
| `save_keepsake` | Archive moments for future callbacks |
| `render_lumi_canvas` | Generate interactive HTML scroll |
| `manage_fragment` | Search / view / update / delete fragments |
| `export_capsule` | Export .lumi ZIP capsule + PNG + HTML |
| `import_capsule` | Import and merge external .lumi capsule |
| `check_time_echoes` | Detect milestone dates for proactive reminders |

---

## 🚀 Quick Start

**Solo mode** — just chat:
> "Good morning! Need to finish the competitive analysis today."

**Circle mode** — Lumi captures group highlights when invited to a group:
> Jake: "Just made the most insane breakfast burrito"
> Emily: "Bro that's just eggs in a tortilla"

**Event mode** — start a trip scroll:
> "Lumi, start the Joshua Tree Trip!"

**Export** — share the memory:
> "Lumi, export the Joshua Tree scroll as a capsule!"

---
> Source: [Thhoho/lumi-diary](https://github.com/Thhoho/lumi-diary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
