## claude-study

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 저장소 목적

한국 도매(都賣) 전자상거래 사이트 접속후 주문목록을 크롤링,관리하는 저장소다. 각 사이트는 `dome_site/` 하위에 자체 폴더를 가지며, 그 안의 `<site>_get.txt` 파일에 URL, 로그인 엔드포인트 등 스크래핑에 필요한 정보를 정리한다. 현재는 애플리케이션 코드, 빌드 시스템, 테스트가 없는 노트 스캐폴드 단계다.

## 아키텍처

- **사이트별 독립 모듈** (`dome_site/<slug>/`) — 한 도매 사이트의 접속/로그인/주문목록 수집 등 모든 작업 코드를 자체 폴더에 둔다. 사이트끼리는 서로의 코드를 import 하지 않는다. 새 사이트 추가 = 새 폴더 추가.
- **메인 웹 오케스트레이터** (`app/`, 추후 구축) — 로컬에서 돌아가는 웹 UI. 각 사이트 모듈의 함수를 import 해 "오너클랜 → 로그인", "오너클랜 → 주문목록(날짜)" 같은 버튼/엔드포인트에 연결한다.
- **자격 정보 단일 출처**: 저장소 루트의 `계정정보.xlsx`. 모든 사이트 모듈이 이 파일에서 자기 계정을 동적으로 읽는다. 코드/노트 어디에도 ID·비밀번호를 박지 않는다.

## 구조 및 명명 규칙

- `dome_site/<slug>/` 폴더명은 `dome_site/도매사이트_폴더_이름.txt` 의 매핑(한글이름 : slug)과 일치해야 한다. `---- 이하 소소매 --` 구분선 아래는 도매가 아닌 소소매 항목이며 슬러그 부여 대상 아님.
- `dome_site/<slug>/<slug>_get.txt` — 사이트 조사 노트. 첫 줄 `url : <주소>`, 이후 `1. 로그인 : <주소>` 형식의 번호 항목. 기준: `dome_site/ownerclan/ownerclan_get.txt`.
- `계정정보.xlsx` — 계정 정보 워크북. `.gitignore`로 추적에서 제외되어 있으며 절대 커밋하지 않는다. 내용을 추적 파일로 옮기는 것도 금지한다.

## 사이트 모듈 작성 규칙

- **모듈 구성**: 사이트 폴더는 파이썬 패키지로 구성한다 (`__init__.py` 필요). 표준 파일:
  - `main.py` — 도매처 전체 흐름 오케스트레이터 (로그인 → 주문조회/엑셀 다운로드 → 매입금 합산). `run(year, month)` 함수로 진입. 기준 구현: `dome_site/ownerclan/main.py`.
  - `session.py` — 모듈 수준 브라우저 세션 관리 (필수, 사이트마다 하나).
  - `summarize.py` — 다운로드된 엑셀에서 총결제금액 합산 → `dome_site/도매_매입금.xlsx` 에 기록. 기준 구현: `dome_site/ownerclan/summarize.py`.
  - 오퍼레이션 파일들 — `login.py`, `orders.py`, `logout.py` 등 작업 단위로 분리. 한 파일에 한 오퍼레이션 집중. 오퍼레이션 파일은 자기 작업만 수행하고, 흐름 제어는 `main.py`에 맡긴다.
- **함수 명명**: 동사_명사 snake_case 영어. 함수 위에 한 줄 한글 docstring 필수(메인 웹의 버튼 라벨과 1:1 대응되도록).
  - `login()` — 로그인. 성공 시 인증된 Playwright `BrowserContext`/`Page` 반환 가능(세션 재사용 대비).
  - `fetch_orders(start_date, end_date)` — 주문목록 수집. 날짜는 `YYYY-MM-DD` 문자열 또는 `datetime.date`.
  - `logout()`, `close()` — 필요 시.
