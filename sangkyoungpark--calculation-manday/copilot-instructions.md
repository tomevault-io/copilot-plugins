## calculation-manday

> 이 파일을 읽는 에이전트가 맥락 없이 바로 작업할 수 있도록 정리한 문서다.

# CLAUDE.md — MD 산출서 작성기

이 파일을 읽는 에이전트가 맥락 없이 바로 작업할 수 있도록 정리한 문서다.
사용자 전역 규칙(`C:\Users\GLIM\.claude\CLAUDE.md`)이 상위이며, 여기에는 이 프로젝트에만 해당하는 내용을 적는다.

---

## 1. 이게 뭐하는 물건인가

**MD(맨데이, 공수) 산출서 엑셀을 웹 화면에서 작성해 저장하는 로컬 도구**다.
여기서 MD 는 마크다운이 아니라 **Man-Day** 다. 헷갈리지 말 것.

기존 작업 방식은 이랬다 — 이전 MD 엑셀을 복사 → 파일명 바꿔 저장 → 열어서 내용 수정.
이 방식의 실제 사고가 파일에 남아 있다. `(26.01.22) OC9 통합슬리터 BCD` 와
`(26.08.07) 신뢰성시험3동 흑라인 검출` 두 파일의 셀 내용이 거의 동일하다.
복사만 하고 내용 수정을 못 한 것이다.

**그래서 이 도구는 새 MD 를 항상 빈 양식에서 시작한다.** 이전 내용이 딸려 들어오지 않게 하는 것이
편의성보다 우선하는 설계 목표다. `이전 MD 불러오기` 는 명시적으로 고를 때만 동작한다.

---

## 2. 대상 엑셀 양식

