## claude-session-wrapup-skill

> 세션 마무리, 세션 정리, 세션 래핑, 세션 요약, 배운 점 정리, lesson learned, wrapup, 오늘 뭘 배웠지, 세션 회고, 작업 마무리 시 자동으로 추천됩니다. 세션 종료 시 사용자와 AI 각각의 '배운 것'을 구조화된 JSONL로 자동 기록합니다.


# /wrapup

세션 종료 시 2계층(세션 요약 + Lesson-Learned)을 한 번에 기록하는 스킬.

- 스키마 상세: `references/schema.md` 참조
- 저장 스크립트: `scripts/save-wrapup.py`
- 통계 스크립트: `scripts/read-stats.py`

스크립트 경로 규칙: 이 스킬의 base directory를 `$SKILL_DIR`로 참조한다.
실행 시 `python "$SKILL_DIR/scripts/save-wrapup.py"` 형태로 **절대 경로** 사용.

**CRITICAL — AskUserQuestion 호출 규칙:**
`AskUserQuestion`은 built-in 도구이다. **ToolSearch로 찾지 말고 직접 호출할 것.**
ToolSearch에서 AskUserQuestion이 검색되지 않는 것은 정상 — deferred tool이 아니라 항상 사용 가능한 내장 도구이기 때문이다.
이 스킬의 모든 AskUserQuestion 호출(Step 0/3/5/6d/8)은 **ToolSearch 없이 직접 실행**한다.

**채널 모드(`--channels`) fallback:**
`--channels` 플래그가 활성화된 세션에서는 AskUserQuestion이 **비활성화**된다 (Claude Code v2.1.80+).
이유: AskUserQuestion이 터미널을 블로킹하면 채널 메시지 수신/응답이 불가하기 때문.
- AskUserQuestion 호출이 실패하거나 도구가 없으면 → **text-based fallback** 사용
- 선택지를 텍스트로 번호 목록으로 표시하고 사용자의 번호/텍스트 입력을 받는다
- 이 fallback은 UI 품질은 떨어지지만 기능적으로 동일한 결과를 보장한다
- Anthropic이 AskUserQuestion의 채널 릴레이를 구현하면 이 fallback은 제거 예정

## 언어별 변경 힌트 매핑

| 언어 코드 | language_name | language_label | change_hint |
|-----------|---------------|----------------|-------------|
| ko | 한국어 | 랩업 언어 | `/wrapup 언어 변경` |
| en | English | Wrap-up Language | `/wrapup change lang` |
| ja | 日本語 | ラップアップ言語 | `/wrapup 言語変更` |
| zh-cn / zh | 中文(简体) | 摘要语言 | `/wrapup 切换语言` |
| 기타 | (직접 입력) | Wrap-up Language | `/wrapup change lang` |

---

## 워크플로우

아래 10단계를 순서대로 실행한다.

### Step 0: 언어 설정 확인

**사용자 입력에 "언어 변경", "change lang", "change language", "言語変更", "切换语言" 중 하나라도 포함되어 있으면 → Step 0-B로 이동.**

**Step 0-A: 기존 설정 읽기 (일반 실행)**

```bash
python "$SKILL_DIR/scripts/settings.py" read
```

- 결과에 `"initialized": true` → `language`, `language_name`, `change_hint` 값을 기억한다 → Step 1로 **무음** 진행
- 결과가 `{}` 또는 파일 없음 → Step 0-C

**Step 0-B: 언어 변경 (키워드 트리거)**

AskUserQuestion으로 언어 선택 (4개 고정 옵션 + Other):
- 한국어 (ko)
- English (en)
- 日本語 (ja)
- 中文(简体) (zh-cn)

선택 후:
```bash
python "$SKILL_DIR/scripts/settings.py" write --lang {code} --name "{name}"
```

"{name}(으)로 언어가 설정되었습니다." 안내 → Step 1로 진행.

**Step 0-C: 최초 설정 (설정 파일 없을 때)**

시스템 언어 감지:
```bash
python "$SKILL_DIR/scripts/settings.py" detect
```

AskUserQuestion 1개:
"Select the language for wrapup records. (Detected: {detected_name})"
- {detected_name} 사용 (Recommended)
- English (en)
- 한국어 (ko)
- 日本語 (ja)
- 中文(简体) (zh-cn)

선택 후 write → `language`, `language_name`, `change_hint` 기억 → Step 1로 진행.

---

### Step 1: 세션 메타정보 수집

**단일 호출로 모든 메타정보를 수집한다:**

```bash
python "$SKILL_DIR/scripts/collect-meta.py"
```

