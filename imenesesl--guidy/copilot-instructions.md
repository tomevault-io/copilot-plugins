## guidy

> Product context and domain knowledge for the Guidy EdTech platform


# Product Context — Guidy

## What is Guidy

EdTech platform for high school students and teachers. Improves learning during homework and study while giving teachers visibility into student performance.

## Core Problem

- Students complete assignments by copying (ChatGPT, peers) without understanding the material.
- Teachers spend too much time reviewing work and lack visibility into who understands, who copies, and where students struggle.

## Solution

Teachers create or assign tasks that students complete in-platform. Each student receives AI-generated variations of the same task (anti-copying). A guided AI assistant helps students think step by step — hints, not answers. The platform generates automatic analytics for teachers.

## Target Users

| Role | Relationship |
|---|---|
| Teachers / Schools | Primary buyer |
| High school students | End user |

## Key Features

1. **AI-generated task variations** — each student gets a unique version, reducing copying.
2. **Guided AI assistant** — identifies mistakes, gives hints, never direct answers.
3. **Simple task creation** — teachers build assignments with minimal effort.
4. **Student workspace** — solve tasks within the platform.
5. **Teacher dashboard** — common mistakes, performance by topic, who needs help.

## Product Goals

- Force real learning instead of passive copying.
- Save teachers time in grading and analysis.
- Provide actionable insights to improve teaching.
- Make learning feel guided, not overwhelming.

## Constraints

- **Extremely simple and intuitive** — assume low-tech teachers.
- **Immediate value** — no heavy setup or onboarding required.
- **Low-resource environments** — must work on basic hardware and slow connections.

## MVP Scope

1. Teacher creates a task (text-based, single subject).
2. System generates unique variations per student.
3. Student workspace with guided AI hints.
4. Teacher dashboard with per-student and per-topic analytics.

## When building features

- Every UI decision must prioritize simplicity for non-technical users.
- Performance and lightweight bundles are critical (low-resource constraint).
- Two distinct experiences: **teacher-facing** (creation + analytics) and **student-facing** (workspace + guided learning).
- The `landing` app is the public marketing site. The `core` app is the main product.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/imenesesl) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
