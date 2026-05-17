## myhomepage

> 이 파일은 Claude Code가 이 프로젝트에서 코드를 작성하거나 수정할 때 **항상** 준수해야 하는 규칙입니다.

# CLAUDE.md — AI 코딩 규칙

이 파일은 Claude Code가 이 프로젝트에서 코드를 작성하거나 수정할 때 **항상** 준수해야 하는 규칙입니다.

---

## 프로젝트 구조

```
jamtori-main/
├── client/          # React 18 + TypeScript + Vite + Tailwind
├── server/          # Express 5 + TypeScript + Sequelize ORM
└── .github/
    └── workflows/
        └── ci.yml   # GitHub Actions CI (자동 검증)
```

---

## 코드 수정 후 필수 검증 순서

코드를 수정한 뒤 반드시 아래 순서로 검증하고 에러가 없음을 확인한다.

```bash
# 서버
cd server
npx tsc --noEmit          # TypeScript 타입 에러 0개 확인
npm run lint              # ESLint 경고/에러 0개 확인
npm run format:check      # Prettier 포맷 일치 확인
npm test                  # 테스트 통과 확인
npm run build             # 빌드 성공 확인 (Docker 배포 시 dist/ 필수)

# 클라이언트
cd client
npx tsc --noEmit
npm run lint
npm run format:check
npm run build             # 빌드 성공 확인
```

---

## TypeScript 규칙 (server/tsconfig.json + client/tsconfig.json)

- `strict: true` — 모든 strict 옵션 활성화
- `noUnusedLocals: true` — 미사용 로컬 변수 금지
- `noUnusedParameters: true` — 미사용 파라미터 금지
- `noImplicitAny: true` — any 타입 암묵적 사용 금지
- `noImplicitReturns: true` — 누락된 return 금지

### 미사용 파라미터 처리 규칙

Express/Multer 콜백 등에서 파라미터가 필요하지만 사용하지 않을 때는 `_` 접두사 사용:

```typescript
// 올바름
export const getList = async (_req: Request, res: Response) => { ... };
function fileFilter(_req: Request, file: Express.Multer.File, cb: ...) { ... }
for (const [, value] of map.entries()) { ... }
const [, created] = await Model.findOrCreate({ ... });
```

### any 타입 사용 규칙

`any` 사용은 `@typescript-eslint/no-explicit-any: warn`으로 경고 처리된다.
가능하면 구체적인 타입이나 `unknown`을 사용하고, 불가피할 때만 `// eslint-disable-next-line @typescript-eslint/no-explicit-any`와 함께 사용한다.

---

## ESLint 규칙

### 서버 (server/eslint.config.js)

| 규칙 | 수준 | 설명 |
|------|------|------|
| `@typescript-eslint/no-explicit-any` | warn | any 사용 최소화 |
| `@typescript-eslint/no-unused-vars` | warn | 미사용 변수 (`_` 접두사 허용) |
| `@typescript-eslint/no-floating-promises` | error | await 누락 금지 |
| `@typescript-eslint/no-misused-promises` | error | Promise를 if 조건에 직접 사용 금지 |
| `no-console` | warn | `console.warn/error` 만 허용 |
| `no-duplicate-imports` | error | 중복 import 금지 |
| `eqeqeq` | error | `==` 대신 `===` 강제 |
| `prefer-const` | error | `let` 대신 `const` 우선 |
| `no-var` | error | `var` 사용 금지 |

### 클라이언트 (client/eslint.config.js)

서버 규칙 + React 전용:

| 규칙 | 수준 |
|------|------|
| `react-hooks/rules-of-hooks` | error |
| `react-hooks/exhaustive-deps` | warn |
| `react-refresh/only-export-components` | warn |

---

## Prettier 규칙

서버(`server/.prettierrc`)와 클라이언트는 동일한 설정을 공유한다:

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100,
  "bracketSpacing": true,
  "arrowParens": "avoid",
  "endOfLine": "lf"
}
```

---

## 서버 코드 규칙

### Logger 사용

`console.log` 대신 반드시 `logger` 유틸 사용:

```typescript
import { logInfo, logSuccess, logError, logWarning } from '../utils/logger';

