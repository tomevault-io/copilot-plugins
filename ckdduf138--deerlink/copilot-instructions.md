## deerlink

> 이 파일은 Claude Code가 일관된 코드를 생성하기 위한 프로젝트 가이드입니다.

# Deerlink — CLAUDE.md

이 파일은 Claude Code가 일관된 코드를 생성하기 위한 프로젝트 가이드입니다.

---

## 프로젝트 개요

**Deerlink** — 링크 하나로 그룹 의견을 비교하는 플랫폼. 사슴 뿔처럼 여러 가지가 하나의 뿌리에서 만나는 컨셉.
방 생성자가 질문을 만들고 링크를 공유하면, 참여자 전원이 답변 후 결과를 비교한다.
핵심 원칙: **Answer Lock** — 모두가 답변 완료 전까지 결과 비교 불가 (API 레벨 강제).

---

## 기술 스택

| 레이어 | 기술 |
|---|---|
| Framework | Next.js 16 (App Router, Turbopack) |
| Language | TypeScript 5 (strict) |
| Styling | Tailwind CSS v4 |
| UI Components | shadcn/ui (`src/components/ui/`) |
| Animation | Framer Motion |
| Icons | lucide-react |
| ORM | Prisma 7 (`src/generated/prisma`) |
| DB | SQLite (libSQL / Turso) |
| Runtime | Node.js (서버 컴포넌트 + Route Handlers) |

---

## 디렉토리 구조

```
src/
├── app/
│   ├── layout.tsx          # Root layout (dark, Pretendard 폰트)
│   ├── globals.css         # Tailwind + CSS 변수 정의
│   ├── page.tsx            # 랜딩 페이지 (Server Component)
│   ├── create/
│   │   └── page.tsx        # 방 만들기 (Client Component)
│   ├── room/
│   │   └── [id]/
│   │       ├── page.tsx    # 방 입장 + 답변 (Client Component)
│   │       └── results/
│   │           └── page.tsx # 결과 비교 (Client Component)
│   └── api/
│       └── rooms/
│           ├── route.ts            # POST /api/rooms
│           └── [id]/
│               ├── route.ts        # GET /api/rooms/[id]
│               └── answers/
│                   └── route.ts    # POST /api/rooms/[id]/answers
├── components/
│   ├── landing/            # 랜딩 페이지 전용 섹션 컴포넌트
│   └── ui/                 # shadcn/ui 컴포넌트 (수정 금지)
└── lib/
    ├── prisma.ts            # Prisma client 싱글턴
    └── utils.ts             # cn() 유틸리티
```

---

## 디자인 시스템

### 색상 팔레트

```
배경:       #0d0a07   (따뜻한 다크 브라운)
카드 배경:  #17120c   (따뜻한 브라운)
border:     rgba(240,200,130,0.09)  (amber tint)
텍스트 1:   #fdf4e8   (크림 화이트)
텍스트 2:   #a8a29e   (stone-400, 본문)
텍스트 3:   #78716c   (stone-500, 보조)
텍스트 4:   #57534e   (stone-600, 캡션/라벨)
Accent:     #e8a038   (amber-500, 사슴 털 색)
Accent dim: #d4891e   (amber-600)
```

**색상 원칙**
- accent 색상은 **amber 단색**만 사용. 따뜻한 느낌 유지.
- 의미 있는 대조 색상: teal(청록) — amber와 대비 목적으로만 (balance game B 옵션).
- 그라디언트 남용 금지. 필요 시 `from-amber-600 to-amber-400` 정도.

### 타이포그래피

```
폰트:    Pretendard Variable (globals.css에서 --font-sans 정의됨)
영문:    tracking-tight
숫자:    font-mono tabular-nums

히어로:  text-5xl ~ text-[82px]  font-bold  tracking-tight  leading-[1.05]
섹션:    text-3xl ~ text-4xl     font-bold  tracking-tight
카드:    text-sm ~ text-base     font-semibold
본문:    text-sm                 text-stone-500  leading-relaxed
캡션:    text-[10px] ~ text-xs  text-stone-600  uppercase tracking-widest
```

### 컴포넌트 스타일 패턴

```tsx
// 카드 — 기본
className="rounded-2xl border border-white/8 bg-[#17120c] p-6"

// 카드 — 강조 (selected/active)
className="border-amber-500/25 bg-amber-500/8"

// 버튼 — 주요 CTA
className="inline-flex items-center gap-2 px-6 py-3 rounded-xl bg-amber-600 hover:bg-amber-500 text-white text-sm font-medium transition-all duration-200 shadow-lg shadow-amber-900/40 hover:-translate-y-0.5"

// 버튼 — 보조
className="text-sm text-stone-400 hover:text-white transition-colors duration-200"

// Input
className="bg-white/3 border border-white/8 text-white placeholder:text-stone-600 rounded-xl"

// 태그/뱃지
className="px-3 py-1 rounded-full text-xs border border-white/8 bg-white/3 text-stone-400"

// 섹션 구분선
className="h-px bg-white/5"
```

### 아이콘

