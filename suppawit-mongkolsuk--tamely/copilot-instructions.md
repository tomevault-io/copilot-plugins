## tamely

> **Tamely** — แอปพลิเคชัน Team Collaboration ที่รวม Chat, Feed, Calendar, และ AI ไว้ในที่เดียว

# CLAUDE.md — Tamely Project Root

## โปรเจกต์คืออะไร

**Tamely** — แอปพลิเคชัน Team Collaboration ที่รวม Chat, Feed, Calendar, และ AI ไว้ในที่เดียว  
เป็น Mini Project (Semester Project) สำหรับวิชาเรียน พัฒนาโดยทีม 2 คน

**คล้ายกับ:** Slack + Facebook Feed + Google Calendar + AI Summarizer

---

## โครงสร้าง Repository

```
Tamely/
├── Backend/
│   └── prisma-api/          ← Express API + Prisma + Socket.IO
├── Frontend_Web/            ← React Web App (Vite + Tailwind)
├── Frontend_mobile/         ← React Native Mobile App (Expo)
└── CLAUDE.md                ← ไฟล์นี้
```

แต่ละส่วนมี CLAUDE.md ของตัวเอง:
- `Backend/prisma-api/CLAUDE.md`
- `Frontend_Web/CLAUDE.md`
- `Frontend_mobile/CLAUDE.md`

---

## Tech Stack Overview

| Layer           | Technology                                    |
|-----------------|-----------------------------------------------|
| Backend API     | Node.js + Express 5 + TypeScript              |
| Database        | PostgreSQL (Supabase) + Prisma ORM            |
| Real-time       | Socket.IO 4                                   |
| File Storage    | Supabase Storage                              |
| Auth            | JWT (cookie) + Passport (Google/GitHub OAuth) |
| Email           | Resend                                        |
| AI              | OpenAI API                                    |
| Frontend Web    | React 18 + Vite + Tailwind v4 + shadcn/ui     |
| Frontend Mobile | React Native + Expo SDK 54 + NativeWind       |

---

## Features ที่ทำแล้ว (ณ ปัจจุบัน)

- [x] Auth — Register, Login, Logout, OAuth (Google/GitHub), Forgot/Reset Password
- [x] Workspace — สร้าง/จัดการ workspace, invite via code, roles (OWNER/ADMIN/MODERATOR/MEMBER)
- [x] Room/Channel — สร้างห้อง public/private, manage members
- [x] Real-time Chat — ส่งข้อความ, typing indicator, online status (Socket.IO)
- [x] Direct Message (DM) — chat 1-1 พร้อม file/image upload
- [x] Feed/Post — โพสต์ประกาศ, comment, pin post, @ mention + notification
- [x] Notification — แจ้งเตือน @user และ @role
- [x] Task/Calendar — สร้าง task กับปฏิทิน, AI สร้าง task ได้
- [x] AI Chat — ถาม-ตอบ AI และ AI สรุปบทสนทนา (OpenAI)

---

## Git Branches

- `main` — stable / production
- `Dev` — integration branch (merge feature branches ที่นี่ก่อน)
- `DevTle` — feature branch ของ Tle
- `DevFolk` — feature branch ของ Folk

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Suppawit-Mongkolsuk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
