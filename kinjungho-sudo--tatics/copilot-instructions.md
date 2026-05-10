## tatics

> > Claude Code가 이 파일을 가장 먼저 읽는다.

# CLAUDE.md — SRPG 모바일 게임 프로젝트

> Claude Code가 이 파일을 가장 먼저 읽는다.
> 모든 작업 전에 이 파일 전체를 숙지하고 시작할 것.

---

## 프로젝트 개요

- **장르**: SRPG (시뮬레이션 롤플레잉 게임)
- **레퍼런스**: 파랜드 택틱스 1·2 (시스템 참고, 콘텐츠 오리지널)
- **플랫폼**: 모바일 (Phaser.js → Capacitor → Android/iOS)
- **개발 방식**: Claude Code 주도 / 정호님 기획·QA·밸런싱

---

## 검증된 핵심 로직 (수정 금지)

아래 로직은 프로토타입에서 플레이테스트 완료된 것들이다.
**반드시 동일한 방식으로 구현할 것. 임의 변경 금지.**

### 이동 범위 계산 (BFS)
```javascript
// 너비 우선 탐색 — terrain cost 포함
// 평지: cost 1 / 숲: cost 2 / 장애물: 통과 불가
// 아군 유닛 위치: 통과 불가 / 적군 위치: 이동 목적지 불가
function calcMoveRange(unit) { ... }
```

### 공격 범위 계산 (맨해튼 거리)
```javascript
// dist = |ax - bx| + |ay - by|
// dist <= unit.range 이면 공격 가능
function calcAtkRange(unit, targetTeam) { ... }
```

### 데미지 공식
```javascript
// 최소 1 보장 + 랜덤 편차
damage = max(1, attacker.atk - defender.def) + random(0, 5)
```

### 경험치 & 레벨업
```javascript
// 공격 성공: +5 EXP / 적 처치: +15 EXP 추가
// 레벨업 필요 EXP = 현재레벨 × 20
// 성장: 직업별 growthTable 기준 + 50% 확률 추가 +1 (파랜드 스타일)
```

### 적 AI (기본)
```javascript
// 1. SPD 높은 적부터 행동
// 2. 가장 가까운 아군 타겟 선정
// 3. 공격 가능하면 즉시 공격
// 4. 불가능하면 타겟 방향으로 최대 이동 후 재공격 시도
```

---

## 스탯 시스템

### 유닛 기본 스탯 구조
```javascript
{
  id, name, team,        // 식별자
  x, y,                 // 격자 좌표
  lv, exp,              // 레벨, 경험치
  hp, maxHp,            // 체력
  atk, def,             // 공격력, 방어력
  spd, move, range,     // 속도, 이동력, 사거리
  acted                 // 이번 턴 행동 완료 여부
}
```

### 직업별 기본 스탯
| 직업   | HP  | ATK | DEF | SPD | MOVE | RNG |
|--------|-----|-----|-----|-----|------|-----|
| 기사   | 100 | 30  | 15  | 8   | 3    | 1   |
| 마법사 | 70  | 45  | 5   | 12  | 2    | 2   |
| 궁수   | 80  | 25  | 10  | 10  | 3    | 3   |
| 성직자 | 75  | 15  | 8   | 9   | 2    | 1   |
| 기병   | 90  | 28  | 8   | 11  | 5    | 1   |

### 직업별 성장 테이블 (레벨업 시 기본 증가량)
| 직업   | HP | ATK | DEF | SPD |
|--------|----|-----|-----|-----|
| 기사   | 8  | 2   | 3   | 0   |
| 마법사 | 4  | 4   | 1   | 1   |
| 궁수   | 5  | 3   | 2   | 1   |
| 성직자 | 5  | 1   | 2   | 1   |
| 기병   | 6  | 3   | 1   | 1   |

---

## 지형 시스템

```javascript
const TERRAIN = {
  0: { name: '평지', moveCost: 1, defBonus: 0 },
  1: { name: '장애물', moveCost: Infinity, defBonus: 0 },
  2: { name: '숲', moveCost: 2, defBonus: 10 },  // 방어 보너스 % (향후)
  3: { name: '강', moveCost: 3, defBonus: -5 },  // 향후 추가
  4: { name: '산', moveCost: Infinity, defBonus: 20 }, // 향후 추가
}
```