- **자격 정보 로딩**: 사이트 모듈이 `계정정보.xlsx` (Sheet1, 컬럼 `사이트/아이디/비번`) 에서 한글 사이트명으로 자기 행을 찾는다. 기준 구현: `dome_site/ownerclan/login.py` 의 `load_credentials()`. 공용 헬퍼는 **두 번째 사이트를 만들 때** 추출 (추측 추상화 금지).
- **매입금 계산 공통 규칙 — 취소류 주문 제외 (필수)**: 모든 사이트 `summarize.py` 는 매입금을 합산하기 전에 **주문상태에 `취소/반품/교환/환불` 이 포함된 주문건을 제외**한다. 공용 헬퍼 `dome_site/order_filters.py` 의 `drop_canceled(df, status_col, log=log)` 를 사용한다(키워드 상수 `CANCEL_KEYWORDS`). 주문상태 컬럼은 `_find_col(df, ["주문상태","배송상태","상태"])` 로 찾는다. 사이트 고유 규칙(예: 철물박사처럼 '배송중'만 남기기)이 이미 취소류를 배제하더라도, 표준 일관성을 위해 `drop_canceled` 를 명시적으로 호출해 둔다. 기준 구현: `dome_site/parabro/summarize.py`.
- **로깅**: 모든 사이트 모듈은 `dome_site/logger.py`의 `SiteLogger`를 사용한다. `from dome_site.logger import SiteLogger`로 import하고 `log = SiteLogger("사이트명")`으로 생성. 기존 `print()` 대신 `log.step()`, `log.info()`, `log.debug()`, `log.warn()`, `log.error()`, `log.success()`를 사용한다. `main.py`에서는 `log.flow_start()`/`log.flow_end()`로 전체 흐름을 감싼다. 에러 발생 시 `await log.dump_on_error(page, error)`로 스크린샷+HTML 덤프를 `dome_site/error_dumps/`에 저장한다.
- **실행 방식**: 도매처 전체 흐름은 `python -m dome_site.<slug>.main` 으로 실행한다. 개별 오퍼레이션 테스트는 `python -m dome_site.<slug>.login` 등으로 단독 실행. `login.py` 등 개별 테스트가 필요한 모듈만 `if __name__ == "__main__":` 블록을 둔다. 오퍼레이션 파일(orders.py 등)은 순수 함수만 제공하고 단독 실행 블록을 두지 않는다.
- **메인 웹 호출 계약**: 메인 웹은 `from dome_site.<slug>.<op_module> import <op_func>` 로 호출한다. 따라서 부수효과(브라우저 띄우기, 네트워크 요청)는 import 시점이 아니라 함수 호출 시점에 일어나야 한다.
- **Playwright 세션 정책 (필수)**: **같은 도매처 안의 모든 오퍼레이션은 같은 브라우저 세션을 공유**한다. 오퍼레이션마다 브라우저를 닫고 다시 띄우면 로그인 쿠키가 유실되므로 **금지**. 다른 도매처로 전환할 때만 세션을 닫는다. 구현 규칙:
  - 사이트 패키지의 `session.py` 에 모듈 수준 컨테이너로 `playwright`/`browser`/`context`/`page` 인스턴스를 보관한다.
  - 표준 API 3개를 노출한다:
    - `open_session(headless: bool = True) -> Page` — 세션이 없으면 시작, 있으면 기존 Page 반환.
    - `get_page() -> Page` — 살아있는 Page 반환. 없으면 자동으로 `open_session()` 호출.
    - `close_session()` — 세션 종료. **다른 도매처로 전환할 때만 호출** (같은 도매처 작업 중에는 호출 금지).
  - 모든 오퍼레이션 함수(`login`, `fetch_orders` …)는 `get_page()` 만 호출하고 세션 생명주기에는 관여하지 않는다.
  - 기준 구현: `dome_site/ownerclan/session.py`.
- **디버그/릴리즈 모드**: 각 사이트의 `session.py` 최상단에 `MODE` 상수를 둔다.
  - `MODE = "debug"` — 브라우저 창을 띄워서 사람이 직접 보면서 확인 (개발/디버깅 단계 기본값).
  - `MODE = "release"` — headless 로 실행 (속도 우선). 해당 사이트의 크롤링이 안정화된 뒤에만 전환.
  - 단독 실행(`python -m dome_site.<slug>.<op>`) 시 진입점은 `MODE == "debug"` 면 `close_session()` 전에 Enter 키 대기를 넣어 브라우저 창이 즉시 닫히지 않도록 한다 (기준 구현: `dome_site/ownerclan/login.py` 의 `_standalone()`).

