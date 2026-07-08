## touhou-bullet-rhapsody

> > **모든 AI CLI (Gemini, Claude 포함) 는 아래 파일을 절대 직접 수정/생성/삭제하지 말 것.**

# Touhou Bullet Rhapsody — GEMINI.md

## 🚨 CLI 절대 금지 규칙

> **모든 AI CLI (Gemini, Claude 포함) 는 아래 파일을 절대 직접 수정/생성/삭제하지 말 것.**

```
금지 확장자: .unity  .prefab  .asset  .meta  .controller  .anim  .overrideController
```

- Unity 씬/프리팹/에셋은 **반드시 Unity 에디터에서만** 수정
- GUID / fileID 텍스트 직접 편집 **절대 금지**
- 빌드 세팅, 씬 추가 등도 에디터에서 직접 하거나 사용자에게 단계별 안내로 전달
- **수정 가능한 파일**: `.cs` `.md` `.txt` `.json` `.xml` 등 순수 텍스트 파일만

## 🔇 CLI 출력 규칙

- 작업 과정 혼잣말 **출력 금지** ("이미 턴을 썼으니~", "수정!", "고!", "시작!" 등)
- 완료 후 **결과만 간결하게** 출력 (변경한 파일명 + 핵심 내용 1~2줄)
- 불필요한 중간 상태 메시지, 감탄사, 진행 나레이션 모두 생략

---


## 프로젝트 개요