- **lucide-react** 사용.
- 크기: 카드 내부 `w-4 h-4`, 버튼 내부 `w-3.5 h-3.5`, 대형 `w-5 h-5`.
- **아이콘-인-박스 패턴 금지** (icon을 둥근 배경 box 안에 넣는 것). 인라인으로 사용.

---

## 애니메이션 원칙

Framer Motion 사용. **기능적 애니메이션만** — 장식용 bounce/infinite 남용 금지.

```tsx
// 페이지 진입 — 기본
initial={{ opacity: 0, y: 16 }}
animate={{ opacity: 1, y: 0 }}
transition={{ duration: 0.5, ease: [0.16, 1, 0.3, 1] }}

// 스크롤 트리거 — 기본
initial={{ opacity: 0, y: 12 }}
whileInView={{ opacity: 1, y: 0 }}
viewport={{ once: true, margin: "-60px" }}
transition={{ duration: 0.45, delay: i * 0.08 }}

// 순차 등장 — stagger delay
transition={{ duration: 0.35, delay: index * 0.1 }}

// ease — string 대신 배열 사용 (TypeScript 에러 방지)
ease: [0.16, 1, 0.3, 1]   // spring-like
ease: "easeOut"            // 가능하지만 as const 불필요
```

**viewport={{ once: true }}** — 스크롤 재진입 시 재실행 금지.

---

## 코드 컨벤션

### 컴포넌트

```tsx
// Server Component — "use client" 없음
export default function Page() { ... }
export function SomeSection() { ... }

// Client Component — 최상단에 선언
"use client";
export function InteractiveComponent() { ... }
```

- 상태/이벤트/훅이 필요한 경우만 `"use client"` 추가.
- 랜딩 섹션은 애니메이션 때문에 모두 Client Component.
- 페이지(`page.tsx`)는 가능하면 Server Component 유지.

### 타입 정의

```tsx
// 로컬 타입은 파일 상단 정의
type QuestionType = "balance" | "multiple" | "subjective";

interface Question {
  id: string;
  type: QuestionType;
  title: string;
  optionA?: string;
  optionB?: string;
  options?: string[];
}
```

- `any` 사용 금지. unknown 또는 명시적 타입 사용.
- DB에서 오는 `options` 필드는 `string | null` (JSON.stringify된 배열).

### 유틸리티

```tsx
import { cn } from "@/lib/utils";  // clsx + tailwind-merge
```

조건부 클래스는 항상 `cn()` 사용.

### API Route

```tsx
// app/api/.../route.ts
import { prisma } from "@/lib/prisma";
import { NextResponse } from "next/server";

export async function GET(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  // ...
  return NextResponse.json(data);
}
```

- 에러는 `NextResponse.json({ error: "메시지" }, { status: 4xx })` 형식.
- params는 항상 `Promise<{ id: string }>` + `await params`.

### Prisma

```tsx
// lib/prisma.ts에서 싱글턴 import
import { prisma } from "@/lib/prisma";

// 관계 포함 조회
const room = await prisma.room.findUnique({
  where: { id },
  include: {
    questions: { orderBy: { order: "asc" } },
    participants: { include: { answers: true } },
  },
});
```

---

## DB 스키마 요약

```
Room         id(cuid), title, createdAt
Question     id, roomId, type, title, optionA?, optionB?, options?(JSON), order
Participant  id, roomId, nickname, createdAt
Answer       id, questionId, participantId, value
             unique(questionId, participantId)
```

- `Question.type`: `"balance"` | `"multiple"` | `"subjective"`
- `Question.options`: `JSON.stringify(string[])` — 객관식 선택지
- `Answer.value`: balance → `"A"` | `"B"`, multiple → 인덱스 문자열, subjective → 자유 텍스트

---

## Answer Lock 규칙

결과 페이지(`/room/[id]/results`)는 반드시 API 레벨에서 잠금 확인.

```tsx
// 참여자가 모든 질문에 답변했는지 확인
const totalQuestions = room.questions.length;
const answeredCount = participant.answers.length;
if (answeredCount < totalQuestions) {
  return NextResponse.json({ error: "답변을 완료해주세요" }, { status: 403 });
}
```

---

## 금지 사항

- 이모지 사용 금지 (코드 내, DeerPlaceholder에서만 사용)
- 3D 라이브러리 (`@react-three/fiber` 등) 랜딩 페이지에 사용 금지
- 아이콘-인-박스 UI 패턴 금지
- violet/sky 색상 사용 금지 (대신 amber/teal 사용)
- `any` 타입 금지
- `ease: "easeInOut"` 등 string ease는 `as const` 없이 사용 가능
- 불필요한 `useEffect` 남용 금지 — 서버에서 처리 가능한 건 서버에서
- `// 주석` — 명백한 코드엔 주석 불필요. 비즈니스 로직 의도 설명 시에만.

---

## 개발 명령어

```bash
pnpm dev          # 개발 서버 (Turbopack)
pnpm build        # 프로덕션 빌드
pnpm lint         # ESLint
npx prisma studio # DB 관리 UI
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ckdduf138) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