---

## 턴 시스템

```
[아군 턴]
  └─ 유닛 선택 → 이동 → 공격 (또는 대기)
  └─ 전원 행동 완료 or 수동 종료
[적 턴]
  └─ SPD 순 자동 행동 (이동 + 공격)
[다음 아군 턴]
  └─ 모든 유닛 acted = false 초기화
  └─ turnCount++
```

---

## 프로젝트 파일 구조

```
srpg-game/
├── CLAUDE.md                  ← 이 파일 (항상 루트에 위치)
├── package.json
├── capacitor.config.ts        ← 모바일 빌드 설정
│
├── src/
│   ├── main.js                ← Phaser 게임 진입점
│   ├── config.js              ← 게임 전역 설정 (타일 크기, 해상도 등)
│   │
│   ├── scenes/                ← Phaser Scene 단위
│   │   ├── BootScene.js       ← 에셋 로딩
│   │   ├── TitleScene.js      ← 타이틀 화면
│   │   ├── MapScene.js        ← 스테이지 선택
│   │   └── BattleScene.js     ← 핵심 전투 씬 (가장 중요)
│   │
│   ├── systems/               ← 게임 로직 (씬 독립적)
│   │   ├── MoveSystem.js      ← BFS 이동 범위 계산
│   │   ├── AttackSystem.js    ← 데미지, 공격 범위, 반격
│   │   ├── TurnSystem.js      ← 턴 관리, 행동 순서
│   │   ├── AISystem.js        ← 적 AI 로직
│   │   └── ExpSystem.js       ← 경험치, 레벨업, 성장
│   │
│   ├── data/                  ← 순수 데이터 (로직 없음)
│   │   ├── units.js           ← 직업별 기본 스탯 정의
│   │   ├── growthTable.js     ← 레벨업 성장 테이블
│   │   ├── stages/            ← 스테이지별 맵 + 적 배치
│   │   │   ├── stage01.js
│   │   │   └── stage02.js
│   │   └── terrain.js         ← 지형 이동 코스트, 보너스
│   │
│   ├── ui/                    ← HUD, 패널, 팝업
│   │   ├── StatPanel.js       ← 유닛 스탯 표시
│   │   ├── TurnBadge.js       ← 턴 표시 배지
│   │   ├── OrderPanel.js      ← 행동 순서 패널
│   │   ├── DamagePopup.js     ← 데미지 숫자 팝업
│   │   └── LevelUpPopup.js    ← 레벨업 연출
│   │
│   └── utils/
│       ├── GridUtils.js       ← 좌표 변환, 맨해튼 거리
│       └── Random.js          ← 랜덤 유틸 (시드 기반 향후)
│
├── assets/
│   ├── sprites/               ← 유닛 스프라이트 (정호님 제작/구매)
│   │   ├── knight.png
│   │   ├── mage.png
│   │   └── archer.png
│   ├── tilesets/              ← 맵 타일셋
│   │   └── field.png
│   └── ui/                    ← UI 이미지 리소스
│
└── android/                   ← Capacitor 자동 생성 (수정 금지)
└── ios/                       ← Capacitor 자동 생성 (수정 금지)
```

---

## BattleScene.js 핵심 구조 (구현 순서)

```javascript
class BattleScene extends Phaser.Scene {
  // 1. 초기화
  create() {
    this.map = this.createMap();       // 타일맵 생성
    this.units = this.spawnUnits();    // 유닛 배치
    this.turnSystem = new TurnSystem(this);
    this.ui = this.createUI();
  }

  // 2. 입력 처리 (터치/클릭)
  onTileClick(tileX, tileY) {
    if (phase === 'idle')    → 유닛 선택
    if (phase === 'selected') → 이동 or 다른 유닛 선택
    if (phase === 'moved')   → 공격 or 취소
  }

  // 3. 턴 흐름
  endAllyTurn()   → AISystem.runEnemyTurn() → startAllyTurn()
}
```

---

## 개발 규칙 (Claude Code 필독)