- **장르**: 탄막 로그라이크 (엔터 더 건전 + 아이작 스타일, 탑다운 뷰)
- **엔진**: Unity 2D (C#)
- **개발 형태**: 1인 개발
- **데모 목표**: 루미아 → 치르노 스테이지 + 간단한 엔딩
- **데모 마감**: 2026년 4월 10일

---

## 씬 구조

```
씬 0: MainMenuScene — 타이틀 + 버튼 (게임 시작 / 종료)
씬 1: StoryScene    — 인트로 스토리 (말풍선 타자기 연출, StoryDirector 제어)
씬 2: EternalScene  — 영원정 허브 (NPC 대화, 스테이지 입구, HubManager)
씬 3: SampleScene   — 실제 게임 (맵 생성, 전투, 보스)
```

> Build Settings 순서: MainMenuScene(0) → StoryScene(1) → EternalScene(2) → SampleScene(3)

---

## 핵심 아키텍처

### 재화 시스템
- **조각 (Shard)**: 유일한 영구 재화. 런 중 적 처치/방 클리어/보스 처치로 획득.
  이월됨 — 런 종료 후에도 보존. 영원정에서 강화(메인) 또는 겜블(보조)에 소비.
- **런 중 골드**: PlayerInventory.gold — 런 전용, 상점에서 소비
- **럭키코인**: **폐지됨** — 언급하지 말 것

조각 획득량:
- 잡몹 처치: EnemyData.scoreValue (1~2개)
- 방 클리어: 3개 고정 (AddRoomClear 내부 처리)
- 보스 처치: 15개 고정 (AddBossDefeat 내부 처리)

### 보스 구조
```
BossRegistry (ScriptableObject)
  └ storyBosses[] — stage번호 : BossEntry(prefab + BossData) 1:1 매핑

TilemapPainter → 보스방 RoomController에 bossRegistry 자동 주입
RoomController.TrySpawnBoss() → BossRegistry.GetStoryBoss(currentStage) → 스폰
```
보스 추가 시 BossRegistry ScriptableObject에 항목만 추가하면 됨. 씬 수정 불필요.

### 이벤트 구독 패턴
- UnityEvent 대신 C# `Action` 사용 (`Boss.OnBossDeath`, `Boss.OnPhaseChanged`)
- 오브젝트 풀링: `PoolManager` 사용

---

## 스크립트 폴더 구조

```
Assets/Scripts/
  Audio/         BossBGMController.cs
  Bullet/        BulletController.cs
  Camera/        CameraFollow.cs (클래스명 CameraFallow — 오타지만 그대로)
  Data/          GameData.cs, PermanentData.cs
  Enemy/         Boss.cs, Boss_Cirno.cs, BossData.cs, BossRegistry.cs
                 Enemy.cs, EnemyBullet.cs, EnemyData.cs
                 SandBag.cs, TankEnemy.cs
  Interfaces/    IDamageable.cs, IExplodable.cs, ITrackableEnemy.cs
  Item/          BombProjectile.cs, ItemData.cs, ItemPickup.cs
  Manager/       GameDirector.cs, GameManager.cs, EternalManager.cs (미적용)
  MapGeneration/ DoorController.cs, MapGenerator.cs, RoomInitializer.cs
                 RoomNode.cs, StageManager.cs, TilemapPainter.cs
  Player/        PlayerController.cs, PlayerHitbox.cs
                 PlayerInventory.cs, PlayerStats.cs
                 WeaponController.cs, WeaponInventory.cs
  Room/          BattleTrigger.cs, DestructibleObject.cs
                 LootTableData.cs, RewardManager.cs
                 RoomController.cs, RoomData.cs
                 NPCInteraction.cs, StagePortal.cs
  Stage/         EnemySpawner.cs, StageData.cs
  Story/         StoryScriptData.cs  — 대사 데이터 ScriptableObject
                 DialogueBubble.cs   — 말풍선 UI (타자기 효과)
                 StoryDirector.cs    — 스토리 씬 흐름 제어
                 HubManager.cs       — 영원정 허브 관리
  UI/            AmmoDisplay.cs, BossHPBar.cs, BulletTimeGauge.cs
                 DamageVignette.cs, GameOverUI.cs, HPDisplay.cs
                 InventoryUI.cs, ReloadUI.cs, SpellCardDisplay.cs
                 WeaponSlotUI.cs
  Weapon/        WeaponData.cs
  TimeManager.cs
```

---

## 파일별 주요 설계 결정

### GameData.cs (⚠️ 프로젝트에 구버전 있음 — 교체 필요)
- DontDestroyOnLoad 싱글턴
- `shards` 필드 — 조각 (구버전은 `totalScore`로 돼있음, 교체 필요)
- `shardsEarnedThisRun` — 이번 런 획득량 (결과 화면용)
- `ResetRunData()`에서 shards 초기화 안 함 (이월)
- `CalculateFinalCoins()` — 삭제됨 (구버전에 있음, 제거 필요)

### PermanentData.cs (⚠️ 프로젝트에 구버전 있음 — 교체 필요)
- static 클래스, PlayerPrefs 래퍼
- MD5 해시 변조 방지 (SECRET 키 포함)
- `PurchaseUpgrade()` — GameData.Instance.shards 직접 차감
- LuckyCoins 관련 메서드 전부 삭제됨 (구버전에 있음, 제거 필요)
- 강화 비용 (데모 기준): HP 10/20/35, 게이지 15/25/40, 골드 8/15/25 조각

### Boss.cs
- abstract 클래스
- `BossData` 주입: RoomController가 스폰 시 `boss.bossData = entry.bossData`
- `CurrentHP => currentHP` 프로퍼티로 읽기 전용 노출
- `Die()`에서 `GameData.Instance?.AddEnemyKill(score)` + `AddBossDefeat()` 호출

### RoomController.cs
- `bossSpawnPoint` 없음 — 보스는 `transform.position` (방 중앙)에 스폰
- `enemySpawner` — `[HideInInspector]`, RoomInitializer가 런타임 주입
- `bossRegistry` — TilemapPainter가 런타임 주입

### PlayerStats.cs
- `maxHP`, `currentHP`, `isDead`, `hasTakenDamage` (노다이 트래커)
- 영구 강화 수치는 EternalManager.ApplyPermanentUpgrades()에서 Start 시 반영

---

## 미적용 파일 목록 (뽑았지만 프로젝트에 아직 안 넣은 것)

다음 파일들은 설계/작성됐지만 실제 프로젝트에 아직 교체/추가 안 됨:

| 파일 | 위치 | 상태 |
|------|------|------|
| GameData.cs (shards 버전) | Assets/Scripts/Data/ | ⚠️ 교체 필요 |
| PermanentData.cs (조각 버전) | Assets/Scripts/Data/ | ⚠️ 교체 필요 |
| EternalManager.cs | Assets/Scripts/Manager/ | 🔲 신규 추가 필요 |
| UpgradeUI.cs | Assets/Scripts/UI/ | 🔲 신규 추가 필요 |
| TitleManager.cs | Assets/Scripts/Manager/ | 🔲 신규 추가 필요 |
| StoryScriptData.cs | Assets/Scripts/Story/ | ✅ 작성 완료 (Unity 세팅 필요) |
| DialogueBubble.cs | Assets/Scripts/Story/ | ✅ 작성 완료 (Unity 세팅 필요) |
| StoryDirector.cs | Assets/Scripts/Story/ | ✅ 작성 완료 (Unity 세팅 필요) |
| HubManager.cs | Assets/Scripts/Story/ | ✅ 작성 완료 (Unity 세팅 필요) |
| NPCInteraction.cs | Assets/Scripts/Room/ | ✅ 작성 완료 (Unity 세팅 필요) |
| StagePortal.cs | Assets/Scripts/Room/ | ✅ 작성 완료 (Unity 세팅 필요) |

---

## 데모 남은 작업 (우선순위 순)

1. **GameData.cs / PermanentData.cs 교체** — shards 버전으로
2. **EternalScene 씬 세팅** — 프리미티브로 빠르게 (타일맵 없이 색깔 블록)
   - 바닥 스프라이트 + 벽 BoxCollider2D
   - 에이린/카구야 NPC 오브젝트 배치
   - 스테이지 입구 오브젝트 배치
3. **Boss_Rumia.cs 구현** (3/24~3/30)
4. **스테이지 2개 연결** 루미아 → 치르노 (3/31~4/6)
5. **TitleScene 세팅** — BGM + 버튼 3개 (TitleManager.cs 사용)
6. **버그 픽스 + 데모 마무리** (4/7~4/10)
7. **[상의필요] Antigravity와 다음 작업 논의**

데모 이후 작업: 겜블방(테위), 타일맵 제대로 된 영원정, 호감도 시스템

---

## 🔄 CLI 작업 대기 목록

> Antigravity 설계 완료 후 Claude CLI에서 처리할 작업 목록.
> 완료 시 [완료 날짜]로 업데이트.

| 상태 | 작업 | 메모 |
|------|------|------|
| ✅ 2026-03-26 | StoryScene 빌드 세팅 추가 | Build Settings에 씬 4개 순서대로 등록 |
| ✅ 2026-03-26 | MainMenuScene 버튼 → StoryScene 으로 전환 연결 | 기존 "게임 시작" 버튼 씬 이름 수정 |
| ✅ 2026-03-26 | Boss_Rumia.cs 구현 완성 | [AG:설계완료] 아래 명세 보고 TODO 체우기 |
| ✅ 2026-03-26 | NPCInteraction.cs 연출 추가 | [AG:설계완료] 아래 명세 참고 |
| 🔲 대기 | GameData.cs shards 버전 교체 | [AG:설계완료] 아래 명세 참고 |
| 🔲 대기 | PermanentData.cs shards 버전 교체 | [AG:설계완료] 아래 명세 참고 |

### GameData.cs / PermanentData.cs 교체 명세 (Gemini CLI용)

**주의**: 이건 파일을 **교체**가 아니라 **수정**입니다. 기존 파일에서 변경 사항만 적용.

#### GameData.cs 변경사항
- `totalScore` 필드 → `shards` 로 개명 (타입 int 동일)
- `shardsEarnedThisRun` 필드 추가 (`public int shardsEarnedThisRun = 0;`)
- `CalculateFinalCoins()` 메서드 **삭제**
- `AddEnemyKill(int score)` 내부: `totalScore +=` → `shards +=` 로 변경
- `ResetRunData()` 내부: `shards` 초기화 **하지 않음** (이월), `shardsEarnedThisRun = 0;` 추가
- 주석 상단 summary 에서 `CalculateFinalCoins()` 언급 제거

#### PermanentData.cs 변경사항
- LuckyCoins 관련 메서드/프로퍼티 **전부 삭제**
- `PurchaseUpgrade(int cost)` 구현: `GameData.Instance.shards -= cost`
- 강화 비용 (데모 기준): HP 10/20/35, 게이지 15/25/40, 골드 8/15/25 조각

**공통 주의사항**:
- `.unity` `.prefab` `.asset` `.meta` 절대 수정 금지 — `.cs` 파일만
- `linearVelocity` 사용, `FindFirstObjectByType` 사용

### NPCInteraction 연출 명세 (Gemini CLI용)

**파일 위치**: `Assets/Scripts/Room/NPCInteraction.cs`  
**수정 방식**: 기존 코드에 기능 추가 (덮어쓰기 금지, 병합할 것)

**추가할 기능 3가지**:

1. **마주보기 (Sprite Flip)**
   - NPC가 플레이어 범위 진입 시 플레이어 방향으로 `SpriteRenderer.flipX` 설정
   - `Awake()`에서 `spriteRenderer = GetComponent<SpriteRenderer>()` 캐싱
   - `OnTriggerEnter2D`에서 `flipX = (player.position.x < transform.position.x)`

2. **말할 때 스케일 펄스**
   - 대사 출력 중(bubble.IsTyping)에만 코루틴으로 스케일 1.0 → 1.06 → 1.0 반복
   - 대사 종료 or 스킵 시 스케일 1.0으로 복귀
   - `StartCoroutine(TalkPulse())` / `StopCoroutine` 방식

3. **이모션 아이콘 팝업**
   - `[SerializeField] private GameObject emotionIcon` — Inspector에서 `!` 스프라이트 오브젝트 연결
   - 플레이어 범위 진입 시 `emotionIcon.SetActive(true)` → 대화 시작 후 숨김
   - 첫 대사 시작할 때 `emotionIcon.SetActive(false)`

**주의사항**:
- `.unity` `.prefab` `.asset` `.meta` 절대 수정 금지 — `.cs` 파일만
- `linearVelocity` 사용 (`velocity` deprecated)
- `FindFirstObjectByType` 사용 (`FindObjectOfType` deprecated)
- 기존 E키 대화 로직 건드리지 말고 연출만 추가

### Boss_Rumia 구현 명세 (Gemini CLI용)

**파일 위치**: `Assets/Scripts/Enemy/Boss_Rumia.cs` (스켈레톤 작성 완료)

**레퍼런스**: `Boss_Cirno.cs` 의 `IcicleStrike()`, `FireBullet()` 패턴 참고

**TODO 목록**:
1. `Pattern_DarkSpread()` — 방사형 N발 로직 완성 (Cirno FreezeBurst 참고)
2. `Pattern_ShadowBolt()` — FireBullet 참조 코드 주석 해제
3. `Pattern_NightRain()` — RainStrike 코루틴 구현 (Cirno IcicleStrike 참고)
4. `RainStrike(Vector2 pos)` 코루틴 신규 작성 — 예고서클 → 딜레이 → 낙하탄

**주의사항**:
- `.unity` `.prefab` `.asset` `.meta` 등 Unity 직렬화 파일 **절대 수정 금지**
- FireBullet은 `base.FireBullet(darkBulletPrefab, transform.position, angle, speed)` 므싙
- 시야 제한은 이미 설계 완료, Light2D Inspector 세팅은 유저가 Unity에서 직접 수행

---

## 코딩 컨벤션

- C# Action 사용 (UnityEvent 사용 안 함)
- 싱글턴: `public static XXX Instance { get; private set; }` + Awake에서 처리
- `Rigidbody2D.linearVelocity` 사용 (`velocity` deprecated — Unity 2022.2+)
- `FindFirstObjectByType` 사용 (`FindObjectOfType` deprecated)
- ScriptableObject 기반 데이터 분리 (BossData, BossRegistry, StageData 등)
- 오브젝트 풀링: PoolManager 사용
- Header 어트리뷰트로 Inspector 섹션 구분

---

## 노션 문서

- 프로젝트 루트: https://www.notion.so/TouhouBulletRphasody-3252391f7a83808cac01ef9e236327a7
- DevLog: https://www.notion.so/3252391f7a8381069afdee4cadc1f3c1
- 게임 전체 설정: https://www.notion.so/3252391f7a83815291f9e0bb123bcf90
- 시스템 기획서: https://www.notion.so/3262391f7a8381c99dbaffd7259c2499
- 조각 재화 시스템: https://www.notion.so/3282391f7a8381f2b6edd6705943adb9
- 테위 겜블방: https://www.notion.so/3282391f7a838155847dd032077a604d
- 테위 대사집: https://www.notion.so/3282391f7a83818f9553fd123df0a2e5
- 대사집: https://www.notion.so/3262391f7a838149a548c866dd3316a8

---
> Source: [Pinut1/TouHou_-Bullet_Rhapsody](https://github.com/Pinut1/TouHou_-Bullet_Rhapsody) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