반환 JSON 필드:
- `date` — 현재 시각 (ISO 8601)
- `project` — 프로젝트 경로
- `agent` — 에이전트명 (소문자) 또는 `null` (비에이전트 세션). `CLAUDE_AGENT_NAME` 환경변수에서 자동 추출
- `session_id` — 세션 UUID
- `session_name` — 세션명 (`/rename`으로 설정한 이름)
- `timing` — 세션 시간 측정 (`session_start`, `wrapup_start`, `segment_start`, `elapsed_minutes`, `is_continuation`, `wrapup_number`)
- `stats` — 누적 통계 (user_lessons / ai_lessons / session_summaries)
  - `stats.session_summaries.total` — 현재 프로젝트 랩업 수 (에이전트 세션이면 해당 에이전트 폴더 기준)
  - `stats.session_summaries.global_total` — 전체 프로젝트 합산 랩업 수 (기존 경로 + 모든 에이전트 폴더)
  - `stats.session_summaries.total_elapsed_minutes` — 전체 누적 협업 시간 (분)

`session_name`이 빈 문자열이면 **중단** → "세션명이 설정되지 않았습니다. `/rename 세션명`으로 설정 후 다시 시도해주세요." 안내

### Step 2: 대화 컨텍스트 분석 → 2계층 드래프트 생성

**언어 규칙: 모든 내용(info, qa, conclusions, actions, lesson 제목/요약 등)은 Step 0에서 확인된 언어(`{language_name}`)로 작성한다. 대화가 다른 언어로 진행되었더라도 설정 언어로 번역하여 기록한다.**

**Auto Memory 중복 확인 (선행 작업):**

