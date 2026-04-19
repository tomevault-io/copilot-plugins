## tft-defense

> - **프로젝트명**: TFT Defense

# TFT Defense 프로젝트 컨텍스트 규칙

## 프로젝트 개요
- **프로젝트명**: TFT Defense
- **엔진**: Unity
- **게임 장르**: 타워 디펜스 + 오토 체스 하이브리드
- **언어**: C# (Unity)

## 코딩 스타일 및 규칙

### 네이밍 컨벤션
- 클래스: PascalCase (예: `GameManager`, `MonsterSpawner`)
- 메서드: PascalCase (예: `SetGameState`, `StartWave`)
- 필드/프로퍼티: camelCase (예: `currentWave`, `prepareTime`)
- 이벤트: `On` 접두사 사용 (예: `OnWaveChanged`, `OnPhaseChanged`)
- 인터페이스: `I` 접두사 사용 (예: `IDamageable`, `IMonsterFieldService`)

### 아키텍처 패턴
- **싱글톤 패턴**: `MonoSingleton<T>`, `SceneSingleton<T>` 사용
- **이벤트 기반 통신**: C# Action/Event 사용
- **매니저 패턴**: 각 시스템별 Manager 클래스로 관리

### 주요 시스템 구조
1. **GameLoop**: 게임 상태 관리 (Prepare, Battle, Shop, Win, Lose)
2. **Monsters**: 몬스터 스폰, 경로, 풀링 관리
3. **Units**: 유닛 배치, 전투, 진화 시스템
4. **Economy**: 상점, 소환, 통화 관리
5. **Synergy**: 시너지 시스템
6. **Combat**: 데미지 계산, 전투 로직
7. **Battle**: 그리드, 보드 슬롯 관리

### 파일 구조 규칙
- 스크립트는 `Assets/_Project/01_Scripts/` 하위에 체계적으로 분류
- 시스템별 폴더 구조 유지 (Systems/Battle, Systems/Combat 등)
- 데이터는 `04_Data/` 폴더에 ScriptableObject로 관리

### Unity 특화 규칙
- MonoBehaviour 상속 클래스는 `MonoSingleton` 또는 일반 MonoBehaviour 사용
- 씬 종속 매니저는 `FindAnyObjectByType`으로 바인딩
- 코루틴 사용 시 null 체크 및 정리 로직 필수
- 이벤트 구독 해제는 `OnDisable`에서 처리

### 주의사항
- 씬 재로딩 시 모든 매니저 초기화 필요
- 코루틴 중복 실행 방지 (null 체크 후 시작)
- 이벤트 중복 구독 방지 (구독 해제 후 재구독)
- 싱글톤 인스턴스 null 체크 필수

## 주요 의존성
- Unity Input System
- TextMeshPro
- SPUM (스프라이트 애니메이션 시스템)
- DamageNumbersPro
- Odin Inspector (일부)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jsg9147) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