저장 위치: `D:\00. Software 업무\02. 업무 자료\05. MD\`
파일명 규칙: `[GLIM] (YY.MM.DD) MD-<제목>.xlsx`

시트 3개이며 **6개 기존 파일 전부 구조가 동일**하다. 데이터는 항상 3행부터 시작한다(2행이 헤더).

| 시트 | 구조 |
|---|---|
| `MD 산출 요약` | B열 항목명 / C열 값 의 세로 2열. 3~8행 = 프로젝트명·고객사·개발 시작·개발 종료·투입 인원·최종 산출 MD |
| `기능별 MD 산출` | B~H = No · 대분류 · 기능명 · 상세 작업 · 난이도 · 산출MD · 담당 |
| `일정 분해` | B~H = TaskID · 작업명 · 시작 · 종료 · MD · 담당 · 선행 |

**A열은 비어 있는 여백 열이다.** 데이터는 B열부터다.

투입 인원은 `"3명"`, 최종 산출 MD 는 `"14 MD"` 형태의 **문자열**이다. 숫자가 아니다.
날짜 서식은 요약 시트가 `mm-dd-yy`, 일정 시트가 `mm"월" dd"일"` 로 서로 다르다.

---

## 3. 서식을 어떻게 유지하는가

**스타일을 코드로 재현하지 않는다.** `template/MD_template.xlsx` 를 openpyxl 로 열어
값만 주입하고 다른 이름으로 저장한다. 폰트·테두리·헤더 배경색·열 너비·날짜 서식이 전부 딸려온다.

이 템플릿은 `(26.08.07)` 파일에서 3행 이하 값만 지우고 헤더와 서식은 남겨 만든 것이다.
헤더 배경색이 theme 색상 기반이라 openpyxl 로 rgb 를 못 읽는다.
**그래서 하드코딩이 아니라 템플릿 복사가 유일하게 맞는 방법이다.** 이 구조를 바꾸지 말 것.

데이터 행이 템플릿 행 수를 넘으면 `ensure_rows()` 가 마지막 행 서식을 복사해 늘린다.

---

## 4. 일정 자동 계산 규칙

사용자 요구는 "시작일이랑 며칠 걸리는지만 넣고 나머지는 자동으로" 였다.

```
종료일      = 시작일 + 일수 - 1      (달력일 기준. 주말 제외 안 함)
다음 시작일 = 이전 종료일 + 1
선행        = 이전 TaskID (첫 행은 '-')
TaskID      = T01, T02, …
```

이 규칙은 **기존 파일의 실제 5개 태스크와 전부 일치함을 확인한 것**이다. 추측이 아니다.
주말을 제외하도록 고치자는 요청이 오기 전에는 달력일 기준을 유지할 것.

### 기능별 합계 -> T03 자동 반영

사용자 지시(2026-08-10): "기능별 MD 산출은 T03에 자동으로 넣어줘 일수를 통합한 날짜로".

기능별 표의 산출MD 합계가 **T03(구현) 단계의 일수**로 그대로 들어간다.
기능을 일정에 한 줄씩 늘어놓지 않고 하나로 통합한 기간으로 잡는 방식이다.
`AUTO_MD_TASK_INDEX = 2` 는 0-based 라 T03 이다. 이름이 아니라 **자리(3번째 행)** 기준이다.

- 합계가 **0 이면 손대지 않는다.** 기능을 아직 안 적었는데 기본 5단계의 일수를 지워버리면 곤란해서다
- T03 일수를 직접 적으면 `manualMd` 가 서고 자동 반영이 멈춘다. 비우면 자동으로 복귀한다
  (`manualStart` 와 같은 `!!v` 패턴)
- `전체 다시 계산` 은 `manualMd` 도 함께 푼다
- 소수 MD 는 `setDate()` 특성상 날짜에서 소수점 이하가 버려진다. 알고 둔 것이다

### 손댄 값은 지킨다

사용자 추가 지시가 "세부사항은 입력하도록" 이었다. 자동 계산 결과를 잠그지 않는다.
날짜를 직접 고치면 그 행에 `manualStart` / `manualEnd` 플래그가 서고 화면에 **수동** 배지가 붙는다.
이후 단계는 그 값에서부터 다시 이어진다. `전체 다시 계산` 버튼이 플래그를 모두 푼다.

---

## 5. 구조

```
server.py                로컬 웹서버 (표준 라이브러리 + openpyxl, 포트 8732)
web/index.html           화면 뼈대
web/app.js               일정 계산 + 표 렌더링 + 저장
web/style.css            스타일 (:root CSS 변수)
template/MD_template.xlsx 빈 양식 — 이 파일이 서식의 원본
run.bat                  로컬 실행
```

### server.py API

- `GET /api/meta` — 기존 MD 파일 전부를 훑어 담당자·대분류·고객사·프로젝트·작업명 후보와 파일 목록을 낸다. 오타와 표기 흔들림을 막는 장치다
- `GET /api/load?file=` — 기존 MD 를 폼 데이터로 읽는다
- `GET /api/pickdir` — 윈도우 폴더 선택 창
- `GET /api/opendir` — 탐색기로 폴더 열기 (`os.startfile`, 윈도우 전용)
- `POST /api/setdir` — 저장 폴더 변경. 없는 폴더면 `needCreate` 를 돌려주고 `create:true` 로 다시 오면 만든다
- `POST /api/save` — 저장. `saveDir` 이 오면 그 폴더에 쓰고 그 선택을 기억한다. 같은 이름이 있으면 먼저 `needConfirm` 을 돌려주고, `overwrite:true` 로 다시 오면 기존 파일을 `_backup\<이름>_<타임스탬프>.xlsx` 로 복사한 뒤 덮어쓴다

### 저장 폴더

`MD_DIR` 은 이제 **런타임에 바뀌는 값**이다. `DEFAULT_MD_DIR` 이 초기값이고,
바뀐 폴더는 `APP_DIR/settings.json` 에 남아 다음 실행 때 `restore_md_dir()` 이 복원한다.
목록·불러오기·저장이 전부 이 하나를 따라간다.

폴더 선택 창은 **자기 자신을 `--pickdir` 로 다시 실행해 별도 프로세스에서 띄운다.**
tkinter 를 서버 워커 스레드에서 직접 열면 불안정해서다. exe 로 묶여도 `sys.executable` 이 그 exe 라 그대로 동작한다.
이 구조를 "간단하게" 되돌리지 말 것.

파일이 엑셀에서 열려 있으면 `PermissionError` 가 난다. 이건 화면에 안내 문구로 잡아 둔 상태다.

### 빈 행 판정 (`_row_has_content`)

`taskId` · `prev` · `level` 은 판단 근거에서 **뺀다**.
난이도 기본값 `'중'` 이 들어 있어서 빈 행이 내용 있는 행으로 잡히는 버그가 실제로 있었다.
이 세 키를 제외 목록에서 빼지 말 것.

---

## 6. 작업할 때 주의

- 이 폴더는 사용자 작업 디렉터리 밖이다. 파일을 만들면 PowerShell `Test-Path` 로 실물을 확인할 것
- 서버 프로세스는 `python.exe` 가 아니라 **`python3.13`** 으로 잡힌다.
  `Stop-Process -Name python` 은 안 먹는다. `Where-Object { $_.ProcessName -like "*python*" }` 로 잡을 것
- `server.py` 를 고쳤으면 프로세스를 죽여야 반영된다 (정적 파일은 `no-store` 라 F5 로 충분)
- **테스트할 때 실제 업무 폴더(`D:\00. Software 업무\...`)에 파일을 만들지 말 것.**
  `server.MD_DIR` 을 임시 폴더로 바꾼 별도 인스턴스로 검증한다
- 색은 하드코딩하지 말고 `style.css` 의 `--brand` / `--ok` / `--warn` / `--bad` / `--ink*` 를 쓸 것

---

## 7. 배포

깃 저장소: `https://github.com/SangKyoungPark/Calculation-ManDay` (개인, public, 기본 브랜치 `master`)

`v` 로 시작하는 태그를 밀면 `.github/workflows/release.yml` 이 windows-latest 에서 exe 를 빌드해
릴리스에 올린다(`gh release upload --clobber`). 서드파티 액션은 안 쓰고 `gh` CLI 로만 처리한다.

로컬 빌드도 같은 명령이다.

```bat
pyinstaller --noconfirm --onefile --console --name Calculation-ManDay ^
  --add-data "web;web" --add-data "template;template" server.py
```

`dist/`, `build/`, `*.spec`, `settings.json` 은 커밋하지 않는다.

## 8. 남은 것

- 실사용 검증 (실제 MD 한 건을 이 도구로 작성해 제출해 보기)
- 현재는 혼자 쓰는 로컬 전용이다. 팀 공유 요청이 오면 동시 저장 충돌 처리가 필요하다

---
> Source: [SangKyoungPark/Calculation-ManDay](https://github.com/SangKyoungPark/Calculation-ManDay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
