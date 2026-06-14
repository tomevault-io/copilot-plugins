## voice-spell

> 이 파일은 Claude(또는 다른 LLM 코드 어시스턴트)가 본 프로젝트에 처음 진입했을 때 빠르게

# Voice Spell Multiplayer — CLAUDE.md

이 파일은 Claude(또는 다른 LLM 코드 어시스턴트)가 본 프로젝트에 처음 진입했을 때 빠르게
컨텍스트를 잡고 일관된 스타일로 코드를 추가/수정할 수 있도록 작성된 가이드입니다.

본 가이드는 `Assets/02. Script/` 폴더의 실제 파일을 분석해 추출한 **실제 코드 패턴** 기반이며,
추측이 아닙니다. 새 파일을 작성할 때는 이 문서의 컨벤션과 가장 가까운 기존 파일을 먼저
참고하세요.

---

## 0. 이 게임은 무엇인가 (한 줄 정의)

> **약한 음성 마법 1개로 시작해, 맵의 몬스터를 사냥하며 새 키워드를 배우고 기존 스킬을
> 강화해 자기만의 마법 빌드를 만들어가는 4인 FFA 음성 마법 배틀로얄. 강해질수록 외쳐야
> 할 주문이 늘어나고, 그 발화는 다른 플레이어에게 그대로 들리기에 빌드의 깊이가 곧
> 노출의 위험이 된다.**

상세 디자인 문서: 프로젝트 루트의 `GAME_DESIGN.md` 참조. 본 CLAUDE.md 는 코드 컨벤션
중심, GAME_DESIGN.md 는 게임 디자인 헌법.

### 0.1 핵심 게임플레이 루프

```
[라운드 시작 전: 시작 마법 선택 (화염구 / 얼음계)]
                      ↓
[Phase 1 — 사냥/성장 (조용한 시기)]
  슬라임 처치 → 크리스탈 드롭 → 줍기 → 레벨업
  레벨업 시 3개 옵션 중 1개 선택 (스킬 강화)
                      ↓
[Phase 2 — 조우/심리전 (시끄러운 시기)]
  강해진 자는 자주 외쳐야 함 → 더 자주 들킴
                      ↓
[Phase 3 — 결착]
  마지막 한 명 남을 때까지 PvP, 사망 시 관전
```

### 0.2 핵심 설계 원칙 (GAME_DESIGN.md §4 "게임 정체성" 발췌)

1. **시전 = 의사소통** — 한 행위가 두 가지 목적을 동시 수행. 키 누르는 게임에선 불가능한
   장면이 가능 ("화염구!" 외친 직후 농담 한 마디 → 상대 방심).
2. **빌드의 깊이 = 위험의 깊이** — 강해질수록 외쳐야 할 주문이 길어지고 더 자주 들킴.
3. **헤드폰의 가치** — 거리 감쇠만으로 적의 위치를 추정. 시각 보조(미니맵, 화살표) 없이
   귀로만 판단 → 몰입감/긴장감 자동 발생.
4. **음성 메커니즘 = 자기장 대체** — 강해질수록 시끄러워지므로 후반에 자연스럽게 충돌
   빈도 증가. 인위적 자기장 없이도 라운드 페이싱이 잡힘.

이 4가지 중 하나라도 약화시키는 디자인은 거부한다 (자동 시전, 미니맵 적 표시, 텍스트
입력 백업 모드 등).

### 0.3 버전별 스코프 (현재 v1 데모 작업 중)

| 버전 | 스코프 | 목표 |
|------|--------|------|
| **v1 (데모)** | 4명 FFA, 5분 라운드, 슬라임 1종, 시작 마법 2종(화염/얼음), 강화 키워드 학습, HP 회복 없음 | 핵심 메커니즘이 *재미있는지* 검증 (2-3주) |
| v2 (확장) | 보스 + 자기장 + 다양 잡몹 + 시작 마법 4종 + 새 키워드 학습 + 맵 다수 | 콘텐츠 풍부화 |
| v3 (출시) | 계정/랭킹 + 욕설 필터 + 신고 + 통계 + Photon 유료 | 출시 준비 |

