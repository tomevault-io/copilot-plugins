## md2hwpx

> 마크다운(.md) 파일을 한글 HWPX 형식으로 변환하는 Python 패키지 + FastAPI 웹 서비스.

# md2hwpx 프로젝트 진행 현황

## 프로젝트 개요
마크다운(.md) 파일을 한글 HWPX 형식으로 변환하는 Python 패키지 + FastAPI 웹 서비스.

## 기술 스택
- Python 3.9+, FastAPI, uvicorn
- 순수 HTML/CSS/JS 웹 UI (외부 의존성 없음)
- pytest + httpx (비동기 테스트)

## 저장소
- GitHub: https://github.com/prosnowboard/md2hwpx (public)
- 브랜치: `main`

## 완료된 작업 (2026-01-28)

### 1. 웹 UI 구현 (`src/md2hwpx/static/index.html`)
- 한국어 단일 페이지 SPA (HTML+CSS+JS 인라인)
- 반응형 2컬럼 레이아웃 (768px 이하 세로 스택)
- 마크다운 텍스트 직접 입력 (textarea)
- 스타일 드롭다운 (기본/학술/비즈니스/미니멀) + localStorage 저장
- 드래그앤드롭 / 클릭 파일 업로드 (multiple 지원)
- 여러 파일 동시 변환 (배치) - 각 파일별 진행률/상태 표시
- 파일 변환 시 원본 .md 파일명 유지 (중복 시 (1), (2) suffix)
- Ctrl/Cmd+Enter 단축키

### 2. 서버 수정 (`src/md2hwpx/server.py`)
- `GET /` 엔드포인트 추가 (index.html 서빙)
- `_INDEX_HTML` 모듈 레벨 캐싱 (FileNotFoundError 폴백)
- `_content_disposition()` 헬퍼: RFC 5987 방식으로 한글 파일명 인코딩
  - 기존 코드에서 한글 파일명이 Content-Disposition 헤더에 들어가면 latin-1 인코딩 에러 발생 → `filename*=UTF-8''` 형식으로 해결

### 3. 테스트 (`tests/test_server.py`)
- `TestWebUI` 클래스 3개 테스트 추가:
  - `test_index_returns_html` - GET / → 200 + text/html
  - `test_index_contains_key_elements` - textarea, select, button 존재
  - `test_index_contains_fetch_call` - fetch() 코드 존재
- 전체 110개 테스트 통과

### 4. 테스트 문서 (`test-docs/`)
원본 파일(oh-my-claudecode-guide.md) 기반 6종 테스트 마크다운:
- `01-표-중심.md` - 표 4개
- `02-코드블록-중심.md` - Bash/JSON/Python/TypeScript
- `03-목록-중심.md` - 순서/비순서/체크리스트/중첩
- `04-인용구-강조-중심.md` - 인용구, 강조, 취소선
- `05-종합-전체요소.md` - H1~H6 + 모든 요소 혼합
- `06-간단-최소.md` - 제목 + 본문만

## 알려진 이슈
- **취소선**: HWPX 파일에 `<hh:strikeout shape="SINGLE">` 으로 정상 출력되나, 한컴오피스 웹뷰어에서 렌더링되지 않음. 데스크톱 한컴오피스에서 확인 필요.
- **서버 리로드**: `--reload` 옵션은 .py 파일만 감시. HTML 변경 시 서버 수동 재시작 필요 (`watchfiles` 패키지 설치하면 `--reload-include '*.html'` 사용 가능).

## 실행 방법
```bash
# 테스트
PYTHONPATH=src python3 -m pytest tests/ -x -q

# 서버 실행
PYTHONPATH=src python3 -m uvicorn md2hwpx.server:app --host 127.0.0.1 --port 8000 --reload
# http://127.0.0.1:8000/ 접속
```

## 파일 구조
```
src/md2hwpx/
├── __init__.py
├── cli.py
├── converter.py
├── parser.py
├── renderer.py
├── server.py          ← GET / 엔드포인트 추가됨
├── static/
│   └── index.html     ← 웹 UI (신규)
├── style_manager.py
└── table_handler.py
tests/
├── test_server.py     ← TestWebUI 3개 테스트 추가됨
├── test_converter.py
├── test_parser.py
└── test_cli.py
test-docs/             ← QA용 테스트 마크다운 6종 (신규)
```

---
> Source: [DevAutoFarm/md2hwpx](https://github.com/DevAutoFarm/md2hwpx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