드래프트 생성 전, 현재 프로젝트의 auto memory 디렉토리를 Glob/Read 도구로 스캔한다:
- 경로: `~/.claude/projects/{project-slug}/memory/*.md`
- `{project-slug}`는 Step 1의 `project` 값에서 경로 구분자(`\`, `/`, `:`)를 `-`로 치환한 값
- auto memory 파일이 존재하면 내용을 읽어 세션 중 기록된 항목 목록을 파악한다
- 이후 Lesson-Learned 추출 시 auto memory에 이미 기록된 **사실(fact)과 동일한 내용**은 lesson에서 `[📝 auto memory]` 태그를 붙여 중복임을 표시한다
- auto memory가 사실만 기록했다면, lesson은 **맥락(context)과 발견 과정** 중심으로 경량화한다
- auto memory 디렉토리가 없거나 파일이 없으면 이 단계를 건너뛴다

전체 대화 컨텍스트에서 아래 항목을 추출한다.

**계층 1 — 세션 요약:**
- A. **정보 요약**: 세션에서 조사/파악된 핵심 정보 (불릿 포인트)
- B. **Q&A 정리**: 사용자 질문 + AI 답변 쌍
- C. **협의 결론**: 토론/비교된 안건의 결론 + 이유(rationale)
- D. **작업 내역**: 세션 중 실제로 수행한 작업 목록 (구현, 파일 수정, 버그 수정, 테스트, 문서 작업 등). Q&A나 토론이 아닌 실제 실행된 행위.
- E. **액션 아이템**: 후속 할 일 목록 (priority: high/medium/low)

**계층 2 — Lesson-Learned:**

**사용자 학습 탐지 — Q: 이 대화에서 사용자가 무언가를 알게 됐는가?**
- What/Who/When/Where 질문→AI 답변 패턴 → type: `user_fact_question`
- Why/How/비교/분석 질문→AI 답변 패턴 → type: `user_concept_question`
- 통찰/깨달음 표현 → type: `user_insight_feedback`
  - [확실한 신호] "새로 알게 됐다", "깨달았다", "처음 알았어", "이건 몰랐는데"
    "이렇게 되는 거구나", "그래서 ~이었구나", "결국 ~이구나"
  - [후속 발화와 함께] -군(요)/-네(요) 어미, "아!"/"오!" — 단독으로는 백채널과 구분 불가
- 기존 관점/전제가 틀렸음을 발견 → type: `user_perspective_shift`
  "이렇게 생각하면 안 되겠다", "내가 잘못 알고 있었네", "완전히 다르게 봐야겠어"

**AI 학습 탐지 — Q: 이 대화에서 AI가 무언가를 알게 됐거나, 방식을 바꿨는가?**
- 에러/오답 → 수정 패턴 → type: `ai_trial_error`
- 새로운 정보/맥락 발견 → type: `ai_research_discovery`
- 접근 방식 전환 → type: `ai_strategy_pivot`
- 사용자가 AI에게 가르치거나 방향을 지정 → type: `ai_user_guided`
  - 명시적 교정: "아니야", "틀렸어", "실제로는 ~야"
  - 도메인 주입: "내 상황은 ~이야", 배경/전문 지식 제공
  - 가이드라인: "~스타일로", "~는 하지 마"
  - 방식 교정: 출력 포맷/어조/수준 재지시

양쪽 모두 해당하면 양쪽 다 기록 (같은 내용이지만 관점이 다름).

각 lesson에 `category`와 `tags`를 부여한다.

**계층 3 — 후속 작업 추천:**

세션에서 완료한 작업을 바탕으로, Dylan에게 도움이 될 만한 연계 태스크를 3개 추천한다.

추천 조건:
- 방금 완료한 작업에서 자연스럽게 이어지는 후속 작업
- 작업 중 발견한 개선 기회나 잠재 이슈
- 같은 맥락에서 효율을 높일 수 있는 관련 작업
- `E:/0/_myself/profile/interests.md`를 읽고, 현재 태스크와 교차점이 있는 관심사/활성 작업도 후보에 포함한다

### Step 3: 통합 드래프트 표시 + 확인

2계층을 하나의 통합 드래프트로 표시한다. 아래 포맷 사용:
- `━` 굵은 가로선으로 주요 섹션 구분, `▸`로 서브섹션 표시
- 좌우 세로 테두리 없음 (한/영 혼용 시 정렬 깨짐)
- Q&A·협의 결론은 테이블 금지 — 번호 목록으로 표시 (터미널에서 `|` 정렬 불가)
- **소제목(`▸`) 바로 다음 줄에 가는 선(`─`)이 온다. 소제목과 가는 선 사이에 빈 줄 없음**
- **가는 선(`─`) 앞에 공백 없음. 첫 번째 열(column 0)에서 시작한다**
- 가는 선 다음에 빈 줄 하나, 그 다음 항목 시작
- **Q&A·협의 결론·Lesson 섹션: 번호 항목의 하위 내용은 중첩 불릿(`   - `)으로 표시한다.**
  - 형식: `"   - 내용"` (공백 3개 → `-` → 공백 1개 → 내용)
  - 화살표 기호(`▶`, `-->`, `→`) 사용 금지
  - **금지 패턴 예시** (아래 중 하나라도 나오면 틀린 것):
    ```
    1. 항목
- 내용        ← ❌ 들여쓰기 0칸
▶ 내용        ← ❌ 화살표 기호 사용
    ```

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
◆  세션 요약 드래프트
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

▸ 정보 요약
────────────────────────────────────────────────────────────────────────────────────────

  • ...
  • ...

▸ Q&A
────────────────────────────────────────────────────────────────────────────────────────

  1. 질문 내용
     - 답변 내용 첫 번째 줄    ← ✅ 중첩 불릿 (공백 3개 + -)
     - 답변 내용 두 번째 줄
  2. 질문 내용
     - 답변 내용

  ❌ 틀린 예 (절대 금지):
  1. 질문 내용
- 답변 내용              ← ❌ 들여쓰기 0칸
▶ 답변 내용              ← ❌ 화살표 기호 사용

▸ 협의 결론
────────────────────────────────────────────────────────────────────────────────────────

  1. 주제어 — 부연설명
     - [결정] 결정 내용     ← ✅ 중첩 불릿
     - [이유] 결정 근거
  2. 주제어
     - [결정] 결정 내용
     - [이유] 결정 근거

  포맷 규칙:
  - 항목 제목: `주제어 — 부연설명` (em dash 사용)
  - 부연설명이 불필요하면 `주제어`만 표기
  - `[주제]` 태그, `[대괄호 감싸기]` 금지

  ❌ 틀린 예 (절대 금지):
  1. [주제] 주제 내용        ← ❌ [주제] 태그 반복
  1. [서브에이전트 전략] 설명 ← ❌ 대괄호로 주제어 감싸기
- [결정] 결정 내용          ← ❌ 들여쓰기 0칸
- [이유] 결정 근거          ← ❌ 들여쓰기 0칸

▸ 작업 내역
────────────────────────────────────────────────────────────────────────────────────────

  ✓ ...
  ✓ ...

▸ 액션 아이템
────────────────────────────────────────────────────────────────────────────────────────

  ▶ [high]   ...
  ▷ [medium] ...
  · [low]    ...

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
◇  다음에 이어볼 만한 작업
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  1. 작업 제목 — 간단한 사유
  2. 작업 제목 — 간단한 사유
  3. 작업 제목 — 간단한 사유

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
◈  Lesson-Learned 드래프트
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

▸ 사용자 학습
────────────────────────────────────────────────────────────────────────────────────────

  1. 주제 내용 [학습분류]
     - 내용 요약 첫 번째 줄    ← ✅ 중첩 불릿
     - 내용 요약 두 번째 줄
  2. 주제 내용 [학습분류] [📝 auto memory]    ← auto memory에 이미 기록된 경우
     - 맥락/과정 중심 요약 (사실은 auto memory 참조)

▸ AI 학습
────────────────────────────────────────────────────────────────────────────────────────

  1. 주제 내용 [학습분류]
     - 내용 요약 첫 번째 줄    ← ✅ 중첩 불릿
     - 내용 요약 두 번째 줄
  2. 주제 내용 [학습분류] [📝 auto memory]    ← auto memory에 이미 기록된 경우
     - 맥락/과정 중심 요약

  ❌ 틀린 예 (절대 금지):
  1. 주제 내용 [학습분류]
- 내용 요약              ← ❌ 들여쓰기 0칸
▶ 내용 요약              ← ❌ 화살표 기호 사용

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**출력 전 자가 검증 (필수):**
드래프트 텍스트를 생성한 직후, 출력하기 전에 아래를 확인한다:
- Q&A 답변, 협의 결론 하위, Lesson 요약 줄이 `   - ` (공백 3개 + `-`) 형식인지 확인
- `- ` 앞에 공백 3개가 없으면(0칸 시작이면) → 들여쓰기 추가 후 재출력
- `▶`, `-->`, `→` 등 화살표 기호 사용 금지

AskUserQuestion으로 확인:
- "확인 - 이대로 기록"
- "수정 필요"
- "세션 요약만 기록"
- "Lesson-Learned만 기록"

### Step 4: 수정 루프

"수정 필요" 선택 시 사용자의 자유 텍스트 수정 지시를 받아 드래프트 반영 → Step 3 반복.

### Step 5: 후속 작업 추천 + /todo 연동

드래프트의 "다음에 이어볼 만한 작업" 추천과 "액션 아이템"을 /todo에 등록할지 제안한다.
`/todo` Skill을 호출하지 않고, `register-todos.py`를 직접 호출하여 등록한다 (선택 중복 방지).

**5-1: 등록 대상 통합 표시 + 선택**

액션 아이템과 후속 추천을 **순차 번호**로 통합하여 AskUserQuestion 안에 표시한다.
번호는 액션 아이템 → 추천 작업 순서로 1부터 연속 부여.

> **주의**: 항목 목록을 별도 텍스트로 출력하면 AskUserQuestion 오버레이에 가려진다.
> 반드시 question 필드 안에 항목 목록을 포함시킬 것.

AskUserQuestion으로 선택 (단일 선택):
- header: `ToDo 등록`
- question: 아래 형식으로 항목 목록 + 안내 문구를 **하나의 question 문자열**에 포함:
```
액션 아이템 (N건)
1. [medium] 항목 제목
2. [low] 항목 제목

추천 작업 (M건)
3. 항목 제목
4. 항목 제목
5. 항목 제목

등록할 항목을 선택하세요. 번호 직접 입력도 가능합니다 (예: 1,3,5)
```
- options:
  - `"전부 등록 (N건)"` — description: 액션 아이템 + 추천 작업 모두
  - `"액션 아이템만 (A건)"` — description: 액션 아이템만
  - `"추천 작업만 (R건)"` — description: 추천 작업만
  - `"나중에"` — description: 건너뜀

사용자가 "Other"로 쉼표 구분 번호를 입력하면(예: `1,3,5`) 해당 번호 항목만 등록한다.

액션 아이템 0건 + 추천 0건이면 이 단계를 건너뛴다.

**채택/도출 카운트 추적**: Step 5 실행 중 아래 값을 기억하여 Step 9에서 사용한다.
- `total_actions` / `adopted_actions` — 도출/채택 액션 아이템 건수
- `total_recommendations` / `adopted_recommendations` — 도출/채택 추천 아이디어 건수

**5-2: 직접 등록 (선택 확정 후)**

사용자 선택이 확정되면, `/todo` Skill을 호출하지 않고 직접 등록한다:

① todo.xlsx에서 마지막 인덱스를 조회한다:
```bash
python -c "import openpyxl; wb = openpyxl.load_workbook('E:/0/_myself/todo.xlsx'); ws = wb.active; rows = list(ws.iter_rows(min_row=2, values_only=True)); print(f'LAST_INDEX={rows[-1][0] if rows else \"td0000000000\"}'); print(f'TOTAL_ROWS={len(rows)}')" 2>/dev/null || echo "LAST_INDEX=td0000000000"
```

② 선택된 항목을 JSON으로 구성하여 Write 도구로 `$SKILL_DIR/scripts/.todo-tmp-{AGENT}-{SID8}.json`에 저장한다.

> **멀티 에이전트 경합 방지 — 임시 파일명 패턴 (필수)**
> - `{AGENT}`: Step 1의 `agent` 값 (에이전트 세션이면 소문자 에이전트명, 비에이전트면 `default`)
> - `{SID8}`: Step 1의 `session_id` 첫 8자 (UUID 앞 8자, 예: `f96d68a5`)
> - 예시: `.todo-tmp-tars-f96d68a5.json` / `.todo-tmp-default-abc123de.json`
> - 사유: 여러 에이전트 동시 wrapup 시 글로벌 단일 임시파일 경합으로 데이터 덮어쓰기 위험. 멤버별·세션별 unique 파일명으로 충돌 0 보장
각 항목의 필드:
- `index`: 마지막 인덱스 + 1부터 순차 부여 (예: `td0000000041`, `td0000000042`)
- `category`: 세션 컨텍스트에서 추론 (기존 todo의 카테고리와 일관성 유지)
- `title`: 항목 제목 (~20자)
- `details`: 배경/목적/구체 내용 1-2문장
- `status`: `"대기"`
- `created`: Step 1의 date
- `target_date`: `"미정"`
- `deadline`: `"-"`
- `notes`: 관련 참고사항, 없으면 `"-"`
- `related`: 기존 todo 중 연관된 인덱스, 없으면 `"-"`
- `context_folder`: Step 1의 project (forward slash 사용, backslash 금지)
- `session_id`: Step 1의 session_id
- `session_name`: Step 1의 session_name

③ 등록 스크립트 실행:
```bash
python "$HOME/.claude/skills/todo/scripts/register-todos.py" --file "$SKILL_DIR/scripts/.todo-tmp-{AGENT}-{SID8}.json"
```

④ 임시 파일 정리:
```bash
python -c "import os; os.remove(r'SKILL_DIR/scripts/.todo-tmp-{AGENT}-{SID8}.json')"
```

⑤ 등록 결과를 간략히 표시:
```
  todo 등록 완료: td0000000041 [카테고리] 제목, td0000000042 [카테고리] 제목
  현재 대기 중 todo: N건
```

등록 실패 시: 항목 텍스트만 표시, 수동 등록 안내.

> **참고**: 후속 추천 작업의 발굴 품질은 이후 Step 6 세션 평가의 "학습 가치" 메트릭에 반영된다.

---

### Step 6: 세션 평가

드래프트 확인 후, 세션의 **운영과 흐름(process)** 자체에 대한 품질 평가를 수행한다.
세션 주제(content)가 아니라, 세션이 얼마나 효율적이고 매끄럽게 진행되었는지를 평가한다.

**순서: AI가 먼저 → 사용자가 후에** (anchoring bias 방지)

**6a: AI 자기 진단 생성 (자동)**

대화 컨텍스트를 분석하여 5개 서브 메트릭을 각 1-5점으로 평가하고, 각 점수에 간단한 이유를 붙인다.

| 메트릭 | 평가 기준 |
|--------|-----------|
| 목표 달성 (`goal_achievement`) | 사용자 요청의 완수도 |
| 소통 효율 (`communication_efficiency`) | 의도 파악 속도, 오해 빈도 |
| 기술적 품질 (`technical_quality`) | 코드/출력물 정확도, 수정 횟수 |
| 세션 흐름 (`session_flow`) | 진행의 매끄러움, 삽질/우회 여부 |
| 학습 가치 (`learning_value`) | 의미 있는 학습 도출 여부 |

서브 메트릭 분석을 종합하여 **개선 사항(improvements)**을 도출한다.
각 개선점에는 정규화된 kebab-case `tag`를 부여한다 (향후 반복 탐지용).

> **"AI 만족도"의 정의**: AI는 감정적 만족을 느끼지 않는다. 여기서 "AI 만족도"는 **세션 품질에 대한 AI의 자기 진단(self-assessment)** 을 의미한다.

**6b: AI 평가 표시**

서브 메트릭은 시각적 별(★/☆) + 점수 + 한 줄 이유 형태로 표시:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
◇  세션 평가
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

▸ AI 만족도 (세션 품질에 대한 AI의 자기 진단)
────────────────────────────────────────────────────────────────────────────────────────

  목표 달성     ★★★★★  5/5  사용자 요청 모두 완수
  소통 효율     ★★★★☆  4/5  1회 clarification으로 의도 파악
  기술적 품질   ★★★☆☆  3/5  2번의 수정 필요
  세션 흐름     ★★★☆☆  3/5  중간에 방향 전환 1회
  학습 가치     ★★★★☆  4/5  양측 모두 의미 있는 lesson 도출

  개선 사항
  • [technical-verification] 코드 제시 전 검증 단계 추가 필요
  • [scope-management] 요청 범위 내에서 작업 집중

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

별 생성 규칙: score 수만큼 ★, 나머지 (5 - score)만큼 ☆.

**6c: 사용자 평가 — 점수**

텍스트로 1-5점 척도를 표시하고 번호 입력을 받는다 (AskUserQuestion 미사용 — 옵션 4개 제한으로 5점 척도 표현 불가):

```
이번 세션의 운영/흐름에 대해 종합 점수를 매겨주세요.

  5 — 매우 만족
  4 — 만족
  3 — 보통
  2 — 불만족
  1 — 매우 불만족
```

사용자가 숫자(1-5)를 입력하면 해당 점수로 기록한다.

**6d: 사용자 평가 — 텍스트 피드백 (선택)**

점수 입력 후, AskUserQuestion의 questions 배열로 3개 질문을 **한 번에** 표시한다.
세션 주제가 아니라 **세션 운영/흐름 자체**에 대한 피드백임을 안내한다.

```
AskUserQuestion(questions=[
  { question: "좋았던 점이 있다면? (세션 운영/흐름 관점)",  header: "좋았던 점",  multiSelect: false,
    options: [
      { label: "비워두기", description: "피드백 없이 넘어갑니다" },
      { label: "AI 추출", description: "대화 컨텍스트에서 추출을 시도합니다" }
    ]},
  { question: "아쉬웠던 점이 있다면? (세션 운영/흐름 관점)", header: "아쉬운 점",  multiSelect: false,
    options: [
      { label: "비워두기", description: "피드백 없이 넘어갑니다" },
      { label: "AI 추출", description: "대화 컨텍스트에서 추출을 시도합니다" }
    ]},
  { question: "개선 사항이 있다면? (세션 운영/흐름 관점)",  header: "개선 사항",  multiSelect: false,
    options: [
      { label: "비워두기", description: "피드백 없이 넘어갑니다" },
      { label: "AI 추출", description: "대화 컨텍스트에서 추출을 시도합니다" }
    ]}
])
```

- 사용자가 "Other"를 선택하면 직접 입력한 값을 해당 필드에 사용
- "비워두기" 선택 시 해당 필드를 빈 배열로 설정
- "AI 추출" 선택 시 대화 컨텍스트에서 세션 운영/흐름 관련 피드백을 추출 시도
- 3개 모두 "비워두기"이면 `good_points`, `bad_points`, `improvements` 모두 빈 배열

---

### Step 7: 저장

확인된 드래프트와 평가 데이터를 JSON으로 구성하여 저장한다.

**중요 — 드래프트 용어 → JSON 키 매핑:**

| 드래프트 용어 | JSON 키 | 위치 | 비고 |
|---------------|---------|------|------|
| 정보 요약 | `info` | `summary.` | 배열 of 문자열 |
| Q&A | `qa` | `summary.` | 배열 of `{q, a}` |
| 협의 결론 | `conclusions` | `summary.` | 배열 of `{topic, decision, rationale}` |
| 작업 내역 | `done` | `summary.` | 배열 of 문자열 |
| 액션 아이템 | `actions` | `summary.` | 배열 of `{title, priority}` |
| 현재 시각 | `date` | 최상위 | ISO 8601 형식 |
| AI 평가 | `ai` | `evaluation.` | `{sub_scores, improvements}` |
| 사용자 평가 | `user` | `evaluation.` | `{score, good_points, bad_points, improvements}` |
| 세션 시간 | `timing` | 최상위 | Step 1의 `timing` 객체를 그대로 전달 |
| 에이전트 | `agent` | 최상위 | Step 1의 `agent` 값을 그대로 전달 (`null` 또는 에이전트명) |

**반드시 위 키 이름을 정확히 사용할 것.** `information`, `decisions`, `work_done`, `action_items`, `timestamp` 등은 오류를 유발한다.

저장 방법 — **Write 도구 + 단일 `Bash(python:*)` 호출** 2단계로 처리한다.
`python -c "..."` 안에 데이터를 직접 내장하면 멀티라인 및 데이터 내 `$()` 텍스트가
Claude Code 안전 검사를 유발하므로 사용하지 않는다.

**① Write 도구로 JSON 파일 저장:**

JSON 데이터를 `SKILL_DIR/scripts/.wrapup-tmp-{AGENT}-{SID8}.json` 에 Write 도구로 직접 저장한다.
(Write 도구는 bash 안전 검사 대상이 아님)

> **멀티 에이전트 경합 방지 — 임시 파일명 패턴 (필수, Step 5와 동일)**
> - `{AGENT}`: Step 1의 `agent` 값 (에이전트 세션이면 소문자 에이전트명, 비에이전트면 `default`)
> - `{SID8}`: Step 1의 `session_id` 첫 8자
> - 예시: `.wrapup-tmp-tars-f96d68a5.json` / `.wrapup-tmp-default-abc123de.json`
> - 사유: 여러 에이전트 동시 wrapup 시 글로벌 단일 임시파일 경합 방지. 멤버별·세션별 unique 보장

입력 JSON 전체 구조는 `references/schema.md`의 "save-wrapup.py 입력 JSON" 참조.
`evaluation` 필드는 최상위에 위치 (summary, user_lessons, ai_lessons와 동일 레벨).

"세션 요약만" 선택 시 `user_lessons`, `ai_lessons`를 빈 배열로.
"Lesson-Learned만" 선택 시 `summary` 내부를 빈 배열로.
평가 데이터(`evaluation`)는 어떤 선택이든 항상 포함한다.

**② `--file` 인수로 저장 스크립트 실행 (단일 라인, `$()` 없음):**

```bash
python "SKILL_DIR\scripts\save-wrapup.py" --file "SKILL_DIR\scripts\.wrapup-tmp-{AGENT}-{SID8}.json"
```

실행 후 `SKILL_DIR/scripts/.wrapup-tmp-{AGENT}-{SID8}.json` 파일은 아래 단일 호출로 정리한다:

```bash
python -c "import os; os.remove(r'SKILL_DIR\scripts\.wrapup-tmp-{AGENT}-{SID8}.json')"
```

`SKILL_DIR`은 실제 스킬 base directory 절대 경로로 치환한다 (예: `C:\Users\Pro\.claude\skills\wrapup`).

저장 실패 시: 드래프트 텍스트를 그대로 출력하여 수동 저장 가능하게 안내.

### Step 8: Auto Memory 동기화 제안

저장 완료 후, auto memory와의 양방향 동기화를 제안한다.

**조건:** Lesson-Learned가 1건 이상 저장된 경우에만 실행. 0건이면 Step 9로 건너뛴다.

**동기화 로직:**

1. **Lesson → Auto Memory 방향 (승격 제안):**
   - 저장된 AI lesson 중 `[📝 auto memory]` 태그가 **없는** 항목을 확인한다
   - 이 중 향후 세션에서 반복 활용될 수 있는 패턴/도구/방법론이면 auto memory 등록을 제안한다
   - AskUserQuestion: "AI 학습 N건 중 auto memory에 등록할 항목이 있습니다:"
     - "전부 등록" → 각 항목의 핵심 사실을 auto memory에 Write/Edit
     - "선택해서 등록" → 개별 확인 후 등록
     - "건너뛰기"

2. **Auto Memory → Lesson 방향 (중복 확인):**
   - `[📝 auto memory]` 태그가 붙은 lesson이 있으면 건수를 안내한다:
     "auto memory에 이미 기록된 학습 N건은 맥락 중심으로 경량화하여 저장했습니다."

**Auto Memory 등록 시 규칙:**
- 등록 경로: `~/.claude/projects/{project-slug}/memory/` 하위 적절한 토픽 파일
- 기존 파일이 있으면 Edit 도구로 해당 섹션에 추가, 없으면 새 파일 생성
- MEMORY.md 인덱스에 새 파일 링크 추가 (없는 경우)
- lesson의 `memory_ref` 필드는 이미 저장 완료된 상태이므로 업데이트하지 않는다

### Step 9: 완료 메시지

저장 결과, 평가 요약, 누적 통계를 아래 형식으로 표시한다.
가로선으로 구역을 구분하고, 같은 줄 내 항목 구분 세로선(│)은 그대로 사용한다.
섹션 제목은 `**▸ 제목**` (볼드 + 기호)으로 표시하고, 내용은 2칸 들여쓰기한다.

**주의: 아래 템플릿은 실제 마크다운으로 출력한다. 코드블록으로 감싸지 않는다.**

```
─────────────────────────────────────────────────────────────────
## [랩업 완료] {session_name} {agent_badge}
─────────────────────────────────────────────────────────────────
**▸ 세션 시간**
  시작 HH:MM  │  랩업 HH:MM  │  소요 Xh Ym
─────────────────────────────────────────────────────────────────
**▸ 학습/기억**
  사용자 학습 N건  │  AI 학습 N건  │  auto memory N건 승격
─────────────────────────────────────────────────────────────────
**▸ ToDo**
  액션 아이템     채택 N건 / 도출 M건
  추천 아이디어   채택 X건 / 도출 Y건
  등록 합계       A건 / B건
─────────────────────────────────────────────────────────────────
**▸ 세션 평가**
  AI     목표 ★★★★★  소통 ★★★★☆  기술 ★★★☆☆  흐름 ★★★☆☆  학습 ★★★★☆
  사용자  ★★★★☆  4/5
─────────────────────────────────────────────────────────────────
**▸ 누적 통계**
  세션 N건  │  랩업 N건  │  협업 Xh Ym
  사용자 학습 N건  │  AI 학습 N건
  ToDo  활성 N건  │  진행 N건 (x%)  │  대기 N건 (z%)
─────────────────────────────────────────────────────────────────
  {language_label}: {language_name}  │  변경: {change_hint}
─────────────────────────────────────────────────────────────────
```

**세션 시간 섹션 규칙:**
- `시작`/`랩업` 시각은 `timing.segment_start`/`timing.wrapup_start`의 HH:MM 부분
- `소요`는 `timing.elapsed_minutes`를 `Xh Ym` 형식으로 변환 (1시간 미만이면 `Ym`만 표시)
- 같은 세션에서 2번째 이상 랩업인 경우 (`timing.is_continuation == true`):
  `**▸ 세션 시간 (N번째 랩업)**` + `세션 시작 HH:MM  │  구간 시작 HH:MM  │  랩업 HH:MM  │  구간 소요 Xh Ym`
- `timing`이 `null`이면 세션 시간 섹션을 생략한다

**학습/기억 섹션 규칙:**
- auto memory 승격이 0건이면 해당 항목을 생략하여 2개 항목만 표시
- 사용자 학습, AI 학습이 모두 0건이면 섹션 자체를 생략

**ToDo 섹션 규칙:**
- `채택` = Step 5에서 사용자가 등록을 선택한 건수
- `도출` = 드래프트에서 추출된 전체 건수
- `등록 합계` = 채택 액션 + 채택 추천 / 도출 액션 + 도출 추천
- Step 5를 건너뛴 경우 (액션 0건 + 추천 0건) ToDo 섹션을 생략
- Step 5에서 "나중에"를 선택한 경우: `채택 0건 / 도출 N건` 으로 표시

**누적 통계 규칙:**
- `세션 N건` = `stats.session_summaries.unique_sessions + 1` (현재 세션이 새 세션인 경우 +1, 같은 세션의 추가 랩업이면 +0)
- `랩업 N건` = `stats.session_summaries.global_total + 1` (현재 세션 포함)
- `협업 Xh Ym` = `stats.session_summaries.total_elapsed_minutes + timing.elapsed_minutes` (현재 세션 포함)
- ToDo 상태별 건수는 Step 9 시점에 todo.xlsx를 직접 조회한다 (Step 5 등록 반영):
  ```bash
  python -c "
  import openpyxl, json
  wb = openpyxl.load_workbook('E:/0/_myself/todo.xlsx')
  ws = wb.active
  counts = {}
  for row in ws.iter_rows(min_row=2, values_only=True):
      s = row[4] if len(row) > 4 and row[4] else 'unknown'
      counts[s] = counts.get(s, 0) + 1
  print(json.dumps(counts, ensure_ascii=False))
  "
  ```
- `활성` = 전체 − 완료 (대기 + 진행 중 등 미완료 항목의 합)
- `진행 N건 (x%)` — x% = 진행 / 활성, 소수점 버림
- `대기 N건 (z%)` — z% = 대기 / 활성, 소수점 버림

`{change_hint}`는 "언어별 변경 힌트 매핑" 표에서 현재 언어에 해당하는 값을 사용한다.

**에이전트 배지 규칙:**
- `{agent_badge}`: 에이전트 세션이면 `[{AGENT_NAME}]` (대문자), 비에이전트 세션이면 빈 문자열
- 예: `## [랩업 완료] 세션명 [TARS]` / `## [랩업 완료] 세션명`

## 에러 핸들링

| 상황 | 처리 |
|------|------|
| 세션명 미설정 | Step 1 중단 + `/rename` 안내 |
| JSONL 파일 없음 (첫 실행) | 자동 생성 |
| 저장 실패 | 드래프트 텍스트 출력 → 수동 저장 안내 |
| 대화 컨텍스트 너무 짧음 | "추출할 학습 내용이 부족합니다" 안내 |
| todo 등록 실패 | 액션 아이템 + 추천 작업 텍스트만 표시, 수동 등록 안내 |
| 사용자 평가 전체 스킵 (빈 입력) | evaluation.user의 텍스트 필드를 빈 배열로 저장 (score는 필수) |

---
> Source: [gonnector/claude-session-wrapup-skill](https://github.com/gonnector/claude-session-wrapup-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