**작업 시작 전 항상 GAME_DESIGN.md §6.10 (데모에서 제외하는 것) 확인.**

### 0.4 기술 스택 한눈에

| 영역 | 사용 기술 | 위치 |
|------|----------|------|
| 멀티플레이 | Photon PUN 2 | `Assets/Photon/PhotonUnityNetworking/` |
| 음성 채팅 | Photon Voice 2 | `Assets/Photon/PhotonVoice/` |
| 음성 인식 (STT) | whisper.unity (whisper.cpp 래퍼) | Package, 모델 = `StreamingAssets/Whisper/` |
| 입력 | Unity New Input System | PlayerInput 컴포넌트 |
| UI | TextMesh Pro | 코드로 즉석 생성 |
| 엔진 | Unity 6 | `Rigidbody.linearVelocity` 사용 |

---

## 1. 프로젝트 개요

- **씬 흐름**: `IntroScene` (이름 입력 → PhotonNetwork.NickName 설정) → `MainScene` (게임플레이)
- **스폰**: `MainScene` 의 `GameManager` 가 `PhotonNetwork.Instantiate("Player", …)` 호출
- **스킬 종류 (현재 4개)**:
  - 화염구 (발사체, 25 데미지, 마나 20)
  - 치유 (즉시시전, +30 HP, 마나 30)
  - 방패 (지속형, 4초간 본인 보호, 마나 35)
  - 번개 (발사체, 40 데미지·고속, 마나 40)
- **HP/MP**: 100/100, MP 는 0.5초마다 +2.5 자동 회복

---

## 2. 폴더 구조

```
Assets/02. Script/
├── Camera/        — 3인칭 궤도 카메라
├── Common/        — Constants, GameManager, NetworkManager, ICharacterState, CharacterUtility
├── Player/        — PlayerController, PlayerHealth, PlayerMana, PlayerNameTag, SfxBroadcaster
│   ├── SMB/       — StateMachineBehaviour 들 (애니메이터 콜백)
│   └── State/     — PlayerState 베이스 + Idle/Move/Jump (FSM)
├── Spells/        — Fireball, HealSpell, ShieldSpell (모두 MonoBehaviourPun)
├── UI/            — IntroNameInput, PlayerHealthUI (코드로 즉석 생성)
└── Voice/         — VoiceSpellCaster, SpellDatabase, SpellEntry, SpellPhraseMatcher, HangulUtil
```

리소스 폴더 규칙:

```
Assets/Resources/
├── Player.prefab            — PhotonNetwork.Instantiate("Player", …)
├── Fireball.prefab          — Spells (PhotonNetwork.Instantiate)
├── Heal.prefab
├── Shield.prefab
├── Lightning.prefab
└── SpellDatabase.asset      — SpellDatabase.GetRuntime() 가 Resources.Load 로 로드
```

`Resources/`에 둔 이유: Photon `Instantiate` 의 prefab 이름 lookup, 그리고 SpellDatabase
런타임 로드(클라이언트 간 데이터 일관성).

---

## 3. 코드 스타일 컨벤션

### 3.1 Line ending: 저장소는 LF 통일, 워킹 디렉토리는 OS별

**Git 저장소 기준으로 모든 .cs 파일은 LF로 통일됨** (`.gitattributes` + `core.autocrlf=true`).
Windows 워킹 디렉토리에서는 일부 파일이 CRLF로 보일 수 있으나 이는 Windows 기본 동작이며,
git이 커밋/체크아웃 시 자동 변환하므로 사실상 통일된 상태와 동일하다.

확인 명령:
```bash
git ls-files --eol Assets/02.\ Script/Common/Constants.cs
# 결과 예: i/lf  w/crlf  attr/text eol=lf  →  index가 LF면 정규화 완료
```

