## consolemvc-minholee-20051289

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 기술 스택

- **언어**: Python 3.14
- **패키지 관리**: [uv](https://github.com/astral-sh/uv) (`pyproject.toml` + `uv.lock`)
- **가상환경**: `.venv/` (프로젝트 루트)
- **외부 의존성**: 없음 (표준 라이브러리만 사용)

## 주요 명령어

```powershell
# 가상환경 활성화 (PowerShell)
.\.venv\Scripts\Activate.ps1

# 앱 실행
python main.py

# 문법 검사
python -m py_compile models/user.py views/user_view.py controllers/user_controller.py main.py

# 비대화형 스모크 테스트 (stdin 없이 MVC 계층 직접 검증)
$env:PYTHONIOENCODING = "utf-8"
python -c "
import sys; sys.path.insert(0, '.')
from models.user import UserRepository
from views.user_view import UserView
from controllers.user_controller import UserController

repo = UserRepository()
view = UserView()
ctrl = UserController(repo, view)
repo.add('테스트', 'test@example.com')
ctrl.list_users()
"
```

> PowerShell에서 한글 출력이 깨질 경우 `$env:PYTHONIOENCODING = "utf-8"` 와 `[Console]::OutputEncoding = [System.Text.Encoding]::UTF8` 을 먼저 실행한다.

## 아키텍처

콘솔 기반 MVC 패턴으로, 세 계층이 **단방향 의존성**을 갖는다.

```
main.py  →  Controller  →  Model
                       →  View
```

- **Model** (`models/`): 데이터 클래스(`User`)와 인메모리 저장소(`UserRepository`). View·Controller를 일절 참조하지 않는다. 실제 DB 연동 시 `UserRepository`만 교체하면 된다.
- **View** (`views/`): `print`/`input` 전담. 계산이나 상태 변경 없이 출력과 입력 수집만 수행한다. `User` 데이터 클래스는 참조하지만 Repository는 참조하지 않는다.
- **Controller** (`controllers/`): Repository와 View를 생성자 주입(DI)으로 받아 두 계층을 조율한다. 메뉴 루프와 분기 로직이 여기에 위치한다.
- **main.py**: 세 계층을 인스턴스화하고 연결하는 진입점. 샘플 데이터 삽입 후 `controller.run()` 호출.

### `sys.path` 처리

`main.py`가 `sys.path.insert(0, str(Path(__file__).parent))`로 패키지 루트를 경로에 추가하므로, 프로젝트 루트에서 `python main.py`로 실행해야 한다. 하위 디렉터리에서 실행하면 임포트가 실패한다.

## 테스트 방법

별도의 테스트 프레임워크가 없으므로 계층별로 직접 인스턴스를 생성해 검증한다.

```python
# Model 단위 검증 예시
from models.user import UserRepository
repo = UserRepository()
u = repo.add("홍길동", "hong@example.com")
assert u.user_id == 1
assert repo.get(1).name == "홍길동"
assert repo.delete(1) is True
assert repo.get(1) is None

# Controller 검증 시 View를 목(mock)으로 대체
from unittest.mock import MagicMock
from controllers.user_controller import UserController
mock_view = MagicMock()
ctrl = UserController(repo, mock_view)
```

테스트 프레임워크를 추가할 경우 `uv add --dev pytest`로 설치하고 `pytest` 명령으로 실행한다.

---
> Source: [astralmin/ConsoleMVC-minholee-20051289](https://github.com/astralmin/ConsoleMVC-minholee-20051289) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
