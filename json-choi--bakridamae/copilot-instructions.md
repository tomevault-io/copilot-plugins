## bakridamae

> 새로운 앱 생성 가이드라인


# 새 앱 생성 가이드

## 자동 생성 스크립트 사용

```bash
./create-new-app.sh <app-name> [app-title] [app-description]
```

## 스크립트가 자동으로 수행하는 작업

1. `apps/app-template`을 `apps/앱이름`으로 복사
2. 플레이스홀더 `{{APP_NAME}}`, `{{APP_TITLE}}`, `{{APP_DESCRIPTION}}` 교체
3. 루트 `package.json`에 `dev:앱이름` 스크립트 추가
4. 생성 스크립트 제거

## 수동 작업 (필요시)

- 특별한 의존성이 있다면 `package.json`에 추가
- 앱별 특수 설정이 필요하다면 구성 파일 수정

## 기본 포함 기능

- Next.js 15 + App Router
- TypeScript 설정
- 반응형 디자인 시스템
- 다크 테마 기본값
- shadcn/ui 컴포넌트 지원

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/json-choi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