**새 파일 작성 시**: IDE가 `.editorconfig` 의 `end_of_line = lf` 를 따르므로 자동으로 LF.
신경 쓸 필요 없음.

### 3.1.1 코드 *작성 스타일* 두 종 — 이건 별개

(line ending 과 무관) 본 프로젝트의 .cs 파일은 **작성 스타일 두 종이 공존**한다:

| 영역 | 작성 스타일 |
|------|------------|
| PlayerController, PlayerState*, NetworkManager, GameManager, Constants, ICharacterState, PlayerSmb*, CharacterUtility | 한글 인라인 주석 위주, XML doc 거의 없음 (프로젝트 원작자 스타일) |
| PlayerHealth, PlayerMana, PlayerNameTag, SfxBroadcaster, PlayerHealthUI, IntroNameInput, CameraController, Voice/*, Spells/* | `/// <summary>` XML doc + Header 그룹화 (후속 추가/리팩토링 스타일) |

**새 파일 작성 시**: 같은 폴더의 기존 파일 스타일을 따라가세요. 둘 중 어느 쪽이든
일관성이 더 중요합니다. 모르겠으면 LF + XML doc summary 를 기본으로 하세요.

### 3.2 공통 컨벤션 (양쪽 다 지켜짐)

#### 명명

- **클래스/메서드/Public property**: `PascalCase` (`PlayerHealth`, `CurrentHealth`, `TrySpend`)
- **private 필드**: `_camelCase` (underscore prefix) — `_animator`, `_states`, `_isRecording`
- **`[SerializeField] private`** 필드: underscore 없이 camelCase — `maxHealth`, `currentHealth`,
  `spawnForwardOffset` (인스펙터에 그대로 노출됨)
- **public field (소수만)**: `EPlayerState State;` 처럼 짧고 의도된 곳에만 사용
- **enum**: `E` prefix — `EPlayerState.Idle`, `EPlayerState.Move`
- **상수**: `Constants` 클래스의 `static readonly` 또는 `const` — `Gravity`, `PlayerAniParamMove`

#### 인스펙터 노출

```csharp
[Header("체력")]
[Tooltip("최대 HP. 시작 시 currentHealth 도 이 값으로 초기화.")]
[SerializeField] private int maxHealth = 100;

public int MaxHealth => maxHealth;   // 외부 read-only 노출은 expression-bodied property
```

- `[Header("…")]` 로 그룹화 (한글 가능)
- `[Tooltip("…")]` 로 인스펙터 설명
- `[Range(min, max)]` 로 슬라이더
- 외부에 읽기 전용으로 노출할 때는 `=>` expression-bodied property 사용

#### 컴포넌트 캐싱

```csharp
private Animator _animator;
private PlayerInput _playerInput;
private CharacterController _characterController;

private void Awake()
{
    _animator = GetComponent<Animator>();
    _playerInput = GetComponent<PlayerInput>();
    _characterController = GetComponent<CharacterController>();
}
```

`Update()` 안에서 `GetComponent` 호출은 금지. 항상 `Awake/OnEnable` 에서 캐싱.

#### MonoBehaviour 라이프사이클 분담

- `Awake()`: 컴포넌트 캐싱, `IsMine` 체크 후 `enabled = false`, FSM 딕셔너리 초기화
- `OnEnable()`: 상태 리셋(예: `currentHealth = maxHealth`), 이벤트 구독, 본인 한정 UI 생성
- `Start()`: 코루틴, 외부 검색(`Camera.main`, `FindFirstObjectByType`)
- `Update()`: 입력 처리, FSM tick, `IsMine` 가드
- `LateUpdate()`: 카메라 추적, 빌보드 회전
- `OnDisable()`: 진행 중인 작업 정리(녹음 중단, TransmitEnabled=false 등)

---

## 4. Photon (PUN 2) 패턴

### 4.1 컴포넌트 베이스

```csharp
[RequireComponent(typeof(PhotonView))]
public class PlayerHealth : MonoBehaviourPun
{
    // photonView 는 베이스에서 자동 주입됨 — 별도 GetComponent 불필요
}
```

- 거의 모든 게임플레이 컴포넌트는 `MonoBehaviourPun` 또는 `MonoBehaviourPunCallbacks`(NetworkManager) 상속
- `PhotonView` 는 항상 `[RequireComponent]` 로 강제

### 4.2 IsMine 가드 — 3가지 패턴

```csharp
// 1) 컴포넌트 자체를 끔 (입력/시전 처리 — 본인만 동작)
private void Awake()
{
    if (!photonView.IsMine) enabled = false;
}

// 2) 메서드 진입에서 early return
private void Update()
{
    if (!photonView.IsMine) return;
    _states[State].Update();
}

// 3) 분기 (위치 동기화는 모두 적용, 물리는 소유자만)
if (photonView.IsMine)
    _rb.linearVelocity = transform.forward * speed;
else
    _rb.isKinematic = true;
```

### 4.3 RPC

```csharp
[PunRPC]
public void RPC_TakeDamage(int damage, int attackerActorNumber, PhotonMessageInfo info)
{
    // ...
}

// 호출
otherPv.RPC("RPC_TakeDamage", RpcTarget.All, damage, _casterActorNumber);
```

- RPC 메서드는 **`RPC_` prefix + PascalCase**
- 모든 클라이언트 상태 동기화는 `RpcTarget.All`
- 인자는 primitive + `Vector3` + string 만 (Photon 직렬화 한계)
- 마지막 인자로 `PhotonMessageInfo info` 추가하면 송신자/타임스탬프 확인 가능

### 4.4 InstantiationData 로 데이터 주입

발사체·주문 프리팹은 `PhotonNetwork.Instantiate(prefab, pos, rot, group, initData)` 로 스폰.
받는 쪽은 `Awake` 에서 읽음.

**컨벤션**: `[string keyword, int casterPhotonViewID, int casterActorNumber]`

```csharp
// 송신 (VoiceSpellCaster.CastSpell)
var initData = new object[] { entry.keyword, photonView.ViewID, photonView.OwnerActorNr };
PhotonNetwork.Instantiate(entry.prefabName, spawnPos, spawnRot, 0, initData);

// 수신 (Fireball.Awake)
var data = photonView != null ? photonView.InstantiationData : null;
if (data != null)
{
    if (data.Length >= 1 && data[0] is string s) _keyword = s;
    if (data.Length >= 2 && data[1] is int v1) _casterViewId = v1;
    if (data.Length >= 3 && data[2] is int v2) _casterActorNumber = v2;
}
```

길이/타입 체크는 항상 `is` 패턴 매칭으로 안전하게.

### 4.5 SpellDatabase 런타임 로드

발사체/주문 프리팹은 인스펙터로 SpellDatabase 참조를 받지 못하므로 (런타임 인스턴스화)
싱글톤성 헬퍼 사용:

```csharp
public static SpellDatabase GetRuntime()
{
    if (_runtimeCache != null) return _runtimeCache;
    _runtimeCache = Resources.Load<SpellDatabase>("SpellDatabase");
    if (_runtimeCache == null)
        Debug.LogError("[SpellDatabase] Resources/SpellDatabase.asset 을 찾을 수 없습니다.");
    return _runtimeCache;
}
```

### 4.6 PunVoiceClient 는 씬에 미리 배치

`MainScene` 루트에 `PunVoiceClient` GameObject 가 있어야 함. 옵션:
- `UsePunAppSettings = true` (PUN AppId/Region 자동 재사용)
- `UsePunAuthValues = true`
- `AutoConnectAndJoin = true` (PUN 룸 입장 시 보이스도 자동 합류)

씬에 없으면 "PunVoiceClient component was not found in the scene. Creating PunVoiceClient
object." 라는 경고가 뜨고 자동 생성되지만, AppId 가 비어있을 위험 있음.

---

## 5. Photon Voice 2 통합 패턴

### 5.1 Define 가드

```csharp
#if WHISPER_UNITY
using Whisper;
#endif
#if PHOTON_VOICE_DEFINED
using Photon.Voice.Unity;
using Photon.Voice.PUN;
#endif
```

(Project Settings → Scripting Define Symbols 에 `WHISPER_UNITY`, `PHOTON_VOICE_DEFINED` 등록)

### 5.2 Player 프리팹의 Voice 컴포넌트

Player 프리팹 root 에 다음이 모두 부착되어 있어야 함:
- `Recorder` — 마이크 캡처 + 송출 (옵션: `MicrophoneType = Photon` native, `SamplingRate = 16000`)
- `PhotonVoiceView` — PhotonView 와 자동 연결, voice frame 라우팅
- `Speaker` — 다른 플레이어의 음성을 본인 위치에서 3D 재생

### 5.3 Recorder 토글 (우클릭 홀드 동안만 송출)

```csharp
#if PHOTON_VOICE_DEFINED
_voiceRecorder = GetComponent<Recorder>();
if (_voiceRecorder != null)
{
    _voiceRecorder.TransmitEnabled = false;   // 시작 시 송출 차단
    _voiceRecorder.RecordingEnabled = true;   // 마이크는 워밍업 유지
}
#endif

private void StartRecording()
{
#if PHOTON_VOICE_DEFINED
    if (_voiceRecorder != null) _voiceRecorder.TransmitEnabled = true;
#endif
}
```

`OnDisable()` 에서 `TransmitEnabled = false` 로 송출 차단 잊지 말 것.

### 5.4 STT 와 보이스의 마이크 점유

- whisper STT 는 `Microphone.Start(device, …)` (Unity API)
- Photon Voice 는 `MicrophoneType = Photon` (native, Photon 자체 핸들)
- 같은 device 라도 핸들이 분리되어 있어 보통 충돌 없음. 드라이버 일부 환경에서 STT 가 빈
  클립을 받는 경우가 있음 — 이때는 `MicrophoneType = Unity` 로 바꾸고 IProcessor 통합 방식
  필요 (현재는 미적용).

---

## 6. FSM (Player State Machine)

`PlayerController` 가 `Dictionary<EPlayerState, ICharacterState>` 를 들고 위임:

```csharp
public interface ICharacterState
{
    void Enter();
    void Update();
    void Exit();
}

public class PlayerState   // 베이스: 공유 필드, 공유 메서드
{
    protected PlayerController _playerController;
    protected Animator _animator;
    protected PlayerInput _playerInput;
}

public class PlayerStateMove : PlayerState, ICharacterState
{
    public void Enter() { /* 애니 파라미터, 입력 액션 구독 */ }
    public void Update() { /* 입력 → Rotate → 다른 상태로 전환 */ }
    public void Exit() { /* 애니 파라미터 해제, 입력 액션 구독 해제 */ }
}
```

- `Enter()` 에서 입력 액션 구독, `Exit()` 에서 반드시 해제 (메모리 누수 방지)
- 상태 전환은 `_playerController.SetState(EPlayerState.X)` 로만
- StateMachineBehaviour (`PlayerSmbToIdle`)는 애니메이션 종료 콜백으로 자동 전환

---

## 7. 상수와 Animator 파라미터

`Assets/02. Script/Common/Constants.cs` 에 모은다:

```csharp
public class Constants
{
    public const float Gravity = -9.81f;
    public static LayerMask GroundLayerMask => LayerMask.GetMask("Ground");

