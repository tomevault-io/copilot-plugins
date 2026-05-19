## a-and-i-web-client

> 이 문서는 `GEMINI.md`를 바탕으로, 이 저장소에서 작업하는 모든 AI 에이전트가 공통으로 따라야 할 실행 지침을 정의합니다.

# AGENTS.md

이 문서는 `GEMINI.md`를 바탕으로, 이 저장소에서 작업하는 모든 AI 에이전트가 공통으로 따라야 할 실행 지침을 정의합니다.

## 1) 프로젝트 목적
- A&I 동아리 CS 멘토링 과제 공지/제출 현황 웹 플랫폼을 안정적으로 운영한다.
- UX(특히 과제 맥락), 보안(JWT 기반), 운영 안정성(CI/CD + Firebase Hosting)을 우선한다.

## 2) 아키텍처 원칙 (필수)
- Clean Architecture + MVVM을 준수한다.
- 계층 책임:
  - `domain/`: 엔티티, 유스케이스, 리포지토리 인터페이스 (순수 비즈니스 로직)
  - `data/`: API/스토리지/DTO/리포지토리 구현
  - `ui/`, `providers/`: 화면/상태/입력 처리
- 의존성 방향을 역전하거나 계층 책임을 침범하지 않는다.
- UI 위젯 내부에서 API 호출/비즈니스 로직을 직접 처리하지 않는다.

## 3) 상태관리/모델
- Riverpod 기반 상태 관리를 사용한다.
- 신규 상태 모델/이벤트 모델은 `freezed` 사용을 기본으로 한다.
- 코드 생성이 필요한 변경(`freezed`, `retrofit`, `riverpod_generator`) 후에는 `build_runner`를 실행한다.

## 4) 코딩 스타일
- 기존 코드 스타일을 최대한 모방한다.
- 명명 규칙:
  - 클래스/enum: `PascalCase`
  - 변수/함수: `camelCase`
  - 파일/폴더: `snake_case`
- public API에는 `///` 문서 주석을 작성하며, 설명은 한국어로 작성한다.
- `dynamic` 사용을 피하고 명시 타입을 우선한다.
- 비동기는 `async/await` 우선으로 작성한다.

## 5) 운영/환경
- 실행 시 API URL은 `--dart-define=API_URL=...`로 주입한다.
- 배포 파이프라인(GitHub Actions/Firebase Hosting)은 임의 변경하지 않는다.
- 배포 이슈는 `.github/workflows`를 우선 점검한다.

## 6) 작업 프로세스
복잡한 작업은 아래 순서를 따른다.
1. `PRD.md`: 요구사항 명확화
2. `PLAN.md`: 구현 계획 수립
3. `TASK.md`: 실행 단위 태스크 추적

## 7) 저장소 운영 주의사항
- `.gitignore`에 명시된 AI 작업 문서(`PRD.md`, `PLAN.md`, `TASK.md`, `GEMINI.md` 등)는 Git 관리 대상이 아닐 수 있음을 인지한다.
- 해당 파일은 Git 조작보다 파일 시스템 직접 수정 기준으로 다룬다.

## 8) 빠른 체크리스트
- [ ] 계층 분리가 지켜졌는가?
- [ ] UI에서 비즈니스 로직/네트워크 호출을 제거했는가?
- [ ] 모델/이벤트에 `freezed`를 적용했는가?
- [ ] 필요한 코드 생성을 수행했는가?
- [ ] `flutter analyze` 통과 여부를 확인했는가?

---
> Source: [Team-AnI/A-AND-I-WEB-CLIENT](https://github.com/Team-AnI/A-AND-I-WEB-CLIENT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
