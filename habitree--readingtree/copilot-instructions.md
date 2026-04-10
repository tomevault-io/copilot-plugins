## readingtree

> **프로젝트:** Habitree Reading Hub v4.0.0


## [Data Model Governance]

# 프로젝트 데이터 구조 규칙 (Data Model Governance)

**작성일:** 2025년 1월  
**프로젝트:** Habitree Reading Hub v4.0.0  
**버전:** 2.0  
**적용 범위:** 전체 프로젝트

**참고:** 
- 레이어 분리 상세 규칙은 `.cursor/rules/ui_datarule.mdc`를 참조하세요.
- DB/RLS 관리 규칙은 `.cursor/rules/db_rls_rule.mdc`를 참조하세요.
- 마이그레이션 관리 규칙은 `.cursor/rules/migration_rule.mdc`를 참조하세요.

---

## 목차

1. [규칙 개요](#1-규칙-개요)
2. [레이어 분리 원칙](#2-레이어-분리-원칙)
3. [데이터 모델 단일 기준](#3-데이터-모델-단일-기준)
4. [스키마 변경 규칙](#4-스키마-변경-규칙)
5. [데이터 무결성 원칙](#5-데이터-무결성-원칙)
6. [타입 안정성](#6-타입-안정성)
7. [변경 사항 문서화](#7-변경-사항-문서화)
8. [점진적 마이그레이션 전략](#8-점진적-마이그레이션-전략)
9. [필수 폴더 및 파일 구조](#9-필수-폴더-및-파일-구조)
10. [규칙 위반 시 조치](#10-규칙-위반-시-조치)
11. [실제 적용 예시](#11-실제-적용-예시)

---

## 1. 규칙 개요

### 1.1 규칙의 목적

이 프로젝트는 **Next.js + Supabase 기반**이며, **Vercel에 배포**됩니다.  
개발 방식은 **바이브코딩**이므로, 데이터 구조와 DB 접근이 무너지지 않도록 **엄격한 구조 규칙**이 반드시 필요합니다.

### 1.2 규칙의 핵심 원칙

- ✅ **단일 기준 문서**: 모든 데이터 구조는 `doc/database/DATA_MODEL.md`에 정의
- ✅ **레이어 분리**: 컴포넌트는 UI만, DB 접근(쿼리 실행) 은 `app/actions/`에서만
- ✅ **DB 구조 노출 금지**: UI 레이어(컴포넌트)에서 테이블명, 컬럼명 등 DB 구조 노출 금지
- ✅ **타입 안정성**: 모든 DB 접근은 타입을 명시
- ✅ **문서화 필수**: 스키마 변경 시 반드시 문서 업데이트
- ✅ **점진적 적용**: 기존 코드는 유지, 새 코드부터 엄격히 적용

---

## 2. 레이어 분리 원칙

### 2.1 컴포넌트의 책임

컴포넌트(`.tsx`, `.jsx`)는 다음을 **절대 수행하면 안 됩니다**:

```typescript
// ❌ 금지: 컴포넌트에서 Supabase 직접 호출
import { createClient } from "@/lib/supabase/client";

export function BookList() {
  const supabase = createClient();
  const { data } = await supabase.from("user_books").select("*"); // 금지!
  return <div>...</div>;
}

// ❌ 금지: UI 레이어에서 DB 구조 노출 (테이블명, 컬럼명 등)
const tableName = "user_books"; // 금지!
const columnName = "user_id"; // 금지!
if (book.user_id === user.id) { ... } // 권한 판단 로직도 금지!
```

### 2.2 컴포넌트가 해야 하는 것

컴포넌트는 다음만 수행합니다:

- ✅ 화면 렌더링
- ✅ 사용자 입력 처리
- ✅ `hooks/`를 통한 데이터 접근 (표준 패턴)

```typescript
// ✅ 올바른 예: hooks를 통한 간접 호출 (표준 패턴)
import { useBooks } from "@/hooks/use-books";

export function BookList() {
  const { books, isLoading, error } = useBooks();
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <div>
      {books.map(book => (
        <BookCard key={book.id} book={book} />
      ))}
    </div>
  );
}
```

### 2.3 데이터 접근 레이어 구조

데이터 접근은 다음 계층 구조를 따릅니다:

```
컴포넌트 (UI)
  ↓
hooks/ (상태 관리 및 비동기 처리)
  ↓
app/actions/ (Server Actions - DB 접근)
  ↓
Supabase
```

**표준 패턴:**
- 컴포넌트 → `hooks/` → `app/actions/` → Supabase

### 2.4 허용된 데이터 접근 경로

Supabase와의 모든 통신은 **오직 아래 경로에서만** 허용됩니다:

1. **`app/actions/**`** (Next.js Server Actions - **유일한 DB 접근 레이어**)

### 2.5 예외 사항 (인증 관련)

다음은 예외로 허용됩니다:

- ✅ `lib/supabase/client.ts`, `lib/supabase/server.ts` (클라이언트 생성 유틸리티)
- ✅ `contexts/auth-context.tsx` (인증 상태 관리)
- ✅ `app/callback/route.ts` (OAuth 콜백 처리)

```typescript
// ✅ 허용: 인증 컨텍스트에서의 Supabase 호출
// contexts/auth-context.tsx
const supabase = createClient();
const { data: { user } } = await supabase.auth.getUser(); // 허용
```

### 2.6 Server Actions 규칙

`app/actions/` 내의 Server Actions는:

- ✅ Supabase 호출을 직접 수행할 수 있음
- ✅ 테이블명, 컬럼명은 문자열 리터럴로 사용 가능 (Server Actions 내부이므로)
- ✅ 모든 DB 접근 로직은 이 레이어에 집중
- ✅ 권한 검증 및 RLS 정책을 신뢰하되, 필요한 경우 추가 검증 수행
- ✅ 반환 타입은 반드시 명시

```typescript
// ✅ 올바른 예: Server Action에서 타입 명시
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

---

## 3. 데이터 모델 단일 기준

### 3.1 단일 기준 문서

데이터베이스 구조의 **단일 기준 문서**는 하나만 존재해야 합니다:

**`doc/database/DATA_MODEL.md`**

### 3.2 필수 정의 사항

모든 데이터 관련 개념은 반드시 이 문서에 정의되어야 합니다:

- ✅ 테이블
- ✅ 컬럼
- ✅ 테이블 간 관계
- ✅ 소유 구조(사용자/그룹)
- ✅ 접근 제어 원칙(RLS 의도)

### 3.3 테이블 정의 요구사항

어떤 테이블도 다음 정보 없이 존재할 수 없습니다:

1. **한 문장으로 된 목적 정의**
2. **소유자/권한 기준** (auth.users 기준)
3. **관계 정의** (외래 키)

### 3.4 중복 테이블 금지

동일한 개념을 표현하는 테이블이 둘 이상 존재하는 것은 **금지**합니다.

```sql
-- ❌ 금지: 중복 테이블
CREATE TABLE user_books (...);
CREATE TABLE user_bok2 (...); -- 같은 개념의 중복 테이블 금지!
```

### 3.5 임시/실험용 테이블 관리 규칙

임시 또는 실험용 테이블은 다음 규칙을 따라야 합니다:

- ✅ **명명 규칙**: `_temp_<기능명>` 또는 `_experimental_<기능명>` 접두사 필수
- ✅ **문서화**: `DATA_MODEL.md`에 "임시/실험용" 표시 및 목적 명시
- ✅ **정리 조건**: 프로덕션 배포 전 반드시 제거 또는 정식 테이블로 전환
- ✅ **생명주기**: 생성일로부터 3개월 이내 정리 또는 전환

```sql
-- ✅ 허용: 명명 규칙 준수
CREATE TABLE _temp_search_cache (...);
CREATE TABLE _experimental_ai_suggestions (...);
```

---

## 4. 스키마 변경 규칙

### 4.1 변경 절차

데이터베이스 구조 변경이 발생하면 반드시 다음 절차를 따릅니다:

1. **`doc/database/DATA_MODEL.md`를 먼저 수정**
2. **실제 스키마 변경이 있다면 SQL 마이그레이션 파일 생성**

### 4.2 마이그레이션 파일 위치

SQL 마이그레이션 파일은 반드시 다음 경로에 생성합니다:

**`doc/database/`** (향후 `migrations/` 폴더로 정리 예정)

### 4.3 파일명 규칙

```
migration-YYYYMMDDHHmm__<기능명>__<변경내용>.sql
```

**예시:**
- `migration-202412151430__notes__add_page_number_column.sql`
- `migration-202412151500__books__add_isbn_index.sql`
- `migration-202412151600__users__update_rls_policy.sql`

### 4.4 스키마 변경 범위

스키마 변경에는 다음이 포함됩니다:

- 테이블 생성/삭제
- 컬럼 추가/삭제
- 인덱스 변경
- RLS 정책 변경
- 트리거 또는 함수 추가

### 4.5 Idempotent 작성 원칙

마이그레이션 파일은 **idempotent**하게 작성합니다:

```sql
-- ✅ 올바른 예: Idempotent 마이그레이션
CREATE TABLE IF NOT EXISTS notes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  ...
);

DROP POLICY IF EXISTS "Users can view own profile" ON users;
CREATE POLICY "Users can view own profile"
  ON users FOR SELECT
  USING (auth.uid() = id);

-- ❌ 나쁜 예: Idempotent하지 않음
CREATE TABLE notes (...); -- 여러 번 실행 시 오류 발생
```

### 4.6 UUID 생성 방식

프로젝트 표준 UUID 생성 방식:

- ✅ **권장**: `gen_random_uuid()` (PostgreSQL 13+ 기본 제공, 확장 불필요)
- ⚠️ **허용**: `uuid_generate_v4()` (uuid-ossp 확장 필요, 기존 코드 호환)

```sql
-- ✅ 권장: gen_random_uuid() 사용
CREATE TABLE books (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  ...
);

-- ⚠️ 허용: uuid_generate_v4() (기존 코드 호환)
CREATE TABLE books (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  ...
);
```

**새로운 테이블 생성 시에는 반드시 `gen_random_uuid()`를 사용합니다.**

---

## 5. 데이터 무결성 원칙

### 5.1 단일 테이블 원칙

- 하나의 도메인 개념은 반드시 **하나의 테이블로만** 표현합니다.
- 의미가 같은 데이터를 여러 테이블로 나누는 행위는 **금지**합니다.

### 5.2 소유자 판단 기준

소유자 판단은 다음 기준으로만 합니다:

- ✅ **인증 정보**: `auth.uid()` (Supabase Auth의 사용자 ID)
- ✅ **명시적 그룹 멤버십**: `group_members` 테이블 기준

**중요**: 모든 사용자 관련 외래 키는 `auth.users(id)`를 참조합니다.

```typescript
// ✅ 올바른 예: auth.uid() 사용
const { data: { user } } = await supabase.auth.getUser();
const { data } = await supabase
  .from("user_books")
  .select("*")
  .eq("user_id", user.id); // auth.users.id 기준

// ❌ 금지: 다른 방식의 소유자 판단
if (book.owner_email === user.email) { ... } // 금지!
```

```sql
-- ✅ 올바른 예: auth.users 참조
CREATE TABLE user_books (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  book_id UUID NOT NULL REFERENCES books(id) ON DELETE CASCADE,
  ...
);

-- ❌ 나쁜 예: 다른 테이블 참조
CREATE TABLE user_books (
  user_id UUID NOT NULL REFERENCES users(id), -- users 테이블이 아닌 auth.users 참조해야 함
  ...
);
```

### 5.3 접근 제어 강제

접근 제어는 **UI가 아닌 DB(RLS)에서 강제**합니다.

```sql
-- ✅ 올바른 예: RLS에서 접근 제어
CREATE POLICY "Users can view own books"
  ON user_books FOR SELECT
  USING (auth.uid() = user_id);

-- ❌ 나쁜 예: UI에서만 접근 제어 (RLS 없음)
-- 컴포넌트에서 if (user.id === book.user_id) 체크만 하는 경우
```

### 5.4 외래 키 관계

모든 외래 키 관계는 명시적으로 정의하고 **CASCADE 규칙을 명확히** 합니다.

```sql
-- ✅ 올바른 예: 명시적 CASCADE 규칙
CREATE TABLE user_books (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  book_id UUID NOT NULL REFERENCES books(id) ON DELETE CASCADE
);
```

---

## 6. 타입 안정성

### 6.1 타입 정의 필수

- 모든 테이블은 `types/database.ts`에 타입 정의가 있어야 합니다.
- Supabase 쿼리 결과는 반드시 타입을 지정합니다.
- 테이블 이름 변경 시 타입도 함께 업데이트합니다.

### 6.2 타입 사용 예시

```typescript
// ✅ 좋은 예: 타입 명시
import type { Database } from "@/types/database";

const { data } = await supabase
  .from("user_books")
  .select("*")
  .returns<Database["public"]["Tables"]["user_books"]["Row"][]>();

// ❌ 나쁜 예: 타입 없음
const { data } = await supabase.from("user_books").select("*");
```

### 6.3 타입 일관성

타입 정의는 `DATA_MODEL.md`와 일치해야 합니다.

---

## 7. 변경 사항 문서화

### 7.1 필수 문서화 항목

데이터와 관련된 기능을 구현하거나 수정할 때, 항상 아래 내용을 함께 문서화해야 합니다:

#### 1) 수정되거나 추가된 파일 목록

- `app/actions/**` (Server Actions)
- `hooks/**` (해당되는 경우)
- `types/database.ts` (타입 변경 시)

#### 2) 수정된 `doc/database/DATA_MODEL.md` 섹션

- 변경된 테이블/컬럼 정의
- 관계 변경 사항
- 접근 제어 원칙 변경

#### 3) 생성된 SQL 마이그레이션 파일 (해당되는 경우)

- `doc/database/migration-YYYYMMDDHHmm__<기능명>__<변경내용>.sql`

### 7.2 문서화 예시

```markdown
## 변경 사항 문서화 예시

### 작업: 기록에 페이지 번호 컬럼 추가

1. **수정된 파일:**
   - `app/actions/notes.ts` (페이지 번호 파라미터 추가)
   - `types/database.ts` (Note 타입 업데이트)

2. **수정된 DATA_MODEL.md:**
   - Notes 테이블에 `page_number INTEGER` 컬럼 추가
   - 목적: 기록과 책의 특정 페이지를 연결

3. **생성된 마이그레이션:**
   - `doc/database/migration-202412151430__notes__add_page_number_column.sql`
```

### 7.3 완료 기준

위 항목이 누락되면 작업은 **완료되지 않은 것으로 간주**합니다.

---

## 8. 점진적 마이그레이션 전략

### 8.1 기존 코드 유지

- 기존 `app/actions/` 구조는 **유지**합니다.
- 새로운 기능부터 규칙을 **엄격히 적용**합니다.

### 8.2 기존 코드 리팩토링

- 기존 코드는 리팩토링 시 규칙에 맞게 수정합니다.
- 규칙 위반 코드를 발견하면 즉시 수정하지 않고, **다음 리팩토링 기회에 수정**합니다.

### 8.3 새 코드 규칙 준수

하지만 **새로운 코드는 반드시 규칙을 준수**해야 합니다.

```typescript
// ✅ 새 코드: 규칙 준수
export async function newFeature() {
  const supabase = await createServerSupabaseClient();
  const { data } = await supabase
    .from("new_table")
    .select("*")
    .returns<Database["public"]["Tables"]["new_table"]["Row"][]>();
  return data;
}

// ⚠️ 기존 코드: 리팩토링 기회에 수정
export async function oldFeature() {
  const supabase = await createServerSupabaseClient();
  const { data } = await supabase.from("old_table").select("*"); // 타입 없음
  return data; // 나중에 수정 예정
}
```

---

## 9. 필수 폴더 및 파일 구조

### 9.1 필수 구조

아래 구조가 없다면 반드시 생성합니다:

```
doc/
└─ database/
   ├─ DATA_MODEL.md (단일 기준 문서)
   └─ migration-*.sql (SQL 마이그레이션 파일)

types/
└─ database.ts (Supabase 타입 정의)
```

### 9.2 DATA_MODEL.md 초기 구조

`doc/database/DATA_MODEL.md` 파일은 아래 항목을 포함한 최소 구조로 작성합니다:

```markdown
# 데이터 모델 (Data Model)

## 개요(Overview)
- 프로젝트 데이터 모델의 전체 목적
- 주요 도메인 개념

## 네이밍 규칙
- 테이블 네이밍 컨벤션
- 컬럼 네이밍 컨벤션
- 인덱스 네이밍 컨벤션

## 테이블 정의
각 테이블마다:
- 테이블명
- 목적 (한 문장)
- 소유 구조 (개인/그룹/공개)
- 주요 컬럼 설명
- 관계 정의 (외래 키, auth.users 기준)
- 인덱스
- RLS 정책 요약

## 관계 요약
- ERD 또는 관계 다이어그램 (선택)
- 테이블 간 관계 설명

## 접근 제어 원칙
- RLS 정책의 의도
- 소유자 판단 기준 (auth.uid())
- 그룹 접근 규칙

## 변경 로그(Change Log, 선택)
- 주요 스키마 변경 이력
```

### 9.3 비즈니스 독립성

비즈니스에 종속된 테이블은 임의로 생성하지 않습니다.  
확장 가능한 **중립적인 구조**로 작성합니다.

---

## 10. 규칙 위반 시 조치

### 10.1 컴포넌트에서 Supabase 직접 호출 발견 시

**조치:**
→ `hooks/`를 통해 간접 호출하도록 수정

```typescript
// ❌ 발견된 위반 코드
export function BookList() {
  const supabase = createClient();
  const { data } = await supabase.from("user_books").select("*");
  return <div>...</div>;
}

// ✅ 수정 후
export function BookList() {
  const { books } = useBooks(); // hooks를 통한 간접 호출
  return <div>...</div>;
}
```

### 10.2 UI 레이어에서 DB 구조 노출 발견 시

**조치:**
→ DB 구조 참조를 제거하고 비즈니스 로직으로 대체

```typescript
// ❌ 발견된 위반 코드
const tableName = "user_books";
if (book.user_id === user.id) { ... }

// ✅ 수정 후
// hooks나 Server Action에서 권한 검증 처리
// 컴포넌트는 비즈니스 로직만 사용
const canEdit = book.canEdit; // Server Action에서 계산된 값
```

### 10.3 DATA_MODEL.md 미반영 스키마 변경 발견 시

**조치:**
→ 즉시 문서 업데이트 및 마이그레이션 파일 생성

### 10.4 중복 테이블 발견 시

**조치:**
→ 통합 계획 수립 및 마이그레이션

---

## 11. 실제 적용 예시

### 11.1 새로운 기능 추가 시

**시나리오:** 기록에 태그 기능 추가

1. **DATA_MODEL.md 수정**: Notes 테이블에 tags 컬럼 정의 추가
2. **마이그레이션 파일 생성**: `migration-YYYYMMDDHHmm__notes__add_tags_column.sql`
3. **타입 업데이트**: `types/database.ts`에 tags 필드 추가
4. **Server Action 수정**: `app/actions/notes.ts`에서 tags 처리 로직 추가

### 11.2 기존 코드 리팩토링 시

**시나리오:** 컴포넌트에서 직접 Supabase 호출 제거

**Before (위반):** 컴포넌트에서 `createClient()` → `supabase.from()` 직접 호출  
**After (규칙 준수):** 컴포넌트 → `hooks/` → `app/actions/` → Supabase

---

## 12. 체크리스트

새로운 기능 개발 시 다음 체크리스트를 확인하세요:

### 개발 전
- [ ] `doc/database/DATA_MODEL.md`에 변경 사항 반영
- [ ] 필요한 마이그레이션 파일 계획 수립

### 개발 중
- [ ] 컴포넌트에서 Supabase 직접 호출하지 않음
- [ ] 컴포넌트에서 DB 구조(테이블명, 컬럼명) 노출하지 않음
- [ ] Server Action에서 타입 명시
- [ ] RLS 정책 확인
- [ ] 외래 키는 `auth.users(id)` 참조

### 개발 후
- [ ] `types/database.ts` 타입 업데이트
- [ ] 마이그레이션 파일 생성 (해당되는 경우)
- [ ] `DATA_MODEL.md` 최종 확인
- [ ] 변경 사항 문서화 완료
- [ ] UUID 생성 시 `gen_random_uuid()` 사용 (새 테이블)

---

## 13. 참고 사항

### 13.1 규칙의 목적

- 이 규칙은 **데이터 구조의 무결성을 보장**하기 위한 것입니다.
- **개발 속도보다 데이터 일관성을 우선시**합니다.

### 13.2 기술 부채 관리

- 규칙 위반은 **기술 부채로 누적**되므로, 가능한 한 즉시 수정합니다.
- 불가피한 예외는 반드시 **문서화하고 이유를 명시**합니다.

### 13.3 팀 협업

- 모든 팀원은 이 규칙을 숙지하고 준수해야 합니다.
- 규칙 위반 발견 시 즉시 팀에 공유하고 수정 계획을 수립합니다.

---

## 14. 규칙 업데이트 이력

| 날짜 | 버전 | 변경 내용 | 작성자 |
|------|------|-----------|--------|
| 2025-01 | 1.0 | 초기 규칙 작성 | - |
| 2025-01 | 2.0 | 경로 통일, UI 레이어 DB 구조 노출 금지 규칙 추가, hooks 패턴 표준화, auth.users 기준 명시, UUID 생성 방식 명확화, 임시 테이블 관리 규칙 추가 | - |

---

**이 문서는 프로젝트의 데이터 구조 무결성을 보장하기 위한 핵심 규칙입니다. 모든 개발자는 이 규칙을 준수해야 합니다.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/habitree) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