    public enum EPlayerState { Idle, Move, Jump, }

    // Animator string → int 캐시 (매 프레임 string lookup 방지)
    public static readonly int PlayerAniParamMove = Animator.StringToHash("move");
}
```

사용처는 `using static Constants;` 로 prefix 없이 접근. 새 enum/애니파라미터/레이어가 필요하면
**반드시 여기에 추가**하고, 사용처에서 직접 문자열 입력하지 말 것.

---

## 8. 입력 시스템 (New Input System)

PlayerInput 컴포넌트 + Action Map 사용. 마우스/키보드 직접 폴링은 Camera 와
VoiceSpellCaster 에서만 (UI 와 무관한 글로벌 입력):

```csharp
if (Mouse.current == null) return;
if (Mouse.current.rightButton.wasPressedThisFrame) { ... }

// PlayerInput Action 사용
_playerInput.actions["Jump"].performed += Jump;
var moveVector = _playerInput.actions["Move"].ReadValue<Vector2>();
```

- **`Mouse.current` 는 항상 null 체크** (입력 디바이스 미연결 환경 대비)
- 직접 폴링은 마우스 우클릭(시전), ESC(커서 락), 마우스 휠(줌) 정도
- 캐릭터 조작은 PlayerInput Actions

---

## 9. UI 패턴

`PlayerHealthUI` 처럼 **본인 한정 UI 는 코드로 즉석 생성** (씬에 미리 두지 않음):

```csharp
// PlayerHealth.OnEnable
if (photonView.IsMine && _uiInstance == null)
{
    _uiInstance = PlayerHealthUI.CreateForLocalPlayer();
    var mana = GetComponent<PlayerMana>();
    _uiInstance.Bind(this, mana);
}
```

- **이벤트 기반 UI 갱신**: `event Action<int, int> OnHealthChanged;` 를
  컴포넌트가 발행 → UI 가 구독
- TMP_InputField, TextMeshProUGUI 사용 (TextMesh Pro 만, legacy Text 사용 X)
- TMP using: `using TMPro;`

---

## 10. 디버그 로그 컨벤션

**모든 로그는 `[ClassName]` prefix** 로 필터링하기 쉽게:

```csharp
Debug.Log($"[VoiceSpellCaster] STT: \"{text}\"");
Debug.LogWarning($"[VoiceSpellCaster] 녹음 너무 짧음 ({duration:0.00}s)");
Debug.LogError("[SpellDatabase] Resources/SpellDatabase.asset 을 찾을 수 없습니다.");
```

- `Debug.Log` (정보), `Debug.LogWarning` (예상된 실패), `Debug.LogError` (구성 오류)
- 가능하면 보간 문자열 `$"…{var}…"` 사용
- 한국어 OK, 절대 콘솔에 영문만 출력하지 않을 것 (개발자 본인 언어 일관)

---

## 11. SFX 재생 — 분산 vs 동기화

| 상황 | 방법 |
|------|------|
| 본인 화면에만 들리면 OK (UI 클릭, 디버그) | `_selfAudio.PlayOneShot(...)` (로컬 AudioSource) |
| 모든 클라이언트에 동기화 (시전음, 충돌음) | `caster.SfxBroadcaster.PlayCastSfxAt(keyword, pos)` → 내부에서 `RPC_PlayCastSfx` |
| 폴백 (caster 끊긴 경우) | `AudioSource.PlayClipAtPoint(clip, pos, vol)` |

`SfxBroadcaster` 는 거리 기반 컷오프(`maxAudibleDistance`) + 임시 3D AudioSource 로
자연스러운 감쇠 처리.

**음성과 SFX 의 차이**: 음성은 Photon Voice 2 가 알아서 동기화 (Speaker 가 자동 재생),
SFX 는 Photon Voice 와 무관하게 우리가 만든 SfxBroadcaster RPC 로 동기화. 둘은 별도 시스템.

---

## 12. 외부 의존성

```
Packages 또는 Assets:
  com.whisper.unity                — STT (whisper.cpp 래퍼)
  Assets/Photon/PhotonUnityNetworking  — PUN 2
  Assets/Photon/PhotonVoice            — Photon Voice 2

