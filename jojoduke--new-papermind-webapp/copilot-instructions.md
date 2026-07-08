## new-papermind-webapp

> **Papermind** is an AI-native study and exam prep web app that personalizes learning through intelligent quiz generation.

# Papermind

**Papermind** is an AI-native study and exam prep web app that personalizes learning through intelligent quiz generation.

---

## 🧠 Core Idea
Exam prep and studying should feel personalized, not generic.  
Papermind acts like your own study companion — a friendly AI named **Paige**, a pink fox mascot who helps you understand, revise, and prepare smarter.

---

## 🎯 MVP Scope
**Goal:** Build a focused quiz-based study experience for students and professionals.

### Features
1. **Upload materials**
   - Users can upload PDFs, notes, or study guides.
   - File content is used to generate quizzes.

2. **Generate personalized quizzes**
   - AI (via Mastra) creates questions based on uploads or selected exam types (SAT, WAEC, GRE, University finals, CFA).
   - Quizzes adapt based on performance.

3. **Track progress**
   - Stores quiz history and weak topics in Convex.
   - Can regenerate targeted questions later.

---

## ⚙️ Tech Stack
| Layer | Tool | Purpose |
|-------|------|----------|
| Frontend | **Next.js + Tailwind CSS** | Web app UI |
| Backend / DB / Auth | **Convex** | Data storage, auth, and server functions |
| AI Framework | **Mastra** | Handles quiz generation logic |
| File Storage | **Cloudflare R2** | Stores uploaded PDFs and notes |
| Hosting | **Vercel** | Deployment and hosting |

---

## 🗄️ Storage Flow
- Files (PDFs, notes) → stored in **R2**
- Convex stores only **metadata**:  
  ```json
  {
    "ownerId": "user_123",
    "filename": "chapter1.pdf",
    "r2Key": "user_123/chapter1.pdf",
    "r2Url": "https://r2-bucket-link/user_123/chapter1.pdf"
  }

---
> Source: [JojoDuke/new-papermind-webapp](https://github.com/JojoDuke/new-papermind-webapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
