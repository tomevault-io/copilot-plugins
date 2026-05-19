## typescript-deployment-safety

> 배포 시 TypeScript 타입 체크에서 다음과 같은 오류가 발생할 수 있습니다:

# TypeScript 배포 타입 안전성 가이드

## 문제 상황

배포 시 TypeScript 타입 체크에서 다음과 같은 오류가 발생할 수 있습니다:

```
Type error: Module '"@/types/database"' has no exported member 'ClientType'.
```

이는 타입 정의와 실제 사용 사이의 불일치로 인해 발생합니다.

## 원인 분석

### 발생 원인

1. **존재하지 않는 타입 import**
   - `@/types/database`에 정의되지 않은 타입을 import하려고 할 때
   - 타입 정의가 제거되었는데 코드에서 여전히 사용할 때

2. **타입 정의 누락**
   - 인터페이스나 타입 별칭이 정의되지 않았을 때
   - export 구문이 누락되었을 때

3. **타입 이름 오타**
   - 타입 이름이 잘못 입력되었을 때 (예: `ClientType` vs `ClientStatus`)

4. **변수 선언 누락**
   - Hook을 import했지만 실제로 변수로 선언하지 않았을 때
   - 예: `useUpdateClientWorker`를 import했지만 `updateClientWorkerMutation` 변수를 선언하지 않은 경우

### 이번 사례

#### 사례 1: 존재하지 않는 타입 import
- `ASTCompanyDashboard.tsx`에서 `ClientType`을 `@/types/database`에서 import하려고 했으나, 실제로는 해당 타입이 정의되어 있지 않음
- `clientCreateInitial` 상태에서 `client_type?: ClientType`을 사용하려 했으나, 데이터베이스 스키마에도 해당 필드가 없음
- 실제로 `clientCreateInitial`이 사용되지 않았음에도 타입 정의가 남아있었음

#### 사례 2: 변수 선언 누락
- `ReactStudioDashboard.tsx`에서 `useUpdateClientWorker`, `useDeleteClientWorker` hook을 import했지만
- 실제로 `updateClientWorkerMutation`, `deleteClientWorkerMutation` 변수를 선언하지 않아 사용 시 오류 발생
- 오류: `Cannot find name 'updateClientWorkerMutation'. Did you mean 'createClientWorkerMutation'?`

## 해결 방법

### 1. 타입 오류 해결 절차

1. **오류 확인**
   ```bash
   npm run build
   ```
   또는 IDE에서 TypeScript 오류 확인

2. **타입 정의 확인**
   - `src/types/database.ts`에서 해당 타입이 실제로 export되는지 확인
   - 데이터베이스 스키마와 타입 정의가 일치하는지 확인

3. **사용처 확인**
   ```bash
   # grep을 사용하여 타입 사용처 확인
   grep -r "ClientType" src/
   ```

4. **수정 방법 선택**
   - **타입이 필요하지 않은 경우**: import 및 사용 부분 제거
   - **타입이 필요한 경우**: 타입 정의 추가 및 export
   - **변수 선언 누락**: Hook을 사용하여 변수 선언 추가

#### 변수 선언 누락 해결 예시

```typescript
// ❌ 잘못된 예: Hook을 import만 하고 변수 선언 누락
import { useUpdateClientWorker, useDeleteClientWorker } from '@/features/erp/hooks';

// 코드에서 사용 시도
await updateClientWorkerMutation.mutateAsync({ ... }); // 오류!

// ✅ 올바른 예: Hook을 사용하여 변수 선언
import { useUpdateClientWorker, useDeleteClientWorker } from '@/features/erp/hooks';

const updateClientWorkerMutation = useUpdateClientWorker();
const deleteClientWorkerMutation = useDeleteClientWorker();

// 이제 사용 가능
await updateClientWorkerMutation.mutateAsync({ ... });
```

### 2. 예방 방법

#### 개발 단계에서 확인

1. **로컬 빌드 확인**
   ```bash
   npm run build
   ```
   - 배포 전 반드시 로컬에서 빌드 테스트 수행
   - TypeScript 오류가 있으면 빌드가 실패함

2. **IDE 설정 활용**
   - TypeScript 언어 서버가 활성화되어 있어야 함
   - 빨간 밑줄(오류 표시)을 무시하지 말고 즉시 수정

3. **Pre-commit Hook 설정 (권장)**
   ```json
   // package.json
   {
     "scripts": {
       "type-check": "tsc --noEmit"
     }
   }
   ```
   - Git commit 전에 타입 체크 수행

#### 코드 리뷰 체크리스트

- [ ] 새로 추가된 import 문이 실제로 사용되는가?
- [ ] 타입 정의가 실제 데이터베이스 스키마와 일치하는가?
- [ ] 사용되지 않는 타입 import가 있는가?
- [ ] 모든 타입이 올바르게 export되는가?
- [ ] Hook을 import한 경우, 실제로 변수로 선언되어 있는가?
- [ ] 코드에서 사용하는 모든 변수가 선언되어 있는가?

## 모범 사례

### 1. 타입 정의 관리

```typescript
// ✅ 좋은 예: 명확한 타입 정의와 export
export type ClientStatus = 'active' | 'inactive' | 'archived';
export interface ClientCompany {
  id: number;
  status: ClientStatus;
  // ...
}

// ❌ 나쁜 예: 존재하지 않는 타입 참조
import { ClientType } from '@/types/database'; // ClientType이 정의되지 않음
```

### 2. Import 정리

- 사용하지 않는 import는 즉시 제거
- IDE의 자동 정리 기능 활용 (VS Code: `Organize Imports`)

### 3. 점진적 타입 제거

타입이 필요 없어진 경우:
1. 타입 사용처를 모두 찾음
2. 사용처를 제거하거나 대체 타입으로 변경
3. 마지막으로 import 문 제거

### 4. Hook 사용 패턴

```typescript
// ✅ 좋은 예: Hook import와 변수 선언이 일관되게 유지됨
import { useUpdateClientWorker, useDeleteClientWorker } from '@/features/erp/hooks';

const updateClientWorkerMutation = useUpdateClientWorker();
const deleteClientWorkerMutation = useDeleteClientWorker();

// ❌ 나쁜 예: Hook을 import만 하고 변수 선언 누락
import { useUpdateClientWorker } from '@/features/erp/hooks';
// 변수 선언 없이 바로 사용 시도하면 오류 발생
```

### 5. 타입 검증 워크플로우

```
코드 작성 → 타입 체크 → 로컬 빌드 → 커밋 → 배포
           ↑                                    ↑
      (tsc --noEmit)                      (CI/CD)
```

## 체크리스트

배포 전 반드시 확인:

- [ ] `npm run build`가 성공적으로 완료되는가?
- [ ] 모든 TypeScript 오류가 해결되었는가?
- [ ] 사용되지 않는 import가 없는가?
- [ ] 타입 정의와 실제 데이터베이스 스키마가 일치하는가?
- [ ] IDE에서 타입 오류가 없는가?
- [ ] Hook을 import한 경우, 해당 Hook을 사용한 변수 선언이 있는가?
- [ ] 코드에서 사용하는 모든 변수(mutation, query 등)가 선언되어 있는가?

## 관련 파일

- 타입 정의: `src/types/database.ts`
- 타입 체크 설정: `tsconfig.json`
- 빌드 설정: `next.config.ts`

## 참고

- TypeScript 공식 문서: https://www.typescriptlang.org/docs/
- Next.js TypeScript 가이드: https://nextjs.org/docs/app/building-your-application/configuring/typescript

---
> Source: [tkaykim/totalmanagement](https://github.com/tkaykim/totalmanagement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
