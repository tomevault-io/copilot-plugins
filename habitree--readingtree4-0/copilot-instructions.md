## readingtree4-0

> **프로젝트:** Habitree Reading Hub v4.0.0


## [UI/데이터 접근 레이어 분리 규칙]

# UI/데이터 접근 레이어 분리 규칙

**프로젝트:** Habitree Reading Hub v4.0.0  
**작성일:** 2025년 1월  
**버전:** 1.0  
**적용 범위:** 전체 프로젝트

**참고:** 데이터 모델 관련 규칙은 `.cursor/rules/datarule.mdc`를 참조하세요.

---

## 목차

1. [규칙 개요](#1-규칙-개요)
2. [레이어 정의](#2-레이어-정의)
3. [레이어별 허용/금지 사항](#3-레이어별-허용금지-사항)
4. [데이터 접근 흐름](#4-데이터-접근-흐름)
5. [예외 사항](#5-예외-사항)
6. [규칙 위반 시 조치](#6-규칙-위반-시-조치)
7. [마이그레이션 가이드](#7-마이그레이션-가이드)

---

## 1. 규칙 개요

### 1.1 목적

확장 가능하고 유지보수 가능한 코드베이스를 위해 **엄격한 레이어 분리**를 강제합니다.

### 1.2 핵심 원칙

- ✅ **UI 레이어**: 화면 렌더링 및 사용자 입력 처리만 담당
- ✅ **데이터 레이어**: DB 접근은 `app/actions/`에서만 허용
- ✅ **lib 레이어**: 클라이언트 생성, 순수 유틸, 형식 검증만 허용
- ✅ **app/api**: 서버 라우트이지만 DB 접근 금지 (필요 시 actions 위임)

---

## 2. 레이어 정의

### 2.1 UI 레이어

**범위:**
- `components/**`
- `hooks/**`
- `contexts/**` (인증 제외)
- `app/**` (page, layout, loading, error 등)

**책임:**
- 화면 렌더링
- 사용자 입력 처리
- 로딩/에러/빈 상태 표시
- hooks를 통한 데이터 접근

**금지 사항:**
- ❌ Supabase 쿼리 직접 실행 (`supabase.from(...)`)
- ❌ 테이블명/컬럼명 등 DB 구조 노출
- ❌ 권한/정책 최종 판정 로직

### 2.2 데이터 접근 레이어

**범위:**
- `app/actions/**`

**책임:**
- Supabase 쿼리 실행 (조회/저장/수정/삭제)
- 타입 명시 (`Database` 타입 사용)
- 에러 처리 및 변환

**금지 사항:**
- ❌ UI 렌더링 (`useState`, `useEffect` 등)
- ❌ 브라우저 API 사용

### 2.3 lib 레이어

**범위:**
- `lib/**`

**책임:**
- Supabase 클라이언트 생성기 (`lib/supabase/`)
- 외부 API 래퍼 (`lib/api/`)
- 순수 유틸리티 함수 (`lib/utils/`)
- 형식 검증 (`lib/validations/`)

**금지 사항:**
- ❌ Supabase 쿼리 함수 (`.from()`, `.select()` 등)
- ❌ UI에서 import되는 권한/정책 최종 판정 로직

### 2.4 app/api 레이어

**범위:**
- `app/api/**`

**책임:**
- HTTP 요청/응답 처리
- Rate limiting
- 외부 API 연동 (Gemini, Naver 등)

**금지 사항:**
- ❌ Supabase 쿼리 직접 실행
- ✅ 필요 시 `app/actions/`를 호출하여 위임

**예외:**
- Supabase Storage 접근은 허용 (파일 업로드/다운로드)

---

## 3. 레이어별 허용/금지 사항

### 3.1 UI 레이어 (components, hooks, contexts)

#### ✅ 허용

```typescript
// ✅ 컴포넌트에서 hooks 사용
import { useBooks } from "@/hooks/use-books";

export function BookList() {
  const { books, isLoading, error } = useBooks();
  // ...
}

// ✅ hooks에서 actions 호출
import { getUserBooks } from "@/app/actions/books";

export function useBooks() {
  const [books, setBooks] = useState([]);
  useEffect(() => {
    getUserBooks().then(setBooks);
  }, []);
}
```

#### ❌ 금지

```typescript
// ❌ 컴포넌트에서 Supabase 직접 호출
import { createClient } from "@/lib/supabase/client";

export function BookList() {
  const supabase = createClient();
  const { data } = await supabase.from("user_books").select("*");
  // ...
}

// ❌ UI 레이어에서 DB 구조 노출
const tableName = "user_books"; // 금지!
if (book.user_id === user.id) { ... } // 권한 판단 로직도 금지!
```

### 3.2 데이터 접근 레이어 (app/actions)

#### ✅ 허용

```typescript
// ✅ Server Action에서 Supabase 쿼리 실행
"use server";
import { createServerSupabaseClient } from "@/lib/supabase/server";
import type { Database } from "@/types/database";

export async function getUserBooks() {
  const supabase = await createServerSupabaseClient();
  
  const { data, error } = await supabase
    .from("user_books")
    .select("*")
    .returns<Database["public"]["Tables"]["user_books"]["Row"][]>();
  
  if (error) throw error;
  return data || [];
}
```

#### ❌ 금지

```typescript
// ❌ UI 관련 코드
"use server";
import { useState } from "react"; // 금지!

// ❌ 브라우저 API
window.localStorage.setItem(...); // 금지!
```

### 3.3 lib 레이어

#### ✅ 허용

```typescript
// ✅ Supabase 클라이언트 생성
export function createClient() {
  return createBrowserClient(...);
}

// ✅ 순수 유틸리티
export function formatDate(date: Date): string {
  return date.toISOString();
}

// ✅ 형식 검증
export function isValidEmail(email: string): boolean {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}
```

#### ❌ 금지

```typescript
// ❌ 쿼리 함수
export async function getBooks() {
  const supabase = createClient();
  return await supabase.from("books").select("*"); // 금지!
}
```

### 3.4 app/api 레이어

#### ✅ 허용

```typescript
// ✅ app/actions를 통한 데이터 접근
import { searchNotes } from "@/app/actions/search";

export async function GET(request: NextRequest) {
  const params = new URL(request.url).searchParams;
  const results = await searchNotes(params);
  return NextResponse.json(results);
}

// ✅ Supabase Storage 접근 (예외)
export async function POST(request: NextRequest) {
  const supabase = await createServerSupabaseClient();
  const { data } = await supabase.storage
    .from("images")
    .upload(filePath, file);
}
```

#### ❌ 금지

```typescript
// ❌ Supabase 쿼리 직접 실행
export async function GET(request: NextRequest) {
  const supabase = await createServerSupabaseClient();
  const { data } = await supabase
    .from("notes")
    .select("*"); // 금지! app/actions로 위임해야 함
}
```

---

## 4. 데이터 접근 흐름

### 4.1 표준 패턴

```
컴포넌트 (UI)
  ↓
hooks/ (상태 관리)
  ↓
app/actions/ (Server Actions - DB 접근)
  ↓
Supabase
```

### 4.2 API 라우트 패턴

```
API 요청
  ↓
app/api/** (HTTP 처리)
  ↓
app/actions/ (Server Actions - DB 접근)
  ↓
Supabase
```

---

## 5. 예외 사항

### 5.1 인증 관련

다음은 예외로 허용됩니다:

- ✅ `lib/supabase/client.ts`, `lib/supabase/server.ts` (클라이언트 생성 유틸리티)
- ✅ `contexts/auth-context.tsx` (인증 상태 관리)
- ✅ `app/callback/route.ts` (OAuth 콜백 처리)

### 5.2 Storage 접근

Supabase Storage 접근은 `app/api`에서 허용됩니다:

- ✅ 파일 업로드/다운로드
- ✅ 공개 URL 생성

---

## 6. 규칙 위반 시 조치

### 6.1 발견 시 즉시 수정

규칙 위반 코드를 발견하면:

1. **UI 레이어에서 Supabase 직접 호출**
   → `hooks/`를 통해 간접 호출하도록 수정

2. **app/api에서 Supabase 쿼리 직접 실행**
   → `app/actions/`로 이동하고 API에서 호출하도록 수정

3. **lib에서 쿼리 함수**
   → `app/actions/`로 이동

### 6.2 점진적 마이그레이션

기존 코드는 유지하되, 새로운 코드부터 엄격히 적용합니다.

---

## 7. 마이그레이션 가이드

### 7.1 app/api에서 app/actions로 이동

**Before (위반):**

```typescript
// app/api/search/route.ts
export async function GET(request: NextRequest) {
  const supabase = await createServerSupabaseClient();
  const { data } = await supabase
    .from("notes")
    .select("*");
  return NextResponse.json(data);
}
```

**After (규칙 준수):**

```typescript
// app/actions/search.ts
"use server";
export async function searchNotes(params: URLSearchParams) {
  const supabase = await createServerSupabaseClient();
  const { data } = await supabase
    .from("notes")
    .select("*");
  return data;
}

// app/api/search/route.ts
import { searchNotes } from "@/app/actions/search";

export async function GET(request: NextRequest) {
  const params = new URL(request.url).searchParams;
  const results = await searchNotes(params);
  return NextResponse.json(results);
}
```

### 7.2 체크리스트

새로운 기능 개발 시:

- [ ] UI 레이어에서 Supabase 직접 호출하지 않음
- [ ] app/api에서 DB 접근 시 app/actions 위임
- [ ] lib에서 쿼리 함수 작성하지 않음
- [ ] 모든 DB 접근은 타입 명시

---

**이 규칙은 프로젝트의 레이어 분리 규칙 단일 기준입니다. 모든 개발자는 이 규칙을 준수해야 합니다.**

**상세 문서:** `doc/governance/UI_DATA_ACCESS_RULES.md` 참조

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/habitree)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/habitree)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
