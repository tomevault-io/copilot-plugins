## frams

> **Purpose:** Provide explicit instructions, schemas, workflows, and design guidelines for agentic AI systems interacting with the FRAMS codebase.

# **FRAMS — Agentic Context PRD**

**Purpose:** Provide explicit instructions, schemas, workflows, and design guidelines for agentic AI systems interacting with the FRAMS codebase.
**Scope:** Generate, modify, or read code only within defined project boundaries. Do not assume external behaviors or APIs beyond explicitly allowed services.

---

## **1. Project Overview**

**FRAMS** is a full-stack, browser-based **face recognition attendance system**.

**Core Entities:**

* Students, Teachers, Admins
* Classes, Attendance Sessions
* Student Faces

**Tech Stack:**

* Frontend: Next.js 16 (App Router) + TypeScript
* Database: Neon DB + Drizzle ORM
* Auth: BetterAuth (Teacher/Admin)
* Image Storage: Cloudinary
* ML: TensorFlow.js (Face Recognition)
* UI: shadcn/ui + Radix + TailwindCSS v4
* Forms: react-hook-form + zod
* Animations: Framer Motion

---

## **2. Agent Rules**

### **2.1 Allowed Operations**

| Operation        | Notes                                                                            |
| ---------------- | -------------------------------------------------------------------------------- |
| Read/Write Code  | Only in `app/`, `components/`, `db/`, `lib/`, `actions/`, `handlers/`, `hooks/`. |
| Language         | TypeScript only.                                                                 |
| Auth & DB        | Must respect BetterAuth, Drizzle constraints, and RLS.                           |
| Coding Style     | Follow Section 5 conventions.                                                    |
| Input Validation | Use zod schemas for all DB operations.                                           |
| Public APIs      | Only Cloudinary & TensorFlow.js allowed.                                         |

### **2.2 Forbidden Operations**

| Forbidden Action                               | Reason             |
| ---------------------------------------------- | ------------------ |
| Remove authentication checks                   | Security violation |
| Access external APIs beyond allowed            | Security & privacy |
| Bypass DB constraints or RLS                   | Data integrity     |
| Console logging in production                  | Clean code policy  |
| Modify Drizzle schema without explicit request | Schema stability   |

---

## **3. Folder Responsibilities**

| Folder        | Purpose                                                   |
| ------------- | --------------------------------------------------------- |
| `app/`        | Pages, routes, layouts (`/student`, `/teacher`, `/admin`) |
| `components/` | Reusable UI components                                    |
| `actions/`    | Server Actions for authenticated Teacher/Admin operations |
| `handlers/`   | Public API Route Handlers (e.g., face upload)             |
| `hooks/`      | Custom React hooks (e.g., `useWebcamCapture`)             |
| `lib/`        | Utilities, Cloudinary & TensorFlow helpers, constants     |
| `db/`         | Drizzle schema definitions, Neon DB connection            |
| `public/`     | Static assets (images, fonts, ML models)                  |

---

## **4. Database Schema**

*All tables use `snake_case` in Neon DB.*

### **4.1 Users**

```ts
table users {
  id: string, // BetterAuth ID
  email: string,
  role: 'student' | 'teacher' | 'admin',
  student_id: string | null, // unique student ID
  created_at: Date,
  updated_at: Date,
}
```

### **4.2 Classes**

```ts
table classes {
  id: string,
  name: string,
  teacher_id: string, // FK to users
  start_time: string,
  end_time: string,
  created_at: Date,
  updated_at: Date,
}
```

### **4.3 Sessions**

```ts
table sessions {
  id: string,
  class_id: string, // FK to classes
  session_date: Date,
  created_at: Date,
  updated_at: Date,
}
```

### **4.4 Student Faces**

```ts
table student_faces {
  id: string,
  student_id: string, // FK to users
  image_url: string, // Cloudinary URL
  created_at: Date,
  updated_at: Date,
}
```

### **4.5 Attendance**

```ts
table attendance {
  id: string,
  student_id: string,
  class_id: string,
  session_id: string,
  status: 'marked' | 'missed',
  marked_at: Date,
  created_at: Date,
  updated_at: Date,
}
```

---

## **5. Coding Conventions**

* **Files:** hyphen-case (e.g., `mark-attendance.ts`)
* **Components:** PascalCase
* **Server Actions / Functions:** camelCase
* **Imports:** `@/*` for root paths
* **Styling:** Tailwind + clsx + tailwind-merge (from `lib/utils.ts`)
* **Forms:** Controlled via `react-hook-form` + zod validation

---

## **6. Server Action Guidelines**

* Must be used for Teacher/Admin authenticated actions (e.g., `createClass`)
* Must start with `"use server"` directive
* Input: validated with zod
* Output: `{ success: boolean; data?: T; error?: string }`
* Security: verify user session/role (BetterAuth) and typed DB operations (Drizzle)

---

## **7. Data Flow**

### **7.1 Public Attendance Flow**

1. Student opens link → Browser captures face
2. TensorFlow.js recognizes face → sends to Route Handler
3. Handler validates → updates `attendance` table
4. Returns success/failure (with error codes)

### **7.2 Authenticated Teacher Flow**

1. Teacher submits form → zod validation
2. Server Action (`createClass`, `updateClass`) → Drizzle insert/update
3. Revalidate paths → show success toast

**Note:** Components **never mutate DB directly**, always go through Server Actions or Route Handlers.

---

## **8. Error Handling**

* **Standard Codes:**

  * `INVALID_INPUT`, `UNAUTHORIZED`, `ML_ERROR`, `DB_ERROR`
* Return format: `{ success: false, error: string }`
* Public endpoints must implement **rate-limiting**: e.g., max 5 requests per 10 seconds per IP

---

## **9. Security Constraints**

* Sensitive actions (Teacher/Admin) require BetterAuth session verification
* Teachers can manage only their own classes
* Students cannot access `/teacher` or `/admin` endpoints
* Public endpoints (`/student/face`, `/attendance/[sessionId]`) are rate-limited

---

## **10. UI/UX Guidelines**

### **10.1 Design Principles**

* Clean, minimal, modern
* Use theme tokens (`--background`, `--primary`)
* Typography: Sans-serif
* Webcam view: centered, full-aspect ratio

### **10.2 Component Examples**

| Component | Style                                                                                 |
| --------- | ------------------------------------------------------------------------------------- |
| Button    | `bg-primary text-primary-foreground rounded-xl hover:bg-primary/90 transition-colors` |
| Input     | `border-input bg-background text-foreground rounded-md px-3 py-2 focus:outline-ring`  |
| Card      | `bg-card text-card-foreground rounded-lg p-4 shadow-sm`                               |

### **10.3 Animations (Framer Motion)**

* Page elements: `initial`, `animate`, `exit`
* Interactive: `whileHover`, `whileTap`
* Webcam status indicators must be animated

---

## **11. Example Workflows**

### **11.1 Create Class**

Teacher submits → `createClass` Server Action → Drizzle insert → revalidate → success toast

### **11.2 Mark Attendance**

Student opens link → TensorFlow.js recognizes face → Route Handler updates `attendance` → show success message / block if invalid

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/whilmarbitoco)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/whilmarbitoco)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
