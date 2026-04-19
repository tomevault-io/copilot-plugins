## executejs

> 이 프로젝트는 ExecuteJS 모노레포입니다. 다음 규칙을 따라 개발하세요.

# ExecuteJS Cursor Rules

이 프로젝트는 ExecuteJS 모노레포입니다. 다음 규칙을 따라 개발하세요.

## 프로젝트 구조

### 모노레포 구성
- `apps/executeJS/`: Tauri 데스크톱 애플리케이션
- `packages/`: 공유 Node.js 패키지들
- `crates/`: 공유 Rust 크레이트들
- `docs/`: RSPress 문서 사이트
- `.github/workflows/`: CI/CD 워크플로우

### 기술 스택
- **Frontend**: React 19, TypeScript, Vite, Vitest
- **UI**: Radix UI + Tailwind CSS
- **상태 관리**: Legend State
- **코드 에디터**: Monaco Editor
- **Backend**: Rust, Tauri 2.0
- **JavaScript Engine**: Node.js v24.12.0 (V8 기반)
- **Package Manager**: pnpm
- **Linting**: oxlint (JavaScript)
- **Formatting**: Prettier (JavaScript), rustfmt (Rust)
- **Documentation**: RSPress

### JavaScript 런타임 아키텍처

#### Node.js 통합
- **엔진**: Node.js v24.12.0 (V8 JavaScript 엔진)
- **실행 방식**: `tokio::process::Command`로 서브프로세스 실행
- **호환성**: Tauri 2.0과 완전 호환

#### 핵심 컴포넌트
```
apps/executeJS/src-tauri/src/
├── js_executor.rs           # 실행 결과 관리
└── commands.rs              # Tauri 명령어

crates/node-runtime/
├── src/lib.rs               # Node.js 런타임 구현
└── Cargo.toml
```

#### 실행 흐름
1. **초기화**: `NodeExecutor::new()` - Node.js 바이너리 경로 찾기
2. **실행**: `execute_script()` - 서브프로세스로 Node.js 실행
3. **결과 처리**: stdout/stderr에서 결과 수집 및 반환

#### 지원 기능
- ✅ `console.log()` - 표준 출력
- ✅ 변수 할당 및 계산
- ✅ 문법 오류 감지 (실제 JavaScript 엔진 수준)
- ✅ Node.js 표준 기능 지원

## FSD 아키텍처 규칙

### 폴더 구조
```
src/
├── app/                      # 앱 초기화 및 프로바이더
├── pages/                    # 페이지 레벨 컴포넌트
├── widgets/                  # 복합 UI 블록
├── features/                 # 비즈니스 로직 기능
├── shared/                   # 공유 유틸리티
└── main.tsx                  # Vite 엔트리
```

### 의존성 규칙
- **app** → pages, widgets, features, shared
- **pages** → widgets, features, shared
- **widgets** → features, shared
- **features** → shared
- **shared** → (다른 레이어 import 금지)

