## sellpiece

> 모든 서버 액션은 **`src/actions`** 디렉토리에 위치해야 합니다.


# Sellpiece 프로젝트 규칙

## 서버 액션 (Server Actions) 위치 규칙

모든 서버 액션은 **`src/actions`** 디렉토리에 위치해야 합니다.

### ❌ 잘못된 예시

```
src/app/(storefront)/login/actions.ts
src/app/(storefront)/signup/actions.ts
src/app/admin/(auth)/login/actions.ts
```

### ✅ 올바른 예시

```
src/actions/auth.ts          # 인증 관련 액션 (로그인, 회원가입, 로그아웃)
src/actions/cart.ts          # 장바구니 관련 액션
src/actions/product.ts       # 제품 관련 액션
src/actions/order.ts         # 주문 관련 액션
```

### 이유

1. **중앙 집중화**: 모든 서버 액션을 한 곳에서 관리하여 유지보수성 향상
2. **재사용성**: 여러 페이지에서 동일한 액션을 쉽게 import하여 사용
3. **명확한 구조**: 서버/클라이언트 바운더리를 명확히 구분
4. **일관성**: 프로젝트 전체에서 일관된 구조 유지

### 사용 예시

```typescript
// src/actions/auth.ts
'use server';

export async function loginAction(prevState, formData) {
  // 로그인 로직
}

export async function signupAction(prevState, formData) {
  // 회원가입 로직
}
```

```typescript
// src/app/(storefront)/login/page.tsx
'use client';

import { loginAction } from '@/actions/auth';

export default function LoginPage() {
  // 페이지 컴포넌트
}
```

## 기타 아키텍처 규칙

- `src/lib/db/schema.ts`: 데이터베이스 스키마
- `src/lib/db/queries`: 데이터베이스 쿼리 함수
- `src/lib/supabase`: Supabase 클라이언트
- `src/config`: 프로젝트 설정
- `src/components`: 재사용 가능한 컴포넌트
- `src/actions`: **서버 액션 (Server Actions)**
- `src/app`: 파일 기반 라우팅

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/leejss) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
