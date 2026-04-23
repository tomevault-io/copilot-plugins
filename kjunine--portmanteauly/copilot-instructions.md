## portmanteauly

> Portmanteauly — 영어 단어 2~3개를 조합해 자연스러운 포트맨토 단어를 생성하는 npm 패키지 (CLI + 라이브러리)

# CLAUDE.md

## 프로젝트 개요

Portmanteauly — 영어 단어 2~3개를 조합해 자연스러운 포트맨토 단어를 생성하는 npm 패키지 (CLI + 라이브러리)

## 기술 스택

- **언어**: TypeScript (strict mode)
- **런타임**: Node.js 18+
- **모듈**: ESM only (`"type": "module"`)
- **패키지 관리**: pnpm
- **빌드**: tsup (ESM 번들링, .d.ts 생성)
- **테스트**: Vitest
- **린트**: ESLint + typescript-eslint
- **포맷**: Prettier

## 명령어

```bash
pnpm install          # 의존성 설치
pnpm build            # tsup으로 빌드
pnpm test             # Vitest 테스트 실행
pnpm lint             # ESLint 실행
pnpm format           # Prettier 포맷
```

## 프로젝트 구조

```
src/
├── index.ts              # 라이브러리 엔트리 (public API)
├── core/
│   ├── combiner.ts       # 조합 관리
│   ├── strategies/       # 생성 전략 (overlap, syllable, slice)
│   └── scorer.ts         # 스코어링 엔진
├── utils/                # 음절 분리, 발음 평가, n-gram, 필터
├── data/                 # 내장 영단어 목록, n-gram 빈도 데이터
├── cli/                  # CLI 엔트리
└── types.ts              # 공유 타입 정의
tests/                    # 단위/통합 테스트 (src/ 구조 미러링)
bin/                      # CLI 실행 파일
docs/                     # PRD, TRD, User Flow, Database Design, TASKS
```

## 코딩 컨벤션

- **네이밍**: camelCase (변수/함수), PascalCase (타입/인터페이스)
- **export**: named export 선호, default export 지양
- **동기**: 순수 동기 API (async/await 미사용)
- **에러**: 커스텀 Error 클래스 사용
- **주석**: JSDoc 스타일, public API에 필수
- **파일 길이**: 단일 파일 300줄 이하 권장

## 브랜치 전략

- `main`: 릴리스 가능 상태 유지
- `feat/*`: 기능 개발, `fix/*`: 버그 수정
- 직접 main push 허용 (1인 프로젝트 MVP 단계)
- 태그: `v{major}.{minor}.{patch}` (semver)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kjunine) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
