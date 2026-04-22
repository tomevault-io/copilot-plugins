## tokenization-demo

> Figma 와이어프레임 작업 시 적용. design/ 파일 편집이나 Figma MCP 사용 시 자동 적용.


# Figma 와이어프레임 작업 규칙

## 1. 작업 전 필수 확인

Figma 작업을 시작하기 전에 반드시:
1. `docs/IA.md`에서 해당 화면의 **섹션, 포함 요소, 데이터 필드** 확인
2. `docs/UserFlow.md`에서 해당 화면이 속한 **플로우의 전후 단계** 확인
3. `docs/design/screen-inventory.md`에서 기존 SCR-XX ID와 Figma Frame ID 확인

## 2. 동시 작업 충돌 방지

여러 사람이 동시에 Figma MCP로 작업할 때 충돌을 막기 위해
**작업 영역을 분리**한다. 두 가지 방식 중 상황에 맞는 것을 선택.

### 방식 A: Figma Page 분리 (안전 우선)

완전 격리. Talk to Figma MCP는 현재 보고 있는 페이지에서만 동작하므로 충돌 자체가 불가능.
단, 병합(Main으로 복사)은 수동(Ctrl+C → 페이지 이동 → Ctrl+V, 약 2초).

```
Figma 파일
├── "Main"              ← 확정본 (AI 직접 수정 금지)
├── "WIP-{이름}"        ← 나의 작업 페이지
└── "WIP-{동료이름}"    ← 동료 작업 페이지 (건드리지 않기)
```

**작업 흐름:**
1. Figma에서 자기 WIP 페이지 선택 후 플러그인 연결
2. 작업 수행 (자기 페이지에만 영향)
3. 완성 → Figma에서 Main으로 수동 복사
4. screen-inventory.md에 Main의 Frame ID 기록

**적합한 경우:** 동시 작업이 많을 때, 실수 방지가 중요할 때

### 방식 B: 같은 Page, Zone(좌표) 분리 (자동화 우선)

같은 페이지 안에서 Y좌표로 작업 영역을 나눔. clone_node로 AI가 병합까지 자동화 가능.
단, 같은 페이지이므로 실수로 상대 영역을 건드릴 수 있음.

```
Page 1 (같은 페이지)
y=0 ──────────────────
  🟦 Zone-{이름}         ← 나의 작업 영역 (y: 0 ~ 5000)
y=6000 ───────────────── (버퍼)
  🟧 Zone-{동료}         ← 동료 작업 영역 (y: 8000 ~ 13000)
y=14000 ──────────────── (버퍼)
  🟩 Main Zone           ← 확정본 영역 (y: 16000 ~)
```

**작업 흐름:**
1. 자기 Zone 좌표 범위 안에서만 프레임 생성
2. 완성 → AI에게 "SCR-XX를 Main Zone으로 옮겨줘" 요청
3. AI가 clone_node(nodeId, x: 배치좌표, y: 16000+)로 자동 복사
4. screen-inventory.md에 Main Zone의 Frame ID 기록

**병합 명령 예시:**
```
"SCR-03 완성됐어. Main Zone(y=16000)에 복사해줘."
→ AI: get_node_info → clone_node(y=16000) → screen-inventory 갱신
```

**적합한 경우:** AI 자동화를 최대한 활용하고 싶을 때, 수동 작업 최소화

### 방식 선택 가이드

| 상황 | 추천 |
|------|------|
| 동료가 Figma 작업 동시에 할 때 | 방식 A (Page) |
| 나 혼자 작업하고 동료는 리뷰만 | 방식 B (Zone) |
| 처음 시작할 때 | 방식 B로 시작, 충돌 발생 시 A로 전환 |

## 3. Figma MCP 사용 패턴

### 채널 연결
```
1. (방식 A) Figma에서 자기 WIP 페이지 선택 / (방식 B) Zone 위치 확인
2. "Cursor Talk to Figma" 플러그인 실행
3. 플러그인에 표시된 채널 ID 확인 (예: d5x5n0on)
4. join_channel로 연결
```

### 작업 순서 (화면 하나 만들기)
```
1. join_channel → 채널 연결
2. get_document_info → 현재 구조 파악
3. create_frame → 새 화면 프레임 생성 (1440×900, 자기 영역 좌표)
4. 내부 요소 배치 (create_rectangle, create_text, create_frame 등)
5. screen-inventory.md에 결과 기록
6. git commit
```

### Zone 방식 병합 명령 처리 (방식 B)
사용자가 "Main으로 옮겨줘"라고 하면:
1. get_node_info로 원본 프레임 확인
2. clone_node로 Main Zone(y=16000+)에 복사
3. screen-inventory.md 상태를 "Figma done"으로 갱신
4. 원본 WIP 프레임은 유지 (삭제하지 않음, 히스토리용)

### 와이어프레임 스타일 기준
- **프레임 크기**: 1440×900 (데스크톱 기준)
- **사이드바**: 280px, 좌측 고정
- **콘텐츠 영역**: x:312부터 시작 (사이드바 280px + 여백 32px)
- **배경**: #f9f9fa (라이트) 또는 #0B1220 (다크, 기존 화면과 맞출 것)
- **Primary 색상**: #4281ff
- **간격**: 8px 그리드 기반
- **테이블 헤더**: bg #0f1d39, 흰색 텍스트
- **테이블 바디**: 번갈아 #ffffff / #eaf1ff

## 4. 와이어프레임 수준 정의

현재는 **Mid-fidelity 와이어프레임** 단계:
- 실제 텍스트 레이블 사용 (Lorem ipsum 금지)
- 컬럼명, 버튼명은 IA.md의 정의를 그대로 사용
- 색상은 시스템 색상 사용 (임의 색상 금지)
- 아이콘은 텍스트 또는 단순 도형으로 대체 가능
- 샘플 데이터는 `ex)` 접두사로 구분 (예: `ex) 0x7a3f...4f2e`)

## 5. 화면 ID 규칙

| 접두사 | 용도 | 예시 |
|--------|------|------|
| SCR-XX | 메인 화면 | SCR-01 Dashboard |
| TX-XX | 트랜잭션 상태 오버레이 | TX-01 Sending |
| ES-XX | 빈 상태(Empty State) | ES-01 Token List |
| MOD-XX | 모달 | MOD-01 Mint Modal |

새 화면 추가 시 `screen-inventory.md`의 마지막 번호 다음을 사용.

## 6. screen-inventory.md 업데이트

Figma에 화면을 만들거나 수정한 후 반드시:
```markdown
| SCR-XX | 화면명 | IA 매핑 | Flow 매핑 | 우선순위 | 상태 | Figma Frame ID |
```
형식으로 테이블에 추가/갱신한다. **Figma Frame ID는 필수**.

### 상태값
- **WIP-{이름}** — 자기 WIP 페이지에서 작업 중 (예: `WIP-Jaewon`)
- **Review** — 완성, Main 이동 대기
- **Figma done** — Main 페이지에 확정, Frame ID는 Main 기준

## 7. 기획자 협업 시

기획자가 자연어로 요청하면 다음 순서로 처리:
1. 요청을 IA.md / UserFlow.md의 어느 부분에 해당하는지 매핑
2. 기존 화면 수정인지, 새 화면 추가인지 판단
3. Figma MCP로 실행 (로컬일 때) 또는 명세서 작성 (Cloud일 때)
4. screen-inventory.md 갱신
5. 변경 사항 커밋 제안

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jaerius) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
