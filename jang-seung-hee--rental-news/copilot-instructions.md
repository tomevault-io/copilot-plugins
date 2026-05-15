## typescript-react

> TypeScript/React 전용 규칙

## TypeScript/React 규칙

- 절대경로 임포트 사용: `@/*` 경로 별칭 유지. 상대경로 남용 금지
- SRP 준수: 컴포넌트는 하나의 역할만, 복잡 로직은 훅/서비스로 분리
- 훅 네이밍: `use*` 강제, 사이드이펙트는 훅이나 서비스에만
- 파일 라인 수: 가급적 250~350, 450 경고, 500 초과 금지(신규 추가 시 분리)
- UI와 비즈니스 로직 분리: 프리젠테이션 컴포넌트 vs 서비스/훅
- 스타일: CSS Modules 또는 전역 토큰/유틸 우선. 무작위 전역 클래스 지양
- 테스트: 중요한 로직은 유닛 테스트 동반(예: 유틸/훅)

참고: [템플릿-AI 코딩,바이브 코딩 표준 정책.md](mdc:템플릿-AI 코딩,바이브 코딩 표준 정책.md)

---
> Source: [jang-seung-hee/rental-news](https://github.com/jang-seung-hee/rental-news) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
