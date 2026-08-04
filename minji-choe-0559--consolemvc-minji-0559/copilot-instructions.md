## consolemvc-minji-0559

> 이 문서는 Claude Code가 본 Repository에서 작업할 때 지켜야 할 경량 규칙을 정의한다.

# CLAUDE.md — ConsoleMVC PoC 운영 규칙

이 문서는 Claude Code가 본 Repository에서 작업할 때 지켜야 할 경량 규칙을 정의한다.
도메인 요구사항은 여기 다시 적지 않고 [POC_SPEC.md](./POC_SPEC.md)와 [PLAN.md](./PLAN.md)를 참조한다.

## 문서 우선순위

충돌이 발견되면 아래 우선순위에 따라 판단하되, **Claude는 임의로 판단하지 않고 충돌 내용을 사용자에게
보고한 뒤 지시를 받는다.**

1. 과제 원본 명세(`docs/[CRA_AI] Day3_개인과제_반도체시료관리.pdf`)
2. [POC_SPEC.md](./POC_SPEC.md)
3. [PLAN.md](./PLAN.md)
4. CLAUDE.md (본 문서)
5. 코드와 테스트

## 범위 제한

- MVC 역할 분리 검증만 수행한다.
- 데이터베이스 영속성을 구현하지 않는다.
- 반도체 주문·생산 전체 기능을 구현하지 않는다.
- 향후 기능을 위한 과도한 추상화를 도입하지 않는다.
- 인증, 네트워크, GUI를 구현하지 않는다.

## 구현 규칙

- View는 콘솔 입력과 출력만 담당하며, 도메인 타입(`SampleRecord` 등)이 아닌 Presentation 전용
  DTO(`SampleRegistrationInput`)/ViewModel(`SampleViewModel`, `SampleRegistrationResult`)만 참조한다.
  여러 필드를 개별 primitive 인자로 흩어서 전달하지 않는다.
- Controller는 사용자 흐름을 조정하며, `ISampleService`만 호출한다. `ISampleRepository`를 직접
  호출하지 않는다.
- Model 또는 Service(`ISampleService`)는 입력 검증(trim/필수/범위)과 업무 규칙을 담당한다.
- `SampleCode` 중복 거부의 최종 책임은 `ISampleRepository::Add`에 있다. Controller/Service가 `List`나
  별도 존재 확인 메서드로 먼저 조회한 뒤 `Add`를 호출하는 방식(Check-Then-Act)으로 중복을 보장하지 않는다.
- View는 Repository 또는 데이터 저장 구현에 직접 접근하지 않는다.
- Controller는 콘솔 출력 형식을 직접 생성하지 않는다.
- 비즈니스 로직은 `std::cin`, `std::cout`에 직접 의존하지 않는다.
- 전역 가변 상태를 사용하지 않는다.
- 의미 없는 `Manager`, `Helper`, `Util` 이름을 남용하지 않는다.
- 테스트만을 위한 불필요한 public API를 추가하지 않는다.
- 데이터 영속성의 세부 구현(JSON 포맷, `schemaVersion`, Atomic Write, 파일 잠금 등)은 다루지 않는다.
  `ISampleRepository`는 InMemory Fake 또는 GoogleMock 기반 Mock으로만 대체하며, 실 저장소 구현은
  `DataPersistence-*` PoC의 책임이다.

### RAII와 소유권 규칙

- 소유권이 불명확한 raw pointer(owning raw pointer)를 사용하지 않는다.
- `IView`, `ISampleService`, `ISampleRepository` 등 인터페이스에는 virtual destructor를 선언한다.
- 구체 객체(`ConsoleView`, `SampleService`, `InMemorySampleRepository` 등)의 생명주기는 `main`
  (Composition Root)이 소유한다.
- Controller와 Service는 주입받은 의존성(View, Service, Repository)을 non-owning reference(`&`)로
  보관하며, 소유권을 가지지 않는다.
- 불필요한 `shared_ptr` 사용을 금지한다. 공유 소유권이 실제로 필요하지 않다면 참조 또는
  `unique_ptr`(Composition Root 한정)을 사용한다.
- 수동 `delete`를 사용하지 않는다. RAII(스택 객체, `unique_ptr` 등)로 자원을 관리한다.

## Harness 규칙

작업 완료 전 아래 항목을 검증한다.

- Build
- Unit Test
- Compiler Warning (`/W4`, `/WX`, 프로젝트 코드에만 적용 — GoogleTest/GoogleMock 등 외부 벤더 헤더는
  `/external` 계열 옵션으로 제외하여 외부 코드의 경고가 빌드 실패를 유발하지 않게 한다)