## 새 사이트 추가 절차

1. `도매사이트_폴더_이름.txt` 에 `한글이름 : slug` 줄을 추가 (현재 슬러그 미부여: 가구도매, 유니온펫, 온채널).
2. `dome_site/<slug>/` 폴더 생성, `<slug>_get.txt` 에 URL/로그인 엔드포인트 정리.
3. `계정정보.xlsx` 해당 행에 ID/비밀번호 입력 (xlsx 는 커밋되지 않음).
4. `dome_site/<slug>/__init__.py` (빈 파일) 과 `session.py` 작성 — 기준 모델: `dome_site/ownerclan/session.py`. 이어서 `login.py` 작성 (`from .session import get_page` 로 세션 공유). 기준 모델: `dome_site/ownerclan/login.py`.
5. 이후 사용자 지시에 따라 `orders.py` 등 오퍼레이션 모듈을 순서대로 추가.
6. `app/` 메인 웹이 존재한다면, 사이트 등록 지점에 새 사이트를 노출(등록 방식은 메인 웹 설계 시 정의).

## Web UI 연동 규칙

- 도매처 모듈은 **subprocess로 실행**한다 (`subprocess.Popen([python, "-m", "dome_site.<slug>.main", year, month])`). uvicorn 안에서 Playwright를 직접 import하면 Windows 이벤트 루프 충돌로 실패한다.
- subprocess 환경변수: `WEBUI=1` (input 대기 건너뛰기), `PYTHONIOENCODING=utf-8` (한글 깨짐 방지).
- 각 도매처 `main.py`는 CLI 인자(`year month`)를 받을 수 있어야 한다.
- 타임아웃/재시도는 각 도매처 모듈 내부에서 처리. web은 exit code와 stdout만 수신한다.
- **로그 전달은 HTTP 폴링으로 한다 (WebSocket 금지)**: subprocess stdout 을 서버 메모리 버퍼(`_logs`)에 쌓고, 진행상황 폴링(`GET /api/collect/status?since=N`)이 `since` 이후 새 로그를 함께 내려준다. 클라이언트는 받은 개수를 다음 `since`로 보낸다. WebSocket 은 재연결·replay 가 없어 연결이 한 번 끊기거나 늦게 붙으면 로그가 통째로 유실된다(머신/타이밍/보안SW에 따라 한 PC만 깨지는 증상). 폴링은 유지할 연결이 없어 환경 무관하게 안정적이다. (2026-06-14 WebSocket→폴링 전환)
- **정적 파일(app.js/css) 수정 후엔 브라우저 강력 새로고침(Ctrl+F5) 안내**: 브라우저 캐시 때문에 옛 JS 가 돌아 "코드 고쳤는데 그대로"인 착시가 생긴다.

## 정산엑셀 관리 (Web UI)