Resources:
  Assets/StreamingAssets/Whisper/ggml-small.bin   — Whisper 모델 (한국어, ~466MB)

Scripting Define Symbols:
  WHISPER_UNITY
  PHOTON_VOICE_DEFINED
  PHOTON_UNITY_NETWORKING (PUN 자동 추가)
  PUN_2_0_OR_NEWER (PUN 자동 추가)
```

---

## 13. 자주 하는 실수 / 주의

1. **`Resources/` 폴더 안에 무거운 자산 넣기 금지** — 모든 빌드에 통째로 포함됨.
   현재 `Resources/` 에는 prefab 5개 + SpellDatabase.asset 만.
2. **새 발사체 추가 시 PhotonView.ObservedComponents 에 PhotonTransformView 등록 필수**.
   누락하면 다른 클라이언트에서 멈춰 보임.
3. **Photon `Instantiate` 의 prefab 이름은 `Resources/` 안 파일명** (확장자 제외).
4. **Recorder 의 `MicrophoneType` 은 `Photon` (native) 권장** — Unity Microphone API 와
   동시 점유 시 충돌 위험 줄임.
5. **PhotonServerSettings 는 `Assets/Photon/PhotonUnityNetworking/Resources/` 한 곳만**.
   여러 곳에 두면 어느 게 로드될지 비결정적.
6. **`linearVelocity` 는 Unity 6 전용**. 하위 버전이면 `velocity` 로 변경.
7. **`MainScene` 에 `PunVoiceClient` GameObject 미배치 시 자동 생성** 되지만 AppId 누락 위험.
   직접 배치 + UsePunAppSettings/UsePunAuthValues = true 권장.

---

## 14. 새 주문(Spell) 추가 가이드

1. `Assets/02. Script/Spells/` 에 새 `.cs` 작성. `Fireball.cs` 또는 `HealSpell.cs` 가
   가장 가까운 템플릿.
2. `Assets/Resources/<이름>.prefab` 생성. PhotonView + (필요시) PhotonTransformView,
   Rigidbody 부착. ObservedComponents 등록.
3. `Assets/Resources/SpellDatabase.asset` 인스펙터에서 `Spells` 배열에 SpellEntry
   추가 (keyword, prefabName, isProjectile, manaCost, damage, …, castSfx, impactSfx).
4. 끝. `VoiceSpellCaster` 코드는 건드릴 필요 없음. 키워드 매칭은 자모 거리 기반.

---

## 15. 새 RPC 추가 가이드

1. 메서드 명: `RPC_PascalCase`, `[PunRPC]` 어트리뷰트.
2. 인자: primitive + Vector3 + string 만. 객체 참조는 `int viewID` 로 보내고
   수신 측에서 `PhotonView.Find(viewID)` 로 해석.
3. 호출: `targetPv.RPC(nameof(RPC_X), RpcTarget.All, args…)` (string 보다 `nameof` 권장).
4. 상태 변경 RPC 는 `RpcTarget.All` (모든 클라이언트가 동일 상태).
   비주얼/사운드만이면 `RpcTarget.Others` 도 가능 (송신자는 즉시 처리, 다른 곳만 알림).

---

## 16. 작업 시작 전 체크리스트

코드 추가/수정 전에 매번:

- [ ] 같은 폴더의 기존 파일 1개를 열어 스타일(인코딩, XML doc, Header 사용 여부) 확인
- [ ] PhotonView 가 필요한가? → `[RequireComponent(typeof(PhotonView))]`
- [ ] `IsMine` 가드가 필요한가? → 입력/시전/UI 생성은 보통 필요
- [ ] 새 enum/상수/애니 파라미터 → `Constants.cs` 에 추가
- [ ] 디버그 로그 prefix `[ClassName]` 일관 적용
- [ ] 외부 패키지 사용 시 `#if XXX_DEFINED` 가드
- [ ] Resources 에 prefab 추가 시 PhotonView ObservedComponents 등록 확인
- [ ] 새 시스템 추가 시 — 이게 "음성으로 시전하는 마법 대전" 이라는 게임 정체성에 부합하는가?

---
> Source: [ZeroDimen/Voice-Spell](https://github.com/ZeroDimen/Voice-Spell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
