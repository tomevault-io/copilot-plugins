## commit-rule

> 프로젝트 커밋 메시지 및 브랜치 규칙

# 커밋 메시지 규칙

## 기본 형식
```
<type>(<scope>): <subject>

<body>

<footer>
```

## 타입 (Type)
- `feat`: 새로운 기능 추가
- `fix`: 버그 수정
- `docs`: 문서 변경
- `style`: 코드 포맷팅, 세미콜론 누락 등 (코드 변경 없음)
- `refactor`: 코드 리팩토링
- `perf`: 성능 개선
- `test`: 테스트 추가/수정
- `chore`: 빌드 프로세스, 패키지 매니저 구성 등 변경
- `design`: UI/UX 디자인 변경
- `ci`: CI 구성 파일 및 스크립트 변경
- `revert`: 이전 커밋으로 되돌리기

## 범위 (Scope)
프로젝트의 어느 부분이 변경되었는지 명시 (선택 사항)
- `auth`: 인증/인가 관련
- `api`: API 엔드포인트
- `ui`: UI 컴포넌트
- `db`: 데이터베이스 관련
- `config`: 설정 파일
- `deps`: 의존성 관련

## 제목 (Subject)
- 명령문, 현재 시제로 작성 ("변경" 아닌 "변경함")
- 첫 글자는 소문자로 시작
- 끝에 마침표(.) 없음
- 50자 이내 작성

## 본문 (Body)
- 제목과 본문 사이에 빈 줄 추가
- "어떻게"보다 "무엇을", "왜" 변경했는지 설명
- 각 줄은 72자 이내로 작성

## 꼬리말 (Footer)
- 이슈 트래커 ID 등 참조 정보
- `Fixes #123` 또는 `Closes #123` 형식으로 이슈 참조
- `Breaking change: ` 접두어로 주요 변경사항 명시

## 예시
```
feat(auth): 소셜 로그인 기능 추가

OAuth2 프로토콜을 사용한 Google 로그인 기능을 구현
- 사용자 프로필 정보 가져오기 추가
- 토큰 갱신 로직 구현

Closes #123
```

# 브랜치 명명 규칙
- `main`: 배포 가능한 기본 브랜치
- `dev`: 개발 중인 코드의 통합 브랜치
- `feature/<이름>`: 새로운 기능 개발 (예: feature/social-login)
- `bugfix/<이름>`: 버그 수정
- `hotfix/<이름>`: 긴급 수정
- `release/<버전>`: 릴리스 준비 (예: release/1.2.0)
- `docs/<이름>`: 문서 작업

# 풀 리퀘스트 규칙
- 모든 PR은 최소 1명의 코드 리뷰를 받아야 함
- PR 제목은 관련 이슈 번호와 커밋 타입 포함
- PR 본문에는 변경 사항에 대한 설명과 테스트 방법 기술

---
> Source: [yaong-stream/yaong-v2](https://github.com/yaong-stream/yaong-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