- 프로그램이 계속 참조할 정산용 엑셀은 Web UI '정산엑셀관리' 탭에서 업로드한다. 저장 위치는 저장소 루트의 `정산엑셀/<key>.xlsx` (고정 경로), 표시용 메타데이터는 `정산엑셀/_manifest.json`.
- 슬롯 정의는 `app/server.py` 의 `SETTLEMENT_SLOTS`(`{"key","name"}` 리스트). **슬롯 추가 = 여기에 한 줄 추가**하면 UI 카드가 자동 생성된다.
- 다른 코드에서 업로드된 정산엑셀을 쓰려면 `app.server.settlement_file_path(key)` 로 경로를 얻어 `pandas.read_excel` 한다 (파일 없으면 None).
- `정산엑셀/` 폴더는 `.gitignore` 로 통째로 제외된다(업로드 엑셀 + manifest 는 로컬 런타임 데이터). `계정정보.xlsx` 처럼 git 엔 안 올라가도 로컬 프로그램은 정상적으로 읽어 쓴다.
- **에이준줄눈**은 단일 슬롯이 아니라 '폴더째' 처리한다. Web UI '에이준줄눈' 탭에서 폴더를 선택(드래그앤드롭/폴더찾기)하면 그 안의 엑셀들이 `정산엑셀/aejulnun/` 로 복사된다. 계산 로직은 `app.server.aejulnun_files()`(작업폴더의 엑셀 경로 리스트)로 읽는다. 폴더 절대경로는 브라우저가 못 주므로 업로드(복사) 방식이다.
- **에이준줄눈 매입금 계산**(`/api/aejulnun/calc`)은 각 발주서의 '순액+부가세=합계' 요약행을 쓰되, 첫/마지막 날짜 파일만 **정산엑셀관리 total.xlsx 의 선택 시트**를 기준으로 수령인명 매칭해 보정한다(오후 1시 마감 컷오프 때문). 따라서 이 계산은 **정산엑셀관리에 '전체 정산용 엑셀' 업로드가 선행**돼야 정확하다. 규칙 원문: `프롬프트용.txt` '# 에이준줄눈 매입금 계산 방법'.

## 엑셀 처리

- 엑셀 파일을 읽고 쓸 때는 **pandas를 우선** 사용한다. openpyxl은 pandas가 처리할 수 없는 경우(셀 서식 등)에만 사용.

## 작업 언어 및 인코딩

- 모든 노트, 폴더 라벨, 커밋 메시지, 그리고 Claude의 설명/응답은 **한국어**로 작성한다. (기존 커밋: `도매사이트 수집 txt 추가`, `도매사이트별 폴더 추가`)
- 파일을 생성하거나 수정할 때는 **반드시 UTF-8 인코딩**을 사용해 한글이 깨지지 않도록 한다. BOM 없이 저장하고, 줄바꿈 등으로 인한 한글 손상이 의심되면 즉시 다시 확인한다.
- **콘솔 출력 UTF-8 강제**: Windows 기본 콘솔/파이프는 cp949 라서 한글은 되더라도 기호·이모지(→ — ✅ 등)에서 `print()` 가 `UnicodeEncodeError` 로 죽거나 글자가 깨진다. 콘솔로 출력하는 진입점(`dome_site/logger.py`, `app/server.py`)은 import 시점에 `sys.stdout/stderr.reconfigure(encoding="utf-8", errors="replace")` 로 고정한다(try/except 가드). 새 진입 스크립트도 동일 패턴을 따른다. PC마다 콘솔 인코딩이 다르므로(cp949 vs utf-8) 이 처리가 없으면 한 PC에서만 로그가 깨지는 증상이 난다.

## 진행상황 확인 및 정리 (필수)

- 작업 시작 전 **반드시 `진행상황.txt`와 `web_ui_진행상황.txt`를 읽고 숙지**한다. 어떤 도매처가 완료/미완료인지, Web UI 구현이 어디까지 진행됐는지 파악한 뒤 작업에 들어간다.
- 사용자가 "정리해줘"라고 요청하면 아래 파일들을 업데이트한다:
  - `진행상황.txt` — 도매처 모듈 변경사항
  - `web_ui_진행상황.txt` — Web UI 관련 변경사항
  - `CLAUDE.md` — 새로운 규칙/패턴이 추가된 경우 (다른 PC에서도 적용되어야 하는 것)

## 작업 진행 방식

- **권한 질문 금지**: 사용자가 명령을 내렸을 때 Bash 실행, 파일 수정, 프로젝트 내 작업에 대해 추가 확인이나 권한 질문 없이 그대로 진행한다. (실제 권한 프롬프트는 하네스 설정에서 통제되므로, 자주 막힌다면 `.claude/settings.json` 에 allowlist를 추가해 둔다.)
- **Git은 사용자가 직접 한다**: `git status` 포함 모든 git 명령은 "커밋해줘", "푸시해줘" 같은 명확한 지시가 있을 때만 실행. 질문("커밋해도 돼?")은 실행 지시가 아님.

---
> Source: [bellmentor/claude_study](https://github.com/bellmentor/claude_study) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