### 레이어별 책임
- **app/**: 앱 초기화, 전역 프로바이더
- **pages/**: 페이지 레벨 컴포넌트 (라우팅 단위)
- **widgets/**: 복합 UI 블록 (여러 features 조합)
- **features/**: 비즈니스 로직 (도메인별)
- **shared/**: 재사용 가능한 유틸리티

## 코딩 규칙

### JavaScript/TypeScript
- oxlint 사용 (eslint 대신)
- Prettier로 포맷팅
- TypeScript strict 모드
- Vitest + Testing Library로 테스트
- 함수형 컴포넌트와 Hooks 사용
- Legend State로 상태 관리
- Radix UI + Tailwind CSS로 스타일링

### Rust
- rustfmt 기본 설정 사용 (4 스페이스)
- clippy 린트 규칙 준수
- `tracing`으로 로깅
- `anyhow`, `thiserror`로 에러 처리
- Tauri 2.0 API 사용

### 파일 명명
- React 컴포넌트: PascalCase (예: `App.tsx`)
- Rust 모듈: snake_case (예: `js_executor.rs`)
- 설정 파일: kebab-case (예: `tauri.conf.json`)

## 개발 워크플로우

### 필수 명령어
```bash
# 개발
pnpm tauri:dev              # Tauri 개발 서버
pnpm docs:dev               # 문서 개발 서버

# 품질 관리
pnpm lint                   # 린트 검사
pnpm format                 # 코드 포맷팅
pnpm type-check             # 타입 검사

# 테스트
pnpm test                   # 프론트엔드 테스트
cargo test                  # Rust 테스트
```

### 커밋 컨벤션
```
type(scope): description

feat: 새로운 기능 추가
fix: 버그 수정
docs: 문서 수정
style: 코드 포맷팅
refactor: 코드 리팩토링
test: 테스트 추가/수정
chore: 빌드 설정 변경
```

## CI/CD 규칙

### 워크플로우 실행 조건
- **javascript-lint.yml**: `apps/executeJS/src/**` 변경 시
- **rust-lint.yml**: `apps/executeJS/src-tauri/**`, `crates/**` 변경 시
- **frontend-test.yml**: 프론트엔드 코드 변경 시
- **rust-test.yml**: Rust 코드 변경 시
- **build.yml**: 전체 프로젝트 변경 시
- **docs.yml**: `docs/**` 변경 시

### 브랜치 정책
- `main`: 메인 브랜치
- 모든 변경사항은 PR을 통해 검토
- CI가 통과해야만 머지 가능

## 코드 품질

### 필수 체크리스트
- [ ] oxlint 통과
- [ ] Prettier 포맷팅 적용
- [ ] TypeScript 타입 체크 통과
- [ ] Vitest 테스트 통과
- [ ] rustfmt 포맷팅 적용
- [ ] cargo test 통과

### 코드 리뷰 포인트
- 타입 안전성
- 에러 처리
- 테스트 커버리지
- 성능 최적화
- 보안 고려사항

## 문서화

### 문서 작성 규칙
- RSPress MDX 형식 사용
- 코드 예제는 실행 가능해야 함
- API 변경 시 문서 업데이트 필수
- 한국어로 작성

### 문서 구조
- `docs/docs/guide/`: 사용자 가이드
- `docs/docs/api/`: API 참조
- `docs/docs/index.mdx`: 홈페이지

## 디버깅 가이드

### 프론트엔드
- 브라우저 개발자 도구 사용
- React DevTools 활용
- Vitest 디버깅 모드 사용

### 백엔드
- `tracing` 로그 확인
- `tauri dev` 실행 시 콘솔 로그
- Rust 디버거 사용

### Tauri
- `tauri dev` 실행
- Tauri DevTools 사용
- 네이티브 로그 확인

### JavaScript 런타임
- `cargo test node_runtime::tests` - 런타임 테스트
- Node.js 바이너리 경로 확인 필요
- 리소스 번들링 확인 (`tauri.conf.json`)

## 주의사항

1. **의존성**: 모든 의존성은 `latest` 버전 사용
2. **성능**: 불필요한 리렌더링 방지
3. **보안**: 사용자 입력 검증 필수
4. **접근성**: 웹 접근성 가이드라인 준수
5. **호환성**: 크로스 플랫폼 지원

## 문제 해결

### 빌드 오류
1. `pnpm install` 실행
2. `cargo clean` 후 `cargo build`
3. Node.js 버전 확인 (22 LTS)
4. Rust 툴체인 확인

### 테스트 실패
1. 개별 테스트 실행으로 원인 파악
2. 테스트 환경 설정 확인
3. Mock 데이터 검증

### 린트 오류
1. `pnpm lint:fix` 실행
2. `cargo fmt` 실행
3. 수동으로 규칙에 맞게 수정

### Node.js 런타임 관련 문제
1. **바이너리 경로 오류**: 개발/프로덕션 모드 경로 확인
2. **리소스 번들링 오류**: `tauri.conf.json`의 `resources` 설정 확인
3. **권한 오류**: Unix 시스템에서 실행 권한 설정 확인
4. **서브프로세스 오류**: `tokio::process::Command` 실행 확인

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ohah) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