### 절대 규칙
1. **검증된 핵심 로직은 수정하지 않는다** — BFS, 데미지 공식, EXP 공식
2. **data/ 폴더는 순수 데이터만** — 로직이 섞이면 안 됨
3. **systems/ 폴더는 Phaser 의존성 없이** — 순수 JS 클래스로 작성
4. **한 번에 하나의 시스템만 작업** — 병렬 수정 금지

### 코딩 컨벤션
```javascript
// 파일명: PascalCase (MoveSystem.js)
// 함수명: camelCase (calcMoveRange)
// 상수:   UPPER_SNAKE (CELL_SIZE, MAX_LEVEL)
// 주석:   한국어로 작성 (정호님이 읽기 위함)
```

### 작업 완료 체크리스트
작업 완료 후 반드시 확인:
- [ ] 이동 범위가 장애물을 피하는가
- [ ] 데미지 공식이 min 1을 보장하는가
- [ ] 턴 종료 후 acted가 초기화되는가
- [ ] 레벨업 시 maxHp도 함께 증가하는가
- [ ] 모바일 터치 이벤트가 클릭과 동일하게 작동하는가

---

## 현재 개발 단계

- [x] Phase 0: 프로토타입 (순수 JS + Canvas) — 완료
- [ ] Phase 1: Phaser.js 마이그레이션 — **현재 단계**
- [ ] Phase 2: 스테이지 5개 + UI 완성 (MVP)
- [ ] Phase 3: Capacitor 모바일 빌드
- [ ] Phase 4: Google Play 출시

---

## 오케스트레이터 (총괄 매니저) 운영 규칙

Claude Code가 이 프로젝트의 **총괄 매니저**로 동작한다.
정호님의 지시를 받아 아래 6개 에이전트에게 작업을 분배하고 전체 흐름을 관리한다.

### 에이전트 구성 (.claude/agents/)
| 에이전트 | 담당 | 병렬 가능 |
|----------|------|---------|
| `setup-agent` | package.json, vite.config.js, main.js, config.js | 없음 (1회성, 최선행) |
| `data-agent` | src/data/ 전체 | systems-agent와 병렬 OK |
| `systems-agent` | src/systems/, src/utils/ | data-agent와 병렬 OK |
| `scene-agent` | src/scenes/ 전체 | data+systems 완료 후 |
| `ui-agent` | src/ui/ 전체 | scene 뼈대 완료 후 |
| `qa-agent` | tests/, 버그 수정 | "검증 해줘" 트리거 시 |

### 실행 시퀀스
```
[1] setup-agent (단독)
         ↓
[2] data-agent + systems-agent (병렬)
         ↓
[3] scene-agent
         ↓
[4] ui-agent
         ↓
[5] qa-agent ("검증 해줘")
```

### 총괄 매니저 행동 원칙
1. 정호님 지시 → 어느 에이전트 담당인지 판단 → 해당 에이전트 명세서(.claude/agents/*.md) 참조
2. 병렬 작업 가능한 경우 반드시 병렬로 실행
3. 에이전트 간 파일 충돌 방지 — 각 에이전트는 자기 담당 디렉토리만 수정
4. 완료 후 qa-agent 또는 관련 스킬로 검증
5. 실수 발생 시 .claude/mistakes/ 자동 기록

### 활용 스킬 목록
```
phaser, game-architecture, game-assets, game-designer, game-qa  ← reference
game:add-feature, game:balance-check, game:team-combat          ← 게임 커맨드
game:team-level, game:qa-game, game:review-game                 ← 게임 커맨드
srpg:srpg-balance, srpg:stage-builder, srpg:unit-check          ← SRPG 전용
dev:tdd-cycle, dev:performance-optimization                     ← 개발 도구
simplify                                                        ← 코드 정리
```

---

## 다음 작업 명령어 (정호님이 Claude Code에 입력할 것)

```
Phase 1 시작:
"setup-agent를 실행해서 Phaser.js 프로젝트 초기 세팅을 해줘."
```

---

*마지막 업데이트: 2026.04.02*
*작성: 정호 + Claude Code (총괄 매니저)*

---
> Source: [kinjungho-sudo/tatics](https://github.com/kinjungho-sudo/tatics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