logInfo('메시지', { key: value });   // 정보
logSuccess('메시지', { key: value }); // 성공
logError('메시지', error, { key: value }); // 에러
logWarning('메시지', { key: value }); // 경고
// logger.debug()는 존재하지 않음 — logInfo() 사용
```

### 응답 유틸 사용

직접 `res.json()` 대신 응답 유틸 사용:

```typescript
import { sendSuccess, sendError, sendNotFound, sendUnauthorized, sendForbidden } from '../utils/response';

sendSuccess(res, data, '메시지', 201);
sendError(res, 500, '에러 메시지');
sendNotFound(res, '리소스명');
sendUnauthorized(res, '메시지');
sendForbidden(res, '메시지');
```

### 비동기 에러 처리

모든 async 함수에서 Promise를 반드시 await 처리 (`no-floating-promises: error`):

```typescript
// 올바름
await someService.doSomething();
const result = await Model.findAll();

// 금지 — floating promise
someService.doSomething();  // ESLint error
```

### 컨트롤러 구조

비즈니스 로직은 Service 레이어에 위치시키고, 컨트롤러는 얇게 유지:

```typescript
// 올바름 — 컨트롤러는 요청/응답 처리만
export const getList = async (req: AuthRequest, res: Response): Promise<void> => {
  try {
    const result = await someService.getList(req.user.id);
    sendSuccess(res, result);
  } catch (err) {
    logError('조회 실패', err);
    sendError(res, 500, '조회 실패');
  }
};
```

### Import 순서

1. Node.js 내장 모듈 (`fs`, `path`, `crypto`)
2. 외부 패키지 (`express`, `sequelize`)
3. 내부 모듈 (`../models/...`, `../services/...`, `../utils/...`)

중복 import 금지 (`no-duplicate-imports: error`).

---

## 테스트 규칙

### 테스트 위치

`server/src/__tests__/` 디렉토리에 `*.test.ts` 파일로 작성.

### 테스트 환경

테스트는 SQLite in-memory DB를 사용 (`env-setup.ts`에서 자동 설정). 실제 DB에 영향 없음.

### 새 API 추가 시

핵심 경로에 대해 최소한 다음을 테스트:

```typescript
it('인증된 사용자 - 성공', async () => { ... });
it('미인증 - 401 반환', async () => { ... });
it('잘못된 입력 - 400 반환', async () => { ... });
```

### 테스트 헬퍼

```typescript
import { seedTestData, loginAs } from './helpers';

beforeEach(async () => { await seedTestData(); });

const cookie = await loginAs('admin', 'TestAdmin123!');
const res = await request(app).get('/api/...').set('Cookie', cookie);
```

---

## 데이터베이스 규칙

### Soft Delete 이중 전략 (의도된 설계)

- `isDeleted`: 앱 레벨 익명화 처리 플래그
- `deletedAt`: Sequelize paranoid 기반 DB 레벨 소프트 삭제

두 필드가 공존하는 것은 버그가 아닌 의도된 설계.

### 새 모델 추가 시

`server/src/models/index.ts`에 등록하고, 필요하다면 `server/src/scripts/add-indexes.ts`에 인덱스 추가.

---

## CI/CD

`.github/workflows/ci.yml`이 Push/PR 시 자동으로 실행:

1. **서버**: TypeScript 타입 체크 → ESLint → Prettier → Jest 테스트
2. **클라이언트**: TypeScript 타입 체크 → ESLint → Prettier → Vite 빌드

**CI가 실패하는 코드는 머지 불가.**

로컬에서 미리 검증:

```bash
cd server && npx tsc --noEmit && npm run lint && npm run format:check && npm test
cd client && npx tsc --noEmit && npm run lint && npm run format:check && npm run build
```

---

## 금지 사항

- `console.log()` 직접 사용 → `logInfo()` 사용
- `var` 선언 → `const`/`let` 사용
- `==` 비교 → `===` 사용
- `any` 타입 남용 → 구체적 타입 또는 `unknown`
- await 없이 async 함수 호출 (floating promise)
- 미사용 import/변수 방치 (TypeScript 컴파일 에러)
- 동일 모듈을 여러 줄에서 import

---
> Source: [zz1231me/myhomepage](https://github.com/zz1231me/myhomepage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
