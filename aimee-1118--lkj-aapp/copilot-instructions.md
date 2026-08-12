## lkj-aapp

> **App:** LKJ-AAPP (LKJ Lab Archiving & Project Platform)

# Cursor Rules for LKJ-AAPP

## Project Context

**App:** LKJ-AAPP (LKJ Lab Archiving & Project Platform)
**Stack:** Next.js 14 + shadcn/ui + TanStack Query + Novel + NextAuth.js v5 + Google Sheets/Drive + Bottleneck/Upstash Redis + Vercel
**Stage:** MVP Development
**User Level:** Developer

---

## Directives

1. **Master Plan:** 작업 시작 전 `AGENTS.md`를 읽는다. 현재 Phase와 Task를 확인한다.

2. **Documentation:** 상세 정보는 `agent_docs/`를 참조한다:
   - `tech_stack.md` — 라이브러리, 버전, 설치
   - `code_patterns.md` — 아키텍처, 코드 예제
   - `project_brief.md` — 컨벤션, 품질 게이트
   - `testing.md` — E2E 테스트

3. **Plan-First:** 복잡한 기능은 계획 제시 후 승인을 기다린다.

4. **Incremental Build:** 한 번에 하나의 기능만 구현한다.

5. **Verify:** 변경 후 `npm run build && npm run lint` 실행. 실패 시 수정 먼저.

6. **Concise:** 서문 없이 핵심만. 에러 시 사과 대신 수정.

---

## Primary Role: Detail & UI Agent

Cursor는 **시각적 컨텍스트 기반 세밀한 작업**에 최적화되어 있다:

- **UI 컴포넌트 개발:** shadcn/ui 기반 컴포넌트 작성 및 스타일링
- **비즈니스 로직:** `lib/` 내 서비스, 훅, 유틸리티 구현
- **파일 내 수정:** 특정 함수나 컴포넌트의 세밀한 변경
- **에디터 통합:** Novel/Tiptap 확장 및 커스터마이징

---

## File Patterns

**컴포넌트:** `components/[category]/[ComponentName].tsx`
**훅:** `lib/hooks/use[Name].ts`
**서비스:** `lib/services/[domain].ts`
**타입:** `types/[domain].ts`
**API Route:** `app/api/[resource]/route.ts`

---

## Code Style

```typescript
// 컴포넌트: 함수 선언문 + named export
export function PostCard({ post }: PostCardProps) {
  return (/* JSX */);
}

// 훅: 함수 선언문 + named export
export function usePosts() {
  return useQuery({/* config */});
}

// 타입: interface 선호 (확장 가능)
interface Post {
  id: string;
  title: string;
}

// Zod 스키마: camelCase + Schema 접미사
const createPostSchema = z.object({/*...*/});
```

---

## Import Order

```typescript
// 1. React/Next.js
import { useState } from 'react';
import { useRouter } from 'next/navigation';

// 2. Third-party
import { useQuery } from '@tanstack/react-query';

// 3. Internal - lib
import { cn } from '@/lib/utils/cn';

// 4. Internal - components
import { Button } from '@/components/ui/button';

// 5. Internal - types
import type { Post } from '@/types/post';
```

---

## Commands

```bash
npm run dev          # 개발 서버
npm run build        # 빌드 (타입 체크)
npm run lint         # ESLint
npm run test:e2e     # Playwright
```

---

## What NOT To Do

- `any` 타입 사용 금지 → `unknown` + 타입 가드
- Route Handler에서 비즈니스 로직 금지 → `lib/services/`로
- 현재 Phase 외 기능 구현 금지
- `npm run build` 없이 커밋 금지
- 컴포넌트에서 직접 `fetch` 호출 금지 → `lib/hooks/`로

---

## Composer Mode Tips

**UI 변경:** "이 컴포넌트를 shadcn Card로 감싸고 버튼을 오른쪽으로"
**로직 추가:** "이 파일에 5분 캐시 만료 로직 추가"
**리팩터링:** "이 함수를 별도 훅으로 분리"

---

## Context Awareness

Cursor는 열린 파일들의 관계를 인식한다. 관련 파일을 함께 열어두면 더 정확한 응답을 받을 수 있다:

- `types/post.ts` + `lib/hooks/usePosts.ts` + `components/dashboard/PostList.tsx`
- `lib/google/sheets.ts` + `app/api/posts/route.ts`

---
> Source: [Aimee-1118/lkj-aapp](https://github.com/Aimee-1118/lkj-aapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
