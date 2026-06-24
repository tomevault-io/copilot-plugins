## car-assembly-24024224

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# 빌드
./gradlew build          # Windows: gradlew.bat build

# 테스트
./gradlew test

# 클린 빌드
./gradlew clean build

# 단일 테스트 클래스 실행
./gradlew test --tests "ClassName"

# 앱 실행 (컴파일 후)
java -cp build/classes/java/main Assemble
```

## Architecture

단일 Java 클래스(`src/main/java/Assemble.java`)로 구성된 콘솔 기반 자동차 조립 시뮬레이터.

**상태 머신 구조:** 5단계 메뉴를 순서대로 진행하며 사용자 선택을 `stack[5]` 배열에 저장한다.
- State 0: 차종 선택 (SEDAN/SUV/TRUCK)
- State 1: 엔진 선택 (GM/TOYOTA/WIA/Broken)
- State 2: 브레이크 선택 (MANDO/CONTINENTAL/BOSCH)
- State 3: 스티어링 선택 (BOSCH/MOBIS)
- State 4: Run 또는 Test 실행

**비즈니스 규칙 (`isValidCheck`):**
- SEDAN은 Continental 브레이크 사용 불가
- SUV는 Toyota 엔진 사용 불가
- TRUCK은 WIA 엔진, Mando 브레이크 사용 불가
- Bosch 브레이크는 Bosch 스티어링만 허용

**Tech Stack:** Java (표준 라이브러리만 사용), Gradle 9.3.0 (Kotlin DSL), JUnit 5 (테스트 코드 없음)

현재 패키지 구조 없음 — `Assemble` 클래스가 default package에 위치.

## 리팩토링 플랜

진행 중인 리팩토링 계획은 [`docs/PLAN.md`](docs/PLAN.md) 참고.

---
> Source: [insgahn/car-assembly-24024224](https://github.com/insgahn/car-assembly-24024224) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
