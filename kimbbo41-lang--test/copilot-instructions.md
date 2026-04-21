## test

> - 콘솔 텍스트 기반 턴제 RPG 게임 (C++17)

# Dungeon Adventure - 프로젝트 규칙

## 프로젝트 개요
- 콘솔 텍스트 기반 턴제 RPG 게임 (C++17)
- Visual Studio / MSVC 빌드
- 한글 UI

## 빌드 방법
- Visual Studio에서 `test1.sln`을 열고 빌드 (Ctrl+B)
- 또는 CLI: `msbuild test1.sln /p:Configuration=Debug`

## 한글 인코딩 규칙
- 모든 소스 파일은 **UTF-8 with BOM** 으로 저장
- `main()` 시작 시 반드시 로케일 설정:
  ```cpp
  #include <locale>
  #include <io.h>
  #include <fcntl.h>
  // main() 첫 줄:
  _setmode(_fileno(stdout), _O_U16TEXT);
  // 또는:
  SetConsoleOutputCP(65001);
  ```
- `wcout` 또는 `cout` 중 하나로 통일 (프로젝트에서는 `cout` + `SetConsoleOutputCP(65001)` 사용)

## 코딩 규칙 요약
- 상세 컨벤션은 `docs/coding_convention.md` 참조
- 네이밍: 클래스 `PascalCase`, 함수 `camelCase`, 변수 `camelCase`, 상수 `UPPER_SNAKE`
- 헤더 가드: `#pragma once`
- 들여쓰기: 4칸 스페이스
- 동적 메모리: `std::unique_ptr` / `std::shared_ptr` 사용 (raw `new`/`delete` 금지)

## 파일 구조
```
test1/
├── CLAUDE.md              # 이 파일
├── docs/
│   ├── team/              # 팀 역할 정의
│   └── coding_convention.md
├── Common.h               # 공통 타입, 열거형
├── GameConfig.h           # 게임 설정값
├── Character.h/cpp        # 캐릭터 시스템
├── Item.h/cpp             # 아이템
├── Inventory.h/cpp        # 인벤토리
├── Monster.h/cpp          # 몬스터
├── Skill.h/cpp            # 스킬
├── Combat.h/cpp           # 전투 시스템
├── Dungeon.h/cpp          # 던전
├── Game.h/cpp             # 게임 루프
├── UI.h/cpp               # 텍스트 UI 유틸리티
└── test1.cpp              # main() 진입점
```

## 팀 협업 규칙
- 에이전트 간 모듈 의존성 문제 발생 시 당사자끼리 먼저 협의
- 의견 충돌 시 설계자(메인 에이전트)가 최종 결정
- 상세 팀 구성은 `docs/team/` 참조

## 주의사항
- `test1.vcxproj`에 새 .cpp/.h 파일 추가 시 반드시 등록할 것
- `.vs/` 폴더는 git에 포함하지 않음
- 커밋 메시지는 한글로 작성

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kimbbo41-lang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
