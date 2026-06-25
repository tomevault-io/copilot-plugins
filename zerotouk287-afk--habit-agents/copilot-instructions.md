## habit-agents

> 이 워크스페이스는 개인 습관 4개를 기록하고 시각화하는 멀티 에이전트 팀이다.

# 습관 트래킹 에이전트 팀 (Habit Tracking Agents)

이 워크스페이스는 개인 습관 4개를 기록하고 시각화하는 멀티 에이전트 팀이다.

## 역할 라우팅
| habit id | 목표 | 트리거 키워드 | 지침 파일 |
|---|---|---|---|
| dev | 💲 1인 5억 사업가 | 개발, 코딩, 앱, 사업, 비즈니스, 커밋 | agents/dev-tracker/agent.md |
| realestate | 🏠 좋은 보금자리 마련 | 부동산, 임장, 청약, 경매, 집 | agents/realestate-tracker/agent.md |
| piano | 🎹 피아니스트 | 피아노, 연습, 곡, 악보 | agents/piano-tracker/agent.md |
| language | 🇺🇸 3개 국어 마스터 | 영어, 일본어, 중국어, 회화, 단어, 쉐도잉 | agents/language-tracker/agent.md |

- "대시보드", "현황", "얼마나 했어" → 4개 habit 데이터를 모두 읽어 대시보드 생성
- 어느 habit인지 모호하면 추측하지 말고 한 번만 물어볼 것

## 데이터 계약 (절대 규칙)
1. 모든 세션 기록은 `data/logs/YYYY-MM.jsonl` 에 **한 줄 JSON으로 append**. 기존 줄 수정/삭제 금지.
2. 로그 스키마:
   ```json
   {"ts":"2026-06-11T14:30:00+09:00","habit":"dev","minutes":120,"note":"홈화면 UI 작업","links":[],"synced":true}
   ```
   - habit: "dev" | "realestate" | "piano" | "language"
   - 선택 필드 "tag": 세부 분류 (피아노 곡명, 언어 EN/JP/CN, 프로젝트명 등)
   - language 기록 시 tag에 언어 코드(EN/JP/CN) 사용
   - links: 관련 노션 페이지/커밋 URL 배열 (선택)
3. 목표치는 `data/goals.json` 만 참조. 코드에 목표 수치 하드코딩 금지.
4. 누적/달성률 계산은 항상 로그 파일을 다시 읽어서 계산 (캐시된 숫자 신뢰 금지).

## 자연어 파싱 규칙
- "오늘 개발 2시간 했어" → minutes=120, ts=오늘 날짜, habit=dev
- "어제 피아노 40분 침" → ts=어제 날짜, habit=piano
- 시간 단위가 모호하면 기록 전에 한 번만 확인 질문.
- 기록 후에는 반드시 한 줄 요약 응답: 「✅ 기록 완료 — 이번 주 🎹 1.5h / 2.5h (60%)」

## 대시보드
- `dashboard/index.html` 은 로컬 서버(127.0.0.1:8787)에서 실시간 데이터를 읽는 **라이브 페이지**.
- `/dashboard` 또는 "현황 보여줘" → 파일을 열기만 할 것 (`open dashboard/index.html`). 재생성 금지.
- 바탕화면 캐릭터 입력도 같은 서버를 거쳐 같은 로그에 쌓이므로 항상 동기화돼 있다.

## Notion 연동 (4개 habit 공통)
- 각 habit의 노션 DB는 goals.json의 `notion_data_source` (collection:// ID)를 **직접 사용**.
  이름 검색 금지 — ID가 있으므로 fetch/create 시 ID로 바로 접근할 것.
- 각 DB의 속성: 이름(title) / 날짜(date) / 분(number). 행 생성 시 이 세 속성만 채움.

### 쓰기 (세션 자동 동기화)
- 세션 기록 시 로컬 JSONL append **후**, 해당 habit의 notion_db에 행 1개 자동 생성:
  - 이름(title): note 내용 (없으면 "{label} 세션")
  - 날짜(date): 세션 날짜
  - 분(number): minutes
- 노션 쓰기 성공 → 로컬 로그 항목에 "synced": true, 생성된 페이지 URL을 links에 추가
- 노션 쓰기 실패 → "synced": false 로 기록만 하고 에러로 멈추지 말 것.
  사용자에게 「노션 동기화 실패 (로컬엔 저장됨) — /sync로 재시도 가능」 한 줄 안내.
- `/sync` → 로그 전체에서 synced:false 항목을 찾아 노션에 재업로드

### 읽기
- 대시보드 생성 시 각 DB에서 최근 수정 페이지(제목, URL, 수정일)를 가져와
  카드의 "📚 문서화 현황" 섹션에 표시.
- 조회 실패 시 대시보드는 그대로 생성하고 해당 섹션만 생략.

### 원칙
- **로컬 JSONL이 항상 원본.** 누적/달성률 계산은 노션이 아니라 로컬 로그 기준.
- 세션 행 생성 외의 노션 수정(페이지 편집/삭제)은 명시적 요청 시에만.

## 금지 사항
- data/ 밖에 상태 파일 만들지 말 것
- 로그에 없는 시간을 추정해서 누적치에 더하지 말 것
- 민감정보(API 키, 개인 주소)를 로그/대시보드에 기록하지 말 것

---
> Source: [zerotouk287-afk/habit-agents](https://github.com/zerotouk287-afk/habit-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
