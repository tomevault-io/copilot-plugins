## reminder

> - 인터페이스는 `service/ports/in` 패키지에 정의

# Coding Conventions

## Service 계층
- 인터페이스는 `service/ports/in` 패키지에 정의
- 구현 클래스는 `service` 패키지에 위치하며, 네이밍은 `Default` 접두사 사용 (예: `DefaultReminderService`)

## 테스트
- 기능 추가/수정 시 반드시 검증 테스트를 함께 작성
- 도메인 엔티티 테스트는 순수 단위 테스트
- Service 테스트는 `@SpringBootTest` 통합 테스트로 작성 (Mock 사용 금지)

## 참고 문서
- spec.md: 기능 명세
- plan.md: 개발 계획 (6 phases - 백엔드 Phase 1~3 → 프론트엔드 Phase 4~6)
- tasks.md: 구현 체크리스트

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ralla0405)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ralla0405)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