- Format Check (clang-format, **검사 전용** 명령만 사용하고 자동 수정(in-place) 명령으로 대체하지 않는다)

실제 통합 명령(단일 verify 절차)은 [PLAN.md](./PLAN.md) Step 1에서 확정되었다: 저장소 루트의
`verify.ps1`이 Build/Unit Test/Compiler Warning/Format Check를 한 번에 실행한다(실행 방법은
[README.md](./README.md) 8절 참고). 이후 모든 작업 완료 전에는 이 스크립트를 실제로 실행해 통과를
확인한다.

이번 PoC 범위에서는 **CI 파이프라인을 도입하지 않는다.** 로컬 환경에서 실행 가능한 단일 verify 명령이
Build/Unit Test/Compiler Warning/Format Check를 자동으로 검증할 수 있으면 충분하다.

## 작업 완료 보고

작업을 마칠 때 Claude는 아래 정보를 출력한다.

1. 구현한 범위
2. 변경한 파일
3. MVC 책임 분리 결과
4. 작성한 테스트
5. 실행한 검증 명령
6. Build 및 Test 결과
7. 남은 제한사항
8. 제안 Commit Message (실제 커밋을 수행한 경우 실제 커밋 메시지/해시, 예상 Commit과 구성이 달라졌다면
   그 사유를 함께 기재)

## Git 작업 규칙

문서 역할 분담: [PLAN.md](./PLAN.md)는 Step별 예상 작업/예상 커밋 단위를, [COMMIT_CONVENTION.md](./COMMIT_CONVENTION.md)는
프로젝트 전체에 공통으로 적용되는 커밋 작성·분리·검증 규칙을, 본 절은 **Claude가 커밋을 실행할 수 있는
조건과 금지된 Git 작업**을 정의한다.

### 커밋 실행이 승인된 경우

- [PLAN.md](./PLAN.md) Step 1~4의 완료 단위 커밋은 사용자가 사전 승인한 것으로 처리한다.
- 커밋 단위는 PLAN.md에 표시된 Step별 "규모"를 따른다.
  - 규모가 작은 Step(Step 1, Step 4): 원칙적으로 **Step 전체 완료 시점**에 1개 커밋(Step 4는 refactor/
    docs 목적이 달라 예외적으로 2개까지 허용).
  - 규모가 큰 Step(Step 2, Step 3): **Step 내 하위 작업 단위**로 커밋할 수 있되, Step당 **3개 내외**를
    넘지 않도록 한다(과도한 세분화 금지).
- 위 경우를 제외한 커밋(예: PLAN.md의 Step 단위/하위 작업 단위에 해당하지 않는 임시 커밋, PLAN.md에
  없는 작업의 커밋, PLAN.md에 새 Step을 추가하는 경우)은 사용자의 별도 승인 없이 수행하지 않는다.

### 커밋 실행 전 확인 절차

사전 승인된 커밋이라도 실행 전 반드시 아래를 확인한다.

1. [COMMIT_CONVENTION.md](./COMMIT_CONVENTION.md)를 확인하여 Atomic Commit 원칙, Type/Scope, 커밋
   가능한 상태 조건에 맞는지 점검한다.
2. Build, Unit Test, Compiler Warning(`/W4`/`/WX`), Format Check(clang-format) 검증 결과를 확인하고,
   하나라도 실패한 상태면 커밋하지 않는다.
3. `git status`/`git diff`/`git diff --staged`로 실제 스테이징 내용을 확인해 의도치 않은 파일이
   포함되지 않았는지 점검한다.

### 금지된 Git 작업

사용자가 명시적으로 요청하지 않는 한 아래 작업을 수행하지 않는다.

- `git push`
- 브랜치 생성(`git branch`, `git checkout -b` 등)
- `git commit --amend`
- `git rebase`
- `git reset`
- Force push
- 원격 저장소 설정 변경(`git remote` 등)
- Merge, PR 생성
- 기존 커밋의 히스토리를 다시 쓰는 모든 작업
- 서로 다른 Git Repository(예: `ConsoleMVC-*`와 `DataPersistence-*`)의 변경을 하나의 커밋으로 묶는 작업

### 예상 커밋과 실제 커밋이 다른 경우

[PLAN.md](./PLAN.md)의 "예상 Commit"과 실제로 수행한 커밋 구성(개수, 분리 기준)이 달라지면, 그 이유를
[작업 완료 보고](#작업-완료-보고)에 함께 설명한다.

---
> Source: [minji-choe-0559/ConsoleMVC-minji-0559](https://github.com/minji-choe-0559/ConsoleMVC-minji-0559) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
